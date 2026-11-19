# SIMHub Android 24.0

[下载 24.0 安装包](SIMHub-24.0-signed.apk)

可直接覆盖同签名的 23.9，原有逐卡配置继续保留。

修复 eSTK 双芯片卡跨芯片切换时，之前的配置仍保持启用的问题。同一读卡器发现 SE0 和 SE1 两个管理应用时，开启配置会先核对两颗芯片；另一颗芯片有启用配置时，先关闭并读回确认，再启用目标配置。

手动关闭配置后，按该配置所属芯片重新读取状态。卡片返回“配置不在启用状态”（DisableProfile 结果码 2）时，也需读回确认，才能将界面更新为关闭。同芯片切换仍交由卡片的单次启用操作替换原配置。

在可控制的双芯片读卡器替身中，已用 23.9 旧包的实际切换代码复现“只启用目标芯片、原芯片未关闭”以及手动关闭返回状态不匹配的问题。旧包与新版的验证报告随包提供。

主机回归通过 5852 条断言。Android 11 模拟器覆盖安装及冷启动通过，98 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 和真实双芯片卡仍需实测；离线测试不等同于实卡切换、移动网络接管、Wi-Fi 通话注册或双向通话验证。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json) · [签名包切换回归](signed-native-cross-switch.json) · [23.9 问题复现](legacy-runtime.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `bf4655a05a7942123e604daaff70a5a73f803689b3550bb04cacc6f5b57f41b9`。
