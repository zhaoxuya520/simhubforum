# SIMHub 25.2 · 通话翻译

在拨号键盘打开“通话翻译”，填入自己的 Gemini API 密钥，选择要显示的字幕语言。你也可以在设置中更改目标语言。接通 SIMHub Wi-Fi 通话后，对方说话时会逐段显示译文。默认字幕语言为简体中文。

25.2 调整了矮窗口中的通话页面。横持手机时可以滚动查看字幕和通话控件。使用本项目长期签名，可从 25.0 和 25.1 覆盖升级。

翻译默认关闭。开启后，应用会把对方的语音片段通过 HTTPS 发送至 Google Gemini。所需网络可用性、额度及费用取决于你的 Gemini API 项目。识别效果和字幕延迟会随语言、音质与网络状况变化。

已在 Android 模拟器中用模拟服务应答验证连续双句翻译、切换目标语言和横竖屏操作。截图中的对话为合成例句。真实运营商通话和 Google 在线翻译效果仍待实测。

[通话字幕](./caption-call.png) · [横屏通话](./caption-call-landscape.png) · [拨号](./caption-dialer.png) · [设置](./caption-settings.png)

[下载 APK](./SIMHub-25.2-signed.apk) · [补丁源码](./patch-source.zip) · [字幕验证](./caption-validation.json) · [变更范围](./apk-scope.json) · [签名验证](./longterm-signed-validation.json) · [文件校验](./SHA256SUMS)
