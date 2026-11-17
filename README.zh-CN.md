# SIMHub Android 23.2 · 界面与通话优化

这是基于固定 Android APK 的可重建补丁工程，不是原作者的 Android Studio 源码。当前入口为 `build_from_config2.py`，不要使用早期 `build.py` 生成当前版本。

## 当前界面

统一浅色/深色主题、文字对比度、分组卡片和点击区域；保存、确认等主操作采用实心按钮，删除保留危险色。配置弹窗保留简洁字段、保存和重置；支持多条 ePDG/短信中心、内置及自定义条目同等编辑删除。运营商支持即时搜索与结果分页，保存中禁用重复操作。

## Wi-Fi 通话及卡片管理

- DITO 同卡归属身份缓存与专用配置继续保留。归属 IMSI 只能来自该卡真实读取，不能由漫游 IMSI 猜出。
- AMR-NB/WB 检测实际 RTP 封装，连续确认后适配收发；重复/乱序 RTP 不再算作新证据，语音源改变后重新确认。
- 原生切卡与 APDU 路径保留，异常重复回调拒绝时清理新增通道/服务，保护原待交付资源。没有重新启用此前导致问题的实验跨芯片协调器。
- 短信中心保存、按卡配置覆盖、连接身份和签名保持兼容。

资源保护不等于跨芯片互斥问题已解决；没有真实卡片命令及读回日志时，不声称已修复用户手机上的所有切卡问题。主机测试也不能证明真实运营商注册和双向通话成功。

## 构建与验证

依赖和工具位置沿用 `tool-provenance.json` 及 `validation/` 内工具清单；基包 SHA-256 固定为 `cdd748532360f5632dc5661c0e8d9c9907967a0a136be16dfde118de43ad81c2`。

```sh
python3 build_from_config2.py
python3 sign_from_config2.py --keystore /private/signing.p12 --alias simhub-longterm --password-file /private/password --out SIMHub-23.2-signed.apk
```

构建自动运行配置/短信/DITO/卡槽/语音回归、实际 DEX 类型检查、实际 APK 运营商与媒体路径探针、主题对比度与原生回调保留检查。探针明确使用 Android 平台、音频硬件及网络替身；真实模拟器界面验证单独记录，不能代替一加/eSTK 实机测试。

安装包使用 versionName `23.2`、versionCode `11`；二进制 Manifest 仅修改版本的两个字节。签名脚本检查来源哈希、应用自检名单、v2/v3 签名和 16KB 对齐。

## 签名与交付

沿用长期签名 `simhub-longterm`，可覆盖同签名修复版。不同签名的更早版本不能保证覆盖升级。

公开证书见 `signing/local-test-certificate.pem`（保留历史文件名，内容是当前长期证书）。eSTK ARA-M SHA-1：

`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

补丁源码、公开证书和验证报告随 APK 保存到独立下载分支；源码包不得包含私钥或密码。历史文档保留调查记录，旧版本签名和操作说明不适用于本版。

## 设计与修复说明

- `UI-THEME.zh-CN.md`：共享颜色、卡片、按钮及约束。
- `MEDIA-RELIABILITY.zh-CN.md`：RTP 确认与资源保护的证据、范围和限制。
- `WIFI-CALL-AUDIO-FIX.zh-CN.md`：上一轮接通无声及封装兼容分析。
- `DITO-29.3-reference.md`：参考包的身份处理与 DITO 模板证据。
