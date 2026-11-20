# SIMHub Android 24.8

24.8 修复了本机 SIM 卡号被 Android 隐藏时，打开 RCS 只能看到“不可用”的问题。使用本机读卡器时，新的手动选卡入口能把收件人和草稿带到默认短信应用。

- 短信页显示 **已连接** 时，在 SIMHub 直接用原发送键发 RCS。
- 显示 **需手动选卡** 时，点 **去〈应用名〉选卡**，在打开的草稿中选目标 SIM。要发送 RCS，请确认发送栏显示 **RCS** 后再点发送。SIMHub 保留原草稿。
- 系统短信应用能否用这张卡发 RCS 取决于该应用的注册状态和收件人是否支持 RCS。系统短信应用只显示普通短信时，可以返回 SIMHub，继续保留草稿。

在 Android 11 模拟器上用普通应用权限复测，两种短信页面都复现了 24.7 无法打开草稿的问题。覆盖升级到 24.8 后，新建短信和已有会话各成功打开一份草稿，交给默认短信应用的内容只有收件人和正文。两次测试均保留了原文。运营商实网投递尚待实卡验证。[24.7 对照记录](./24.7-baseline-handoff-validation.json) · [24.8 草稿测试记录](./handoff-validation.json)

在 Android 17 模拟器上也复测了两个页面。系统隐藏完整卡号时，已有会话和新建短信都显示 **需手动选卡**。分别点击后，默认短信应用各收到一份草稿，原文仍在 SIMHub。两页共通过 86 项检查，另有 15 项草稿交接检查通过。测试使用普通应用权限和离线短信应用。真实一加手机和运营商投递未测试。[Android 17 界面复测](./android17-validation.json) · [草稿交接验证](./android17-core-validation.json)

使用与 24.7 相同的长期签名，可以直接覆盖安装。版本号 24.8，版本代码 27。

[下载 APK](./SIMHub-24.8-signed.apk) · [补丁源码与主机测试](./patch-source.zip) · [Android 设备测试源码](./device-test-source.zip) · [RCS 设备测试报告](./rcs-device-validation.json) · [SHA-256 校验](./SHA256SUMS) · [签名证书](./simhub-longterm-certificate.pem)
