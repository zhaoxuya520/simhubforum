# SIMHub Android 24.5

[下载 24.5 安装包](SIMHub-24.5-signed.apk) · [补丁源码](patch-source.zip) · [文件校验值](SHA256SUMS)

沿用 24.4 的签名，可覆盖升级。

- CTExcel UK（23433）配置外呼时，优先从当前 IMS 注册返回的公开身份中选取有效的国际号码 SIP URI，用于主叫字段；没有合适的候选时使用原来的注册身份。本地拨号带当前 IMS 域的 phone-context，完整国际号码继续使用 tel URI。
- CTExcel 在已建立 Gm IPsec 的注册连接上拨号时，初始 INVITE 增加早期媒体声明。通话诊断可查看本机是否提交 CANCEL，以及接通前收到的响应、数字原因码与 PRACK / UPDATE 顺序。
- 当前卡保存的短信中心号码或列表无效时，发送页会提示去短信中心修改并保存。保存修正后的号码会保留同卡其它有效设置；旧配置有过长 EAP 身份覆盖时也可单独编辑短信中心。

使用模拟器验证了从 24.4 覆盖安装、冷启动、呼叫与短信配置回归。CTExcel UK 真卡通话和实际短信送达尚未验证。[Android 验证结果](emulator-validation.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。
