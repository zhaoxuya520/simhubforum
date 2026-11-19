# SIMHub Android 24.4

[下载 24.4 安装包](SIMHub-24.4-signed.apk)

可直接覆盖同签名的 24.3，原有逐卡配置继续保留。

- 修复保存逐卡配置后，重启应用出现“配置记录损坏”的问题；兼容已有记录末尾的空白，保留原有配置。
- 短信发送使用当前连接卡对应的已保存短信中心，兼容旧版按卡保存的号码；明确删除的条目不会自动恢复，切卡后的旧会话也不能误用新卡配置。
- 网络在通话早期协商中要求前提条件时，后续携带 SDP 的 UPDATE 会带上相应声明。初始拨号策略保持不变。

主机回归通过 5942 条断言。Android 11 模拟器覆盖安装及冷启动通过，476 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 与真实运营商通话、短信尚未验证；以上为离线测试。本次不能据此确认现场 487 的原因或保证运营商接受呼叫。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `b921ee91f00264593b527d22be68cb763807e0e3e320d525fdb523c06be0460d`。
