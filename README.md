# SIMHub Android 23.6

[下载 23.6 安装包](SIMHub-23.6-signed.apk)

可直接覆盖同签名的 23.5。安装后原有 DITO 归属身份记录及逐卡配置继续可用。

- 在读卡结果返回前切换到另一读卡器、发起更新的读取或断开设备，旧任务不能再把前一张卡发布到当前 Wi-Fi 通话页。
- 来电接听通过麦克风授权并确认仍是该通电话后才停止响铃。拒绝授权或收到过期通知时保留当前来电铃声。
- 遇到失效模板或损坏记录时，直接点配置页“完成”也会写入恢复后的默认值。
- 短信页补填短信中心时可在弹窗里编辑号码，点保存后才生效。输入中间值与备选号码重合时，备选号码仍可保留。
- 分享 Wi-Fi 通话日志时隐藏 SIP 身份头里的显示名，包括用来做显示名的电话号码。

主机回归通过 5302 条断言。实际 APK 检查覆盖运营商、语音、注册、通话信令和通知。Android 11 模拟器覆盖安装及冷启动通过，125 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。一加 13 和 eSTK 双芯片上的实卡注册、双向声音及跨芯片关闭旧配置仍需硬件验证。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `36fd89fd5d9b633196479b74cdd67de26e2d52139e36caf6a0e054cdc91e56a0`。
