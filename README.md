# SIMHub Android 23.8

[下载 23.8 安装包](SIMHub-23.8-signed.apk)

可直接覆盖同签名的 23.7，原有逐卡配置继续保留。

- DITO 专用模板的默认 IKE / EAP NAI 归属域按照认证使用的 IMSI 和卡片 MNC 长度生成。
- EAP 永久身份独立于 outer NAI，避免外层身份覆盖内部认证身份。
- ePDG 服务器和 IMS 配置继续使用所选 DITO 模板。

手工核对：本次 20404 认证身份使用内置值时，IKE 身份应以 `@nai.epc.mnc004.mcc204.3gppnetwork.org` 结尾，ePDG 域名仍包含 `mnc066.mcc515`。依据 TS 23.003 §19.3.2、RFC 4187 §§4.1.1.5、10.5；仅凭 `10500` 错误无法确认运营商是否接受该身份。

主机回归通过 5674 条断言。Android 11 模拟器覆盖安装及冷启动通过，90 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 和真实网络尚未测试。能否注册仍取决于运营商接受的身份、网络与授权；离线测试不等同于实卡注册或双向通话验证。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `1258e3c7b7f69cadd7d97932dae72fdb0f40caf202178c6d345c0a703cf93647`。
