# SIMHub Android 23.2 · 界面与 Wi-Fi 通话优化

## 安装

[下载 SIMHub-23.2-UI-WiFi-optimized-signed.apk](SIMHub-23.2-UI-WiFi-optimized-signed.apk)

版本 23.2，versionCode 11，沿用原长期签名，可覆盖上一音频修复版，保留卡片身份缓存和自定义配置。无需重新设置 eSTK ARA-M。更早不同签名版本不能保证覆盖。

## 更新

- 全应用共享组件统一浅/深色蓝色主题、卡片层级、52dp 点击区域；保存/确认等主按钮采用实心蓝色，取消/重置/删除保留各自语义及禁用状态。
- Wi-Fi 配置：清晰的字段/输入层级、可辨别的添加/删除动作、即时运营商搜索与页数；保存及应用运营商时显示忙碌状态并禁止重复操作。
- 通话：在上版封装适配基础上，排除重复/乱序 RTP 对连续确认的影响；新语音源独立确认，支持序号回绕。
- 卡槽：异常重复回调拒绝时释放新增通道/服务，保护原待交付资源。正常原生切卡命令和重试策略保持原实现；不将此保护声称为已解决跨芯片互斥。
- 更新构建入口说明，自动运行现有主机回归，应用内版本与安装版本一致。

## 验证

主机回归 4855 项（包含随机负载检查）；实际 APK 运营商路径 25 项、媒体路径 199 项、共享主题与按钮语义 80 项通过。实际 DEX 类型检查 39 个方法，0 错误。颜色与字号之外的原生共享回调可逆核对；公开下载还会核对 SHA-256、v2/v3 签名、应用签名自检及 16KB 对齐。

主机媒体测试使用音频硬件和网络替身，不能代替真实手机通话。已在真实 Android 11 模拟器安装启动，并显示合成卡片的 NAI 配置弹窗；[浅色弹窗预览](preview-synthetic-NAI-light.png)。深色页面、多条短信中心、运营商搜索与软键盘尚未进行设备界面验收，详见 emulator-ui-validation.json。一加/eSTK 的运营商通话和跨芯片互斥问题仍需实卡验证，当前未承诺跨芯片问题全部解决。

完整工程说明：[README.zh-CN.md](README.zh-CN.md)。补丁源码：[patch-source.zip](patch-source.zip)。设计：[UI-THEME.zh-CN.md](UI-THEME.zh-CN.md)。可靠性：[MEDIA-RELIABILITY.zh-CN.md](MEDIA-RELIABILITY.zh-CN.md)。

## 完整性

APK SHA-256：`901aff4a48d4601e909dcc3ffc0d78f795458c4b8755032614d00cb796d76dff`

签名别名：`simhub-longterm`

证书 SHA-1 / eSTK ARA-M：`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

证书 SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

[公开证书](simhub-longterm-certificate.pem)。本次附件不含私钥或密码。
