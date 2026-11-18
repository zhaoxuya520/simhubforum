# SIMHub Android 23.9

[下载 23.9 安装包](SIMHub-23.9-signed.apk)

可直接覆盖同签名的 23.8，原有逐卡配置继续保留。

同一网关重发上一份 EAP-AKA 身份请求时，应用现在重发原回复，只将这次交换计入认证校验一次。连接日志会在回复字节数后显示实际发送的公共身份域，例如 `mnc004.mcc204`。

手填永久 EAP 身份最多支持 1016 个 UTF-8 字节。旧配置超过上限时，连接会沿用手选运营商、ePDG 和短信中心，身份字段回退到该卡的可用值。编辑器会提示修复，点保存才会更新原记录。

主机回归通过 5852 条断言。Android 11 模拟器覆盖安装及冷启动通过，54 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

DITO 的 10500 错误仍需在一加 13 和实卡上复测。连接日志中的 `EAP 第 1 轮` 一行可用于核对身份域。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json) · [原生 EAP 回归](native-eap-runtime.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `669cb2270e4473bf6261ca627a5e459b085d82a536cadbe5567fb0b46dea2728`。
