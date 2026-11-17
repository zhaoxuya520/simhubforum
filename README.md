# SIMHub Android 23.5 · DITO 卡片识别与配置恢复

[下载 SIMHub 23.5 APK](SIMHub-23.5-DITO-recognition-signed.apk)

versionName **23.5**，versionCode **14**。沿用长期签名，可覆盖同签名的 23.4，保留应用数据及现有授权。

## 为什么会显示 Vodafone

用户此前实际日志是 `PLMN 20404，配置 Vodafone nl（custom）`。当前 USIM 的 `20404` 对应 Vodafone NL；如果程序只按当前 IMSI 匹配，就会与这张卡的 DITO 归属产生差异。23.4 已覆盖有效 DITO 发行方 ICCID + 20404 的常规情况，本轮进一步修复其未覆盖的识别与配置恢复路径。

- **修复正确识别后的错误回退**：保存的模板标识失效或记录损坏时，已确认 DITO 不再回退 Vodafone，旧 ePDG 覆盖也不再旁路干净 DITO 默认配置。编辑器可恢复保存，保存前保留原始记录，有效手动选择不变。
- **识别与当前漫游身份分开**：使用真实归属 PLMN、已知发行方和同卡历史证据，不再只支持一种当前漫游 IMSI。
- **核对 profile 服务商**：仅完整物理 ICCID 与本次传输/读卡快照中的 profile 匹配时，采纳服务商字段；昵称、选中芯片及启用开关不能替代身份核对。
- **补充已有日志中的诊断**：区分缺少 ICCID、缺少可信 DITO 依据、缺少同卡归属身份和 MNC 长度不符，避免只留下一个无法解释的运营商名称。
- **保护已有设置**：完整同 ICCID 的已加载旧身份可进行有限配置迁移；缺少 ICCID 时，不同真实 IMSI 的配置仍相互隔离。历史记录不扫描、不猜测，原记录保留。

仅 7 个既有类改变（含版本显示），新增 1 个读卡快照类。界面资源、原生库、USIM/APDU/认证/切卡类及上一轮语音和通话协议代码与 23.4 一致。

## 验证与限制

主机回归 **5201** 条断言通过。最终 APK 的实际类检查包含运营商识别 **123**、AMR 210、SIP 定时器 1878、注册 82、音频生命周期 73、通知操作 67、日志分享 19、通话分支 7291 项。DEX 检查 **221** 个方法、**2999** 次调用，类型错误为 0。v2/v3 签名、应用签名自检及 16KB 对齐通过。

Android 11 模拟器覆盖安装及冷启动通过，29 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。主机及 APK 探针使用平台、卡片传输和网络替身，不能代替一加/eSTK 实卡验证。已复现代码缺陷，但没有最新手机日志证明用户当次触发哪一条；**不声称真实运营商注册或跨芯片关闭旧卡已经修复**。

识别 DITO 与获得真实归属认证身份分开处理。恢复必须使用同 ICCID 实际读取过的完整 51566 身份，不从漫游 IMSI 或昵称制造身份。若 ICCID 未读到，程序会如实记录证据不足，不借管理页的选中项补造。

## 说明、源码及校验

[具体原因与修复边界](CARD-IDENTIFICATION-23.5.zh-CN.md) · [构建工程](README.zh-CN.md) · [补丁源码与验证证据](patch-source.zip) · [与 23.4 对比](previous-delivery-comparison.json)

APK SHA-256：`c45ec64fdf6069688e3a59f756baadf1a80afd98e899ca0ad8a2e9fbfa7242f2`

签名别名：`simhub-longterm`

证书 SHA-1 / eSTK ARA-M：`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

证书 SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

[公开证书](simhub-longterm-certificate.pem) · [文件校验值](SHA256SUMS)。附件不含私钥、密码或访问令牌。
