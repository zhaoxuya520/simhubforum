# SIMHub Android 24.3

[下载 24.3 安装包](SIMHub-24.3-signed.apk)

可直接覆盖同签名的 24.2，原有逐卡配置继续保留。

- 重复收到接通应答时重发原确认，保留当前媒体配置，避免取消通话时反复发送挂断请求。
- 重复点挂断或拒接只执行一次，不重置原来的结束计时。
- 来电已拒接或被对方取消后，重传请求沿用原回复，避免重新响铃。

主机回归通过 5852 条断言。Android 11 模拟器覆盖安装及冷启动通过，345 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 与真实运营商通话尚未验证；以上为离线测试。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json) · [呼出回归](signed-invite-success.json) · [来电回归](signed-incoming-invite.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `4aff162690628b2e9265b6cf88eeb1cf93b0c644d0c999015ba1d609863abf98`。
