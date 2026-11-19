# SIMHub Android 24.1

[下载 24.1 安装包](SIMHub-24.1-signed.apk)

可直接覆盖同签名的 24.0，原有逐卡配置继续保留。

修复通话已经接通后，迟到或重复的旧 INVITE 487 响应误结束当前通话的问题。现在按原呼叫请求核对响应归属，并确认相关重传，保留已接通通话的状态。

保留正常的呼叫失败处理。运营商导致的呼叫失败仍需实卡排查。

主机回归通过 5852 条断言，签名包原生呼叫回归 78 项通过。Android 11 模拟器覆盖安装及冷启动通过，103 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 与真实运营商通话尚未验证；以上为离线测试。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json) · [签名包呼叫回归](signed-invite-failure.json) · [24.0 对照复现](legacy-invite-failure.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `1727977e6ad2fe300b9218360be8363cf8169f6dffad7b98e87be58c2a4a56dd`。
