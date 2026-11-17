# 已确认通话的 SIP 分叉保护

本次只处理已经选定并接通的外呼对话。实际 23.3 `o82.h` 回归确认：同一 Call-ID / 原 INVITE CSeq 的另一 To-tag 的 200 会覆盖已接通 A 的远端地址、路由和 SDP。后续 B BYE 200 会在 A 挂断中删除 A；B 主动 BYE 会结束 A；B UPDATE 会修改 A 的会话计时。上述缺陷使用规范长头也可复现，和运营商身份识别无关。

SIMCAT 的 `rejectCallFork` 显示先复制腿状态再发送 ACK/BYE 的实现思路。此处仅借鉴该行为，自行编写绑定和边界，不复制其源码、二进制或许可证未明的实现。未移植其固定 8 kHz 音频、较窄的 AMR 解析、单一域名 SMSC 回退和可能晚于短注册周期的更新策略。

## 实现边界

`SipForkProtection` 经 `o82.h` 和 `o82.r` 入口调用，均在原生修改事务、媒体或对话之前：

- 已选定外呼 A（`D != null`、非空 remote tag、W/X 状态）收到原 INVITE CSeq、不同非空 To-tag 的 2xx：建立独立 B，复用原生 `cb4.b/k` 与 `o82.C`，先 ACK 后 BYE。A 的对象引用、字段内容、原 INVITE、SDP、路由、CSeq、计时和事务容器不变。
- 每个不同腿重复 2xx 均 ACK。BYE 仅构造一次；同步发送失败时最多重试一次同一对象/CSeq/Via。发送接口返回不代表远端已经收到。
- B 的 ACK 构造/发送失败不继续 BYE；重传可重试。缺 Contact、无效 URI、负本地 CSeq 或 BYE CSeq 溢出不向 A 的地址发送错误清理消息。已明确为额外腿的异常被消费，不重新进入覆盖 A 的原路径。
- A 的 BYE 响应必须匹配完整两端 tag；其它/无效 tag 的 BYE 响应被隔离。A 的正常 BYE 响应仍进入原生流程。
- 入站同 Call-ID 的已选对话请求校验两端 tag。已记录 B 且本地 tag 匹配的 BYE 返回 200；其它不匹配的对话请求返回 481；ACK 不回包。回复复用 `cb4.m` / `o82.k`，沿用原入站 transport context，不能产生结束 A 的事件。CANCEL 保留原事务匹配；无 To-tag 的非对话 OPTIONS/MESSAGE 等保持原路径。
- 正常 A 请求、同 tag INVITE 200 重传、未选定腿和未接通 CANCEL / 晚到 200 路径保持原生行为。没有重写整个 SIP 状态机、认证、可靠临时响应或会话计时器。

缓存最多保存每个 A 的 64 个不同 B tag，不逐出旧腿。超过上限的新 2xx 被消费，诊断仅一次，不重新污染 A；随后该未知腿的请求回复 481。缓存用 `WeakHashMap` 弱引用 A，value 只含独立 B 和其 BYE，绝不持有 A、engine 或 manager。A 从原生对话集合释放后可被回收，缓存随下一次访问清理。B 的消息头列表、消息体、路由、事务列表及映射均独立。

局部头归一化仅用于 helper 的临时副本。实际网络解析器 `jb4.c` 本身已支持 compact t/f/v/i；测试中的直接 compact 对象是兼容和防御边界，**不是声称原网络解析器缺陷**。还实际执行 raw SIP→原生 parser→r 以验证 Via 全栈及顺序。完整头优先；Via 混用时保留原顺序，原响应/请求不被更改。

## 集成和验证

新增 `sip_fork_patch.py`：`FORK_CLASSES={'o82.smali'}`，`apply_sip_fork(smali)` 必须在 23.3 timer/registration 补丁之后调用；`reverse_sip_fork(name,text)` 精确删除两处 hook 并验证两个整方法 SHA-256。组合还原顺序为 **fork → session timer → registration**。不得移除 timer 的原 `o82.h` 整方法 pin。

DEX 验证须覆盖整方法 `o82.h(Lkb4;)Z`、`o82.r(Lkb4;Loc2;Ljava/lang/String;)V` 和 helper 所有类。两个 hook 只用原生已经初始化的寄存器，临时 v2 在继续原路径的首个使用之前被原生重新赋值。

命令：

```
python tests/sip_fork_check_integration.py build-config3/apktool --report build-config3/sip-fork-integration.json
python tests/check_native_fork_runtime.py FINAL.apk build-config3/apktool --report build-config3/native-fork-runtime.json
```

最终 APK runner 必须接在 carrier runtime 后运行，校验 `native-carrier-runtime.json` 中 APK SHA-256 与输入一致，然后执行其真实 APK 转译 jar 中的 h/r/parser/builders/helper。只在测试中收集最终网络边界 `o82.C` 与 `o82.x`，另使用日志 sink；没有替换 SIP 判断、对话处理或构包方法。测试包括主腿全字段引用/内容快照、响应不变、重复/多腿、无路由不借 A、构造/发送失败、上限、弱引用回收、完整/紧凑头、Via 顺序、原生序列化和入站 TCP context、B BYE/UPDATE、A 正常处理及 CANCEL 晚到 200。

开发阶段为实际 patched smali→DEX→JVM 的 overlay 验证；最终 APK runner 另在打包后执行。JVM 测试不等于 ART、真实 SIM 或运营商实呼，不能据此承诺现场问题全部消失。
