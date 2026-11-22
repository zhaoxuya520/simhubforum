# SIMHub Android 25.0 · Wi-Fi 通话

修复通讯录里的国际号码带可选国内冠码时拨错号的问题。例如 `+32 (0)2 123 45 67`，旧版会把 `+32021234567` 交给运营商，新版拨 `+3221234567`。应用按目的地号码规则处理显式写出的 `(0)`，也接受 `0032` 开头的格式。意大利等需要保留首位零的号码继续保留该位。

Wi-Fi 通话日志现可记录不同运营商的外呼过程。发生 SIP 487 时，可从同一次通话的 INVITE、CANCEL 和网络提供的 SIP 或 Q.850 数字原因码判断终止经过。部分网络只返回 487。

24.9 已能处理分行和普通逗号合并的 `Record-Route`。25.0 修正了显示名同时含转义引号和逗号时拆错路由的问题，使后续 PRACK、UPDATE 和接通后的 ACK 发往正确的代理。处理 IMS 注册回复中的 `P-Associated-URI` 时，也修复了特定大小写和带引号的合并列表会漏掉已注册电话号码的问题。匹配的 AT&T 模板可采用卡内有效的 ISIM 身份和归属域。

外呼现在按匹配的 AT&T、FirstNet 及英国 BT 模板声明早期媒体支持。英国 23410 的主叫身份优先使用本次注册返回的有效 SIP 号码，新西兰 2degrees（53024）的本地被叫号码携带本次 IMS 域的 `phone-context`。BASE.be（20620）的模板和外呼行为另与参考包核对。

离线测试覆盖 3,607 个可选模板、3,117 个自动匹配的 PLMN 和 4,133 个带识别条件的样本。对 30 个来源网络分别输入 11 个号码，与参考包逐项比较了 330 组初始 INVITE。BASE.be 的 11 组在所比较字段上一致。部分其他模板仍有 Contact 能力和接入网信息（P-Access-Network-Info）差异。测试没有接入 BASE 的真实网络，目录里有 2,263 个标准模板仍待运营商实网验证。

25.0 的版本代码为 29，可覆盖安装使用本项目长期签名的 24.9。

[下载 APK](./SIMHub-25.0-signed.apk) · [补丁源码及许可](./patch-source.zip) · [跨运营商测试](./cross-carrier-audit.json) · [Android 安装及拨号测试](./android-dialing-validation.json) · [号码处理测试](./signed-international-dialing.json) · [外呼信令测试](./signed-ctexcel-origination.json) · [BASE 注册身份测试](./signed-base-registration-headers.json) · [Android SIP 地址解析对照](./signed-base-header-art.json) · [BASE 对话路由测试](./signed-base-dialog-routes.json) · [487 诊断测试](./signed-call-diagnostics.json) · [SHA-256 校验](./SHA256SUMS)

[普通短信入口检查](./native-sms-routes.json) · [24.9 至 25.0 的 APK 差异](./apk-scope.json) · [主机回归](./host-test-results.json) · [DEX 类型检查](./dex-type-validation.json) · [签名验证报告](./longterm-signed-validation.json) · [公开签名证书](./simhub-longterm-certificate.pem)
