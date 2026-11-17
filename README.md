# SIMHub Android 23.3 · Wi-Fi 通话稳定性修复

[下载 SIMHub 23.3 APK](SIMHub-23.3-WiFi-Calling-stability-signed.apk)

versionName **23.3**，versionCode **12**。沿用原长期签名，可覆盖同签名的 23.2，保留已有配置及卡片身份缓存；无需重新授权 eSTK ARA-M。不同签名的更早版本不能保证覆盖安装。

## 本次修复

- 坏 AMR 包不再扰乱已确认的语音来源和封装；同一语音源重启后可恢复封装检测。
- IMS 注册按服务器真实有效期提前刷新，修复短有效期过期后才续期、零有效期仍显示注册成功。
- 通话刷新按本次请求确定责任方，正确处理 UPDATE 成功响应的新间隔和角色；重复及旧响应不能覆盖状态，支持紧凑头 x/X。
- 音频启动、焦点申请或可选音效启用失败时回收资源；单项释放异常不再阻断后续清理和模式恢复。

**保留 23.2 的界面、配置交互及原生切卡指令。** 与已发布 23.2 对比，仅 7 个既有类改变、新增 1 个注册计时辅助类；UI/配置类、资源和原生库完全相同。修改了应用版本与相关协议/音频代码。

## 验证

主机回归 4867 条断言；最终 APK 实际类探针：运营商 25、语音 210、SIP 1878、注册 82、音频生命周期 73 条通过。实际 DEX 检查 54 个方法、952 次调用，类型错误为 0。协议补丁与固定基包逐类核对；v2/v3 签名、应用签名自检及 16KB 对齐通过。

这些探针使用合成协议数据及网络/硬件替身，不能代替一加手机的真实运营商注册、长通话和双向声音测试。已在 Android 11 模拟器覆盖安装、冷启动，并通过 27 项离线 ART 协议检查；设备侧 APK 哈希与交付文件一致，未出现应用崩溃或 VerifyError。详见 `emulator-validation.json`。**跨芯片互斥问题尚未确认解决，也不声称已找出全部 bug。**

## 参考与源码

本轮具体对照 PJSIP RTP/AMR、SIP 定时器实现，AOSP Android 音频/IWLAN 实现和 RFC 3550/4867/4028/3261；来源、固定版本及内容哈希保存在源码包中。未将第三方整套实现复制进应用。

[修复说明](WIFI-CALLING-23.3.zh-CN.md) · [媒体审查](MEDIA-PROTOCOL-REVIEW.zh-CN.md) · [SIP 审查](sip_protocol_review.zh-CN.md) · [构建工程说明](README.zh-CN.md) · [补丁源码](patch-source.zip)

## 完整性与签名

APK SHA-256：`4a83e74a5e207fc1ab23085ba4a9558375d2758c3fdf5b3f8dc6c3cc287d805e`

签名别名：`simhub-longterm`

证书 SHA-1 / eSTK ARA-M：`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

证书 SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

[公开证书](simhub-longterm-certificate.pem)。本次下载附件不含私钥、密码或访问令牌。
