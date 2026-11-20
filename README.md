# SIMHub Android 24.9

本版移除了 RCS 开关、RCS 连接和转到系统短信应用选卡的入口。新建短信和已有会话都使用 SIMHub 原来的短信输入框和发送键。

在 Android 11 模拟器上从 24.8 覆盖升级后，旧短信记录可正常打开。已有会话和新建短信各点击一次发送键，都进入了普通短信流程。测试使用离线环境，未验证运营商实际送达。

24.9 使用与 24.8 相同的长期签名，可直接覆盖安装。版本代码为 28。

[下载 APK](./SIMHub-24.9-signed.apk) · [补丁源码](./patch-source.zip) · [短信页面验证](./sms-device-validation.json) · [短信发送与 IMS 检查](./native-sms-routes.json) · [APK 差异](./apk-scope.json) · [SHA-256 校验](./SHA256SUMS) · [签名证书](./simhub-longterm-certificate.pem)
