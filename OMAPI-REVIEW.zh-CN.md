# fix2：对照 EasyEUICC 的本地卡槽访问复查

## 实机证据与结论边界

用户在未 Root 的一加 13 上反馈，EasyEUICC 能列出同一卡片的配置并切卡。SIMHub fix1 截图中，SIM2 的 ISD-R 管理通道和 USIM 认证通道均显示此前补丁生成的“未授权”提示。

该提示只在 `Session.openLogicalChannel(aid)` 抛出 `SecurityException` 时生成。因此可以确认 Android OMAPI 拒绝了打开通道，尚未进入这些通道的读卡/认证操作。上一版将所有此类异常统一解释为卡片规则问题，并丢弃了平台具体消息；仅凭截图还不能确定是证书规则、AID 规则、应用包名规则还是其他系统侧限制。

## 与 EasyEUICC 的逐项对照

参考：`estkme-group/openeuicc`，提交 `b79e03bbf42f4d69b9d9dd21cd6132bb1b9314d7`。

| 项目 | EasyEUICC | SIMHub / 结论 |
|---|---|---|
| 非特权通道 | `SEService`、`getUiccReader`、`openSession`、`openLogicalChannel(aid)` | 相同的 Android OMAPI 接口；没有发现额外的普通应用授权接口 |
| 逻辑卡槽号 | Android 11+ 使用 `logicalSlotIndex + 1` | 使用系统实际 Reader，以及从 1 开始的 `getUiccReader`；没有将 SIM2 当作 SIM1 |
| 默认管理 AID | 标准、eSIM.me、5ber、Xesim、LinksField、eSTK SE0/SE1/AUX，共 8 个 | 原 SIMHub 候选列表包含全部 8 个，共 10 个；已逐项比对，没有遗漏默认 AID |
| 单个 AID 失败 | 继续候选尝试，受平台授权约束 | 发现阶段已经继续尝试其他候选；已有回归覆盖“标准 AID 拒绝、厂商 AID 成功” |
| 证书授权 | README 要求卡片 ARA-M 支持官方证书 | 本测试版是不同证书，EasyEUICC 获准不代表 SIMHub 获准 |
| USIM / Wi-Fi Calling | 非特权 eUICC 管理接口不代表 USIM AKA/IMS 能力 | USIM AID 与管理 AID 独立；管理授权不能作为通信认证已获准的证据 |

官方文档公布的 EasyEUICC ARA-M SHA-1：

`2A2FA878BC7C3354C2CF82935A5945A3EDAE4AFA`

本测试版实际签名证书 SHA-1：

`6F92BF00AF155C4ADA27936451F95ED2F703039C`

包名：`cc.forserein.simhub`。这些是公开的证书/应用标识，不是私钥。若该型号支持卡商或卡片管理员维护访问规则，可以将这些信息用于授权申请；本次没有读取或修改卡片规则，不能承诺该卡支持自行添加授权。

## 本次修正

1. 打开通道失败时显示实际 AID、发生阶段、`SecurityException` 类型和平台原始拒绝原因。平台未提供消息时明确注明；清除控制字符、限制消息长度。普通传输错误、AID 不存在、无可用通道继续分别处理，不改报成功，也不自动重复 APDU。
2. 本地卡槽配置列表为空且没有 EID、或存在管理读取错误时，改为“尚未读取到配置”，不再断言卡内没有 Profile；该错误状态中不显示原空卡卡片的下载按钮。成功读取后确实为空时保留原空卡界面，外接读卡器界面保持原样。
3. 保留 fix1 的测试证书自检修正。它解决的是应用自己的签名检查；Android/卡片的访问规则仍独立生效。

## 下一步需要的证据

- 卡片品牌、型号，以及能工作的 EasyEUICC 版本/来源；若使用了自定义管理 AID，也需对应的 AID。
- fix2 中显示的 Android 原始拒绝原因。

无需提供 EID、ICCID、IMSI、激活码或认证数据。若最终确认是访问规则未授权，需要卡片方授予当前证书对目标 AID 的访问；修改 APK 的提示、包名或复制 LPA 代码不能代替这项授权。

fix2 是错误定位和状态显示修正，不是已经解决卡片授权的版本。131 项 JVM 断言、10 个切卡流程模拟及 3 项签名交付回归用于本地验证，不代表真实卡片管理或通信已经成功。
