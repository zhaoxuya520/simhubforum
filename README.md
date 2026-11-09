# SIMHub 23.1 手机卡槽测试版 fix2

本版本改善 OMAPI 拒绝原因定位和读取失败时的界面状态。**卡片访问授权尚未解决，不应将此版本理解为已成功管理 eSIM 或使用 Wi-Fi Calling。**

用户 fix1 截图中的提示对应 Android Session.openLogicalChannel(aid) 抛出 SecurityException。对照 EasyEUICC 后，默认 8 个管理 AID 都已包含在 SIMHub 的候选列表中；两者签名证书不同，仍需核对卡片规则。详情见同目录 OMAPI-REVIEW.zh-CN.md。

修改：显示具体 AID、操作阶段和 Android 原始拒绝原因；本地读取失败不再显示“卡内没有 Profile”，改为“尚未读取到配置”。沿用 fix1 的应用自身签名校验修正，保留系统/卡片权限检查。

可覆盖安装此前测试版，保留数据。更新后重新选择 SIM2，查看报错中的“原因”内容。尚需卡片品牌/型号及能成功操作的 EasyEUICC 版本/来源，以继续定位。

验证：131 项 JVM 断言、10 个切卡流程模拟、3 项签名交付回归，以及签名/对齐检查通过。没有完成本版本的实机验证。

SHA-256：`01d3b5ffa956195c54ca3ec1de205aaf0094e6f50c743f075312cb6f4aaaba88`
