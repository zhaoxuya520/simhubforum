# SIMHub Wi-Fi 通话音频修复包

基于上一版 DITO v29.3 参考修复包，补上 AMR-NB/WB 语音封装检测与收发适配，处理 SDP 宣告格式与实际负载不符时的大量拆包失败。参考 [maow318/simhubforum v29.3](https://github.com/maow318/simhubforum/releases/tag/v29.3) 的 AMRFramingDetector 行为。用户日志与此类问题吻合；没有原始抓包，未声称已确认实际网络每个数据包的格式。

## 安装

下载 [SIMHub-23.1-WiFi-Call-Audio-fix-signed.apk](SIMHub-23.1-WiFi-Call-Audio-fix-signed.apk)。保持原长期签名，可覆盖上一 DITO 修复包安装，保留配置；无需卸载或重置运营商参数。较早不同签名版本不能保证直接覆盖。

接通后检查能否听到对方。遇到格式不一致，日志会显示“已确认对端 AMR 封装为 …，收发已适配”。解码帧数应持续增长；若依然无声，请提供新一轮媒体日志继续排查。

## 验证及限制

- 旧 APK 实际媒体路径反例：12 个合成 RTP 包，10 个拆包失败，仅 2 帧进入原生抖动队列。
- 修复 APK 实际原生收发路径 193 项检查通过：拆包、语音字节完整性、发送封装、SDP 更新、RTP 扩展/填充、序号/时间戳回绕及多通话隔离。
- AMR 格式检查 198 项场景断言和 4000 次随机输入检查通过；既有 623 项回归通过。
- 实际 DEX 类型检查 33 个方法，0 错误；v2/v3 签名、签名自检和 16KB 对齐通过。
- 相对上一版，APK 负载只改 classes.dex。界面资源、运营商模板、短信保存、切卡逻辑和签名保持原样。
- 主机探针替代 Android 音频硬件、编码器和网络，没有执行 Android ART 或手机真实通话。声音是否恢复需手机复测。

完整说明：[WIFI-CALL-AUDIO-FIX.zh-CN.md](WIFI-CALL-AUDIO-FIX.zh-CN.md)。构建来源及回归报告随包保存，[patch-source.zip](patch-source.zip) 包含与构建哈希一致的补丁源码。没有包含私钥或密钥库密码。

## 完整性与签名

APK SHA-256：`40732c68dd5214303fc9aa51a3082a5d31ba97eeb640f046c40169a7bb74122f`

签名别名：`simhub-longterm`

证书 SHA-1 / eSTK ARA-M：`CAEAEE369CA8933EC6A80050ECAF22D87B155006`

证书 SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`

[公开证书](simhub-longterm-certificate.pem)
