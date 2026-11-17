# SIP 呼叫与会话定时器审查（23.2 → 23.3）

此次修复针对真实原生代码中可复现的会话刷新错误。UI、编解码、RTP、SIM/APDU、签名与网络路由均不在补丁范围。它不能替代运营商实卡长通话测试，也不声称解决了所有 SIP 互通差异。

## 已复现并修复

| 触发 | 原行为 | 修复 |
|---|---|---|
| 最初由本机拨出的电话，对端随后发送 UPDATE/re-INVITE，指定 refresher=uac | cb4.c 将本机记为刷新方；uac 实际是这次请求的对端 | 按当前请求事务识别角色，本机接收请求即为 UAS |
| 最初由对方拨入，本机主动承担后续刷新 | o82.a 按最初方向发送 refresher=uas，把责任指回对端 | 本机主动生成刷新 UPDATE 时始终为此次事务的 UAC |
| UPDATE 的 200/202 改为 Session-Expires:90;refresher=uas | o82.h 忽略刷新响应，继续保留旧 1800 秒和本机刷新状态 | 即使响应没有 SDP，也应用最新间隔、责任方与成功响应接收时间 |
| 最新成功 UPDATE 响应无 Session-Expires | 继续旧定时器 | 按 RFC 4028 停用会话定时器 |
| 请求/响应使用合法紧凑头 x/X | 原头查询不规范化别名 | 请求与新响应处理同时支持 Session-Expires 与 x/X |
| 对端请求写 refresher=UAS/UaS | cb4.c 按大小写敏感比较，错误记录本机不刷新 | 按 SIP token 规则忽略大小写；默认 uac 和响应回显策略保持原样 |
| 重复、过期或未发送过的 UPDATE 2xx | 若直接增加解析会将旧值重新覆盖，并延后过期时间 | cb4.k 记录实际生成 UPDATE 的 CSeq；新处理只接受最新待处理 CSeq，成功消费一次 |

事务记录独立于 ma4.h，因此 UPDATE 发出后再发送 INFO，合法 UPDATE 响应仍可被接受。记录既适用于 TCP，也适用于 UDP，不依赖仅用于 UDP 重传的 F 列表。无效响应间隔（小于 90、溢出、非数字）和未知 refresher 不改变已接受的状态。对于省略 refresher 的非规范响应，沿 PJSIP 的兼容处理默认本机 UAC；省略整个 Session-Expires 则明确停用。

## 真实代码与参考

原始输入为固定 config2 APK 解码类，涉及 cb4.c、cb4.k、o82.a、o82.h；ma4 只增加默认值为 0 的 pending CSeq 字段。补丁精确固定这些方法的原文 SHA-256，可将修改反向恢复。其他 SIP 方法未被替换。

- [RFC 4028 §4、§7.2、§9、§10](https://www.rfc-editor.org/rfc/rfc4028)：最小间隔 90 秒；角色相对此次请求事务；最近成功刷新响应决定间隔和责任；无头停用；仅成功响应延长会话。
- [PJSIP sip_timer.c](https://github.com/pjsip/pjproject/blob/0e62a9d6498d91c475bcc9f98329da329836ef35/pjsip/src/pjsip-ua/sip_timer.c)：`pjsip_timer_update_req` 在发送刷新时切换当前事务角色为 UAC（本次读取 710–725 行）；接收刷新请求记录角色 UAS（1125–1172 行）；处理 2xx 时更新角色与定时器（934–947 行）。只参考行为，没有复制第三方实现。
- 参考源读取内容 SHA-256：PJSIP `ab0dbe9de67b6446999a7ba170d8f80d8f6054c36bf1569df06cb0d98ee182f9`；RFC 文本 `41b18cee9a72eb12281f4984f1a59356425d811008951f6d593c5feb6a46d72d`。

## 同时核对而保留的呼叫路径

- o82.h 的 180/183：更新早期对话，SDP 事件交给既有媒体协商；带 RSeq/100rel 时生成 RAck/PRACK。
- o82.h 的 INVITE 2xx：先更新远端标签、Contact 与 Route；有变化的 SDP 继续交给媒体层；生成 ACK。已进入 terminating 时，先 ACK，再走原生 BYE 路径。此次没有重新实现这些流程。
- cb4.j 的本机取消：未收到任何临时响应时挂起 CANCEL，收到临时响应后再发送；已 confirmed 时用 BYE。
- cb4.e 的 CANCEL 与非 2xx ACK 保留 INVITE 的 CSeq、Via 与 Route。
- k10.a 保留 183 到 200 的媒体协商：远端地址、端口、负载类型、方向更新；实际编解码改变时重建媒体。AMR 拆包修复由独立补丁处理，本次不改 codec/ptime 选择。
- 原生对无 SDP 的首次来电明确回 488；这是一项既有能力限制，未假装支持迟到 offer。

仍需单独实包复现的互通项包括：PRACK 对 RAck 的完整 CSeq/method 校验、分叉对话、UPDATE 422/Min-SE 重试、刷新失败或对端不再刷新时的到期处理。现有 PRACK 分支只提取 RAck 第一段，UPDATE 错误分支只记录日志；这些代码证据不能直接证明某运营商已经触发问题，故没有在此次补丁中扩展协议状态机。

## 验证与集成

`validation/native-sip/sip_protocol_probe.java` 调用实际 APK 的 cb4.c、cb4.k、o82.h。它使用真实模型、真实 SIP 头处理和请求构建；仅绕过 o82 的网络构造函数，不替换协议方法，不访问 SIM 或发送网络流量。SDP 正文事件也经过原处理路径。

已在旧 APK 类及独立组装的修复 DEX 上分别执行 **1642 项断言**：旧行为复现成功，修复行为通过。覆盖初始呼入/呼出、200/202、有无正文、长头和 x/X、错误间隔、责任方、TCP/UDP、UPDATE 后 INFO、重复/旧/未来 CSeq。独立审查另用实际类复现了补丁早期版本的紧凑头和重复响应问题，均已纳入正式回归。

最终审查还用候选 APK 的实际类确认请求方向的紧凑头和大写角色缺口，增加两初始方向 × UPDATE/INVITE × 长头/x/X × 六种角色大小写及缺省角色的请求矩阵，并检查原有响应回显策略。扩充后的 **1878 项断言**已在修复前模式复现问题、修复后的实际原生类通过；本轮仅增加 cb4.c 的请求头 fallback 与 equalsIgnoreCase，两处精确可逆修改。

最终完整 APK 必须继续经过项目 DEX 类型验证及实际 APK 探针，不能以独立 overlay 结果替代。执行顺序为 carrier 翻译校验后运行：

```sh
python tests/sip_check_integration.py build-config3/apktool
python tests/check_native_sip_runtime.py build-config3/unsigned.apk build-config3/apktool --report build-config3/native-sip-runtime.json
```

第二条的 APK 文件名以主构建产物为准。它要求同目录 `native-carrier-runtime.json` 记录的 APK SHA-256 与被验证 APK 完全一致，再调用该 APK 的 normalized `probe.jar`。

主构建集成 `apply_sip_session_timer(smali)`；检查用 `verify_sip_session_timer(smali)`；整类基线还原用 `reverse_sip_session_timer(name,text)`，可与注册补丁按相反顺序组合。DEX 检查须完整覆盖 cb4.c、cb4.k、cb4.sipApplyRefreshResponse、o82.a、o82.h 的所有调用，不能只筛对辅助类的 invoke。ma4 的其余字段和方法须还原为原始类。

JVM 探针需要对 R8 的 DEX→JVM 适配使用 `-noverify`；因此另做 Android DEX 寄存器类型检查，且不将 JVM 结果称为手机 ART 或运营商验证。
