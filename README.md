# SIMHub Android 23.7

[下载 23.7 安装包](SIMHub-23.7-signed.apk)

可直接覆盖同签名的 23.6。安装后原有逐卡配置继续保留。

- 选择 DITO 模板后，有效的当前读卡身份可进入原生 EAP-AKA / IMS 连接流程。
- 若同一完整 ICCID 确实读取并保存过 51566 归属身份，仍优先使用该记录；手动选择其他运营商模板时保留当前读卡身份。
- 读卡器页面的运营商、国旗和 PLMN 跟随当前生效模板显示。

主机回归通过 5545 条断言。Android 11 模拟器覆盖安装及冷启动通过，115 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 和真实网络尚未测试。能否注册仍取决于运营商接受的身份、网络与授权；离线回归不等同于实卡注册或双向通话验证。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `9f4779611ba4d5b9270699f6f504b7a01b7124f2f4b58badc9b4199c8e551375`。
