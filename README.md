# SIMHub Android 25.1 · 通话翻译

打开拨号键盘中的“通话翻译”，填写自己的 Gemini API 密钥并选择字幕语言。你也可以在设置中调整同一个开关和目标语言。应用按对方说话的内容自动识别原语言，并在 Wi-Fi 通话画面上逐段显示译文。字幕默认使用简体中文。

翻译默认关闭。开启后，对方的语音片段会通过 HTTPS 发送到 Google Gemini。需要设备能访问 Gemini API，费用和配额由你自己的 Google API 项目决定。识别效果、支持程度和字幕延迟会随语言、通话音质及网络状况变化。

25.1 使用本项目长期签名，版本代码为 30，可从 25.0 覆盖升级。

已使用模拟的云端应答在 Android 上验证字幕呈现，本次没有以个人付费密钥发送真实通话录音。通话截图展示合成例句。

[拨号界面](./caption-dialer.png) · [设置界面](./caption-settings.png) · [通话字幕示例](./caption-call.png)

重跑 Android 界面验证时，请使用源码包中 `release-checks/25.1/android-guest/` 下的 `build.py` 和 `run.py`。该目录包含双句翻译测试用的离线应答器。[测试源码校验](./source-validation.json)

[下载 APK](./SIMHub-25.1-signed.apk) · [补丁源码](./patch-source.zip) · [字幕验证](./caption-validation.json) · [变更范围](./apk-scope.json) · [签名验证](./longterm-signed-validation.json) · [SHA-256](./SHA256SUMS)
