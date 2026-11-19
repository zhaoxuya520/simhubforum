# SIMHub Android 24.1 · Wi-Fi 通话的 SIP 487 处理

这是基于固定 Android APK 的可重建补丁工程，不是原作者的 Android Studio 源码。当前入口为 `build_from_config2.py`，不要使用早期 `build.py` 生成当前版本。

24.1 修复外呼接通后，迟到的 SIP 487 仍能结束通话的问题。客户端核对 487 对应的初始 INVITE，对重传的最终响应重发 ACK。正在拨出的呼叫收到匹配的 487 时仍会结束，用户主动挂断也按原流程处理。[RFC 3261 §17.1.1.3](https://www.rfc-editor.org/rfc/rfc3261#section-17.1.1.3)

24.0 修复 eSTK 双芯片卡的配置切换。读卡器在同一设备上发现 SE0 和 SE1 两个管理应用时，开启某个配置会先核对两颗芯片。另一颗芯片上有启用配置时，先关闭并读回确认，再启用目标。手动关闭时按配置所属芯片刷新列表，卡片已关闭的配置会显示为关闭。卡片对 DisableProfile 返回代码 2 表示该配置不在启用状态，应用需要用原芯片的读回来确认最终结果。[GSMA SGP.22 §5.7.16–17](https://www.gsma.com/solutions-and-impact/technologies/esim/wp-content/uploads/2020/06/SGP.22-v2.2.2.pdf) · [EasyEUICC 的 eSTK 管理应用识别](https://github.com/estkme-group/openeuicc/blob/b79e03bbf42f4d69b9d9dd21cd6132bb1b9314d7/app-common/src/main/java/im/angry/openeuicc/util/Vendors.kt)

23.9 修复 EAP-AKA 身份请求重传时重复累计校验数据的问题。同一认证会话中收到上一份身份请求的重传时，客户端重发原回复。连接日志在身份回复的字节数后显示公共 EPC 认证域的 MNC/MCC，方便核对实际发送的身份域。手填永久 EAP 身份时按 UTF-8 编码后的长度校验，最大为 1016 字节。[RFC 3748 §4.1](https://www.rfc-editor.org/rfc/rfc3748#section-4.1) · [RFC 4187 §10.13](https://www.rfc-editor.org/rfc/rfc4187#section-10.13)

23.8 按本次用于认证的 IMSI 和卡片报告的 MNC 长度生成 DITO 模板的默认 IKE 与 EAP-AKA 身份。当卡片读出 20404 且缺少同卡 51566 归属记录时，身份的域为 `nai.epc.mnc004.mcc204.3gppnetwork.org`。ePDG 与 IMS 仍采用 DITO 配置。有同卡真实 51566 身份时，认证域为 `nai.epc.mnc066.mcc515.3gppnetwork.org`。IKE 外层 NAI 和永久 EAP 身份可独立覆盖。10500 只表明网关报告网络失败，运营商注册需要实卡验证。

23.7 允许已识别或手选 DITO 的卡使用本次有效读到的 IMSI 发起 Wi-Fi 通话注册。读到 20404 且没有同卡 51566 历史记录时，认证使用当前卡的 IMSI 和 DITO 配置的域。若这张卡曾实际读到 51566，则优先使用该记录。手选其他运营商时，配置和连接均使用现读身份。读卡器的 Wi-Fi 通话卡片显示当前模板的运营商和国家。

23.6 修复了设备切换后旧读卡结果覆盖新卡、接听来电时过早停铃、恢复默认配置后点“完成”没有写入，以及分享 SIP 日志时显示名仍携带电话号码的问题。

## 当前界面

统一浅色/深色主题、文字对比度、分组卡片和点击区域；保存、确认等主操作采用实心按钮，删除保留危险色。配置弹窗保留简洁字段、保存和重置；支持多条 ePDG/短信中心、内置及自定义条目同等编辑删除。运营商支持即时搜索与结果分页，保存中禁用重复操作。

## Wi-Fi 通话及卡片管理

- 识别 DITO 时综合实际归属身份、已知发行方、同卡记录和完整 ICCID 匹配的 profile 服务商信息，不再限制为一种漫游 IMSI。缺少 ICCID 时明确记录证据不足，不使用管理页选中状态补造身份。
- 失效运营商预设或损坏的保存记录不再使已识别的 DITO 回退 Vodafone；编辑器可恢复默认并保存，保存前保留原记录，有效手动运营商选择保持优先。

- 参考 SIMCAT 的会话绑定：通知按原生通话 ID 和本次令牌隔离，旧通知不能操作新通话；麦克风授权回调只有成功且仍为原来电才继续。
- 已确认的出站呼叫收到额外接通分支时，用独立对话处理，保留主通话的路由、音频、计时及生命周期；额外分支后续消息同样隔离。
- 现有日志分享入口隐藏 SIP/tel 用户部分及明确业务日志中的号码，处理鉴权头折行；内部日志、普通剪贴板、配置值和媒体统计保持原逻辑。
- DITO 同卡归属身份缓存与专用配置继续保留。归属 IMSI 只能来自该卡真实读取，不能由漫游 IMSI 猜出。
- AMR-NB/WB 检测实际 RTP 封装，连续确认后适配收发；坏包不再改变来源或推进序号，同 SSRC 重启可以重新确认封装，保留原生编解码和抖动缓冲。
- IMS 注册尊重服务器实际有效期，在到期前刷新；短于 60 秒的有效期不再被擅自延长，零有效期不再显示成注册成功。
- 通话刷新按本次 SIP 请求的 UAC/UAS 角色处理，接收成功 UPDATE 后同步新的 Session-Expires，避免继续沿用旧间隔或旧刷新方。
- 音频启动失败时释放已分配资源；单个资源释放失败不会跳过其余清理、焦点归还与模式恢复。
- 在同一颗芯片上换配置时，由卡片执行单次 EnableProfile 来替换原配置。跨芯片操作按管理 AID、EID 和 ICCID 定位。
- 短信中心保存、按卡配置覆盖、连接身份和签名保持兼容。

测试使用可控制的双芯片读卡器替身。一加 13 上的 eSTK 固件刷新及实际移动网络接管仍需实卡验证。

## 构建与验证

依赖和工具位置沿用 `tool-provenance.json` 及 `validation/` 内工具清单；基包 SHA-256 固定为 `cdd748532360f5632dc5661c0e8d9c9907967a0a136be16dfde118de43ad81c2`。

```sh
python3 build_from_config2.py
python3 sign_from_config2.py --keystore /private/signing.p12 --alias simhub-longterm --password-file /private/password --out SIMHub-24.1-signed.apk
```

构建自动运行配置/短信/DITO/卡槽/语音回归、实际 DEX 类型检查、实际 APK 运营商、媒体、注册、会话刷新及音频生命周期探针，以及主题对比度与原生回调保留检查。探针明确使用 Android 平台、音频硬件及网络替身；真实模拟器验证单独记录，不能代替一加/eSTK 实机测试。

安装包使用 versionName `24.1`、versionCode `20`。签名脚本检查来源哈希、应用自检名单、v2/v3 签名和 16KB 对齐。

## 签名与交付

沿用长期签名 `simhub-longterm`，可覆盖同签名修复版。不同签名的更早版本不能保证覆盖升级。

公开证书见 `signing/local-test-certificate.pem`（保留历史文件名，内容是当前长期证书）。eSTK ARA-M SHA-1：

`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

补丁源码、公开证书和验证报告随 APK 保存到独立下载分支；源码包不得包含私钥或密码。历史文档保留调查记录，旧版本签名和操作说明不适用于本版。

## 设计与修复说明

- `CARD-IDENTIFICATION-23.5.zh-CN.md`：DITO 与 Vodafone NL 的识别原因、已复现缺口及本轮修复边界。

- `SIMCAT-REFERENCE.zh-CN.md`：参考 APK 分析、采用与保留的部分、验证边界。
- `WIFI-CALLING-23.3.zh-CN.md`：上一轮已复现问题、修复范围及测试边界。
- `UI-THEME.zh-CN.md`：共享颜色、卡片、按钮及约束。
- `MEDIA-RELIABILITY.zh-CN.md`：RTP 确认与资源保护的证据、范围和限制。
- `MEDIA-PROTOCOL-REVIEW.zh-CN.md`：对照 PJSIP/RFC 的坏包和同源重启修复。
- `WIFI-CALL-AUDIO-FIX.zh-CN.md`：上一轮接通无声及封装兼容分析。
- `DITO-29.3-reference.md`：参考包的身份处理与 DITO 模板证据。
