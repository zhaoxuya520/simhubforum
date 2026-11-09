# SIMHub 23.1 手机卡槽测试版 fix1

修正上一测试版报错：Security verification failed. Card operations are unavailable in this app copy.

原因是重新签名后，应用内部证书允许列表仍仅包含原证书。fix1 仅加入本次测试证书，保留原校验流程；系统和卡片的 OMAPI 访问授权仍须满足。

APK：SIMHub-23.1-phone-slot-fix1-signed.apk

沿用上一测试版签名，可以覆盖安装上一测试版并保留数据。与原始正式版签名不同，不能直接覆盖正式版。

本地检查：117 项 JVM 断言、10 个切卡流程模拟、3 项签名交付回归通过；签名、对齐及最终 APK 自检证书匹配检查通过。修正后的 EID、配置管理、Wi-Fi Calling、短信和通话仍待实机验证。

SHA-256：`06c8899e0dc91d4c83084e3e8728a3db5577ee3ba8da22fdc183fd77f8543c22`
