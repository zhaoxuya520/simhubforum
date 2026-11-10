# SIMHub 23.1 手机卡槽版 fix2-config1

原“Wi-Fi 通话 → 查看 / 修改配置”改为可编辑表单，支持 ePDG、SMSC、IMEI、IMS Domain / APN / IP / IPsec / NAI、IMPI / IMPU / EAP Identity，以及 IKE 加密 / 完整性 / PRF / DH、ESP 加密 / 完整性与 Keepalive，共 18 项。

用户填写的值优先；单项清空后保存使用当前卡片的内置值。每张卡片独立保存；取消不保存；支持“全部使用内置”。页面显示对应内置值和格式说明。请先关闭 Wi-Fi 通话，修改保存后重新开启。

自定义算法列表不会被原引擎额外追加固定算法，显式 IPv4 / IPv6 不会自动改用其他内层地址族。未填写时保留内置协商行为。算法按当前引擎支持范围校验，Keepalive 支持 10–3600 秒。

继承 fix2-cn1 的 +86 通话放行和号码格式处理，未加入双芯片互斥或节点检测；不宣称修复此前的 SIP 487 或二进制短信显示问题。

安装 `SIMHub-23.1-phone-slot-fix2-config1-signed.apk`，与此前测试版使用同一签名，可覆盖安装，已有 ARA-M 签名授权继续适用。此安装包为 Android APK。

验证：263 项 JVM 断言（其中 95 项为配置功能）、10 个切卡流程模拟、29 项号码接入及 48 项配置接入静态检查、3 项签名回归通过；v2/v3 签名、16 KB 对齐与包完整性已验证。未进行 Android 实机界面或运营商注册/通话测试。

SHA-256：`6001685cc939fa3a2a0ade5d1fd17987a85e980bf8451b8738ee84d79ab4ad7d`

签名 SHA-1：`6F92BF00AF155C4ADA27936451F95ED2F703039C`

源码补丁及验证报告在 `SIMHub-23.1-phone-slot-fix2-config1-patch-source.zip`，包含 `CONFIG-REVIEW.zh-CN.md` 中的设计及实机验证步骤。私钥和签名密码不在发布文件中。
