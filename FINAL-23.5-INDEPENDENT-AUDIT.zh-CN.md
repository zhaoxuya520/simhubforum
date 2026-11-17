# 23.4 → 23.5 独立发布范围审计

结果：通过；当前阶段：最终签名包。未修改生产文件。

已有 6,624 类中恰好 7 类变化，6,617 类逐字节相同；新增 `IdentityBinding$Read` 1 类，无删除、无额外变化。

- `DitoHomeIdentity`、`IdentityBinding`、`IdentityBinding$Selection`、`WifiConfigRuntime` 是本次预期逻辑变化。
- 从实际旧签名 APK 重新解码确认：`DitoHomeIdentity$1` 只有两处 `.line 112 → 177`，`WifiConfigRuntime$Snapshot` 只有一处 `.line 147 → 182`。仅移除 `.line` 后全文件严格一致，指令、字段、方法签名和异常结构均不变。
- `r75` 只把版本显示文字 23.4 改为 23.5。
- 两包各 1,711 个 ZIP 条目，仅 Manifest 与 classes.dex 变化；16 个 native 库、991 项资源、579 项 assets 全相同。
- Manifest 仅 versionCode 13→14、versionName 23.4→23.5。权限、组件及其它属性未变。
- 候选 DEX 与当前审计 Smali 的组装产物完全相同。

因此未发现预期外的 APDU、切卡、媒体、SIP、UI 或配置资源变化。本报告验证范围和产物一致性，不代替实际功能或手机测试。

旧签名 APK SHA-256：`77774c46dbf8d828c7daa7f895306f172fa96e30f630ac83c95c5dd4a11d0eac`。
unsigned APK SHA-256：`a28d913a9bae2e6689534b7425be5e2be86ff4ffc565f1b8483f8220496a188a`。
最终签名 APK SHA-256：`c45ec64fdf6069688e3a59f756baadf1a80afd98e899ca0ad8a2e9fbfa7242f2`。签名包全部 ZIP payload 与 unsigned 包完全一致。

完整证据：`final-23.5-independent-audit.json`；可复现脚本：`independent-release-scope-audit.py`；全量新类 hash：`current-23.5-classes.json`。

额外签名核验：apksigner 对 23.4 和 23.5 均验证通过（v2、v3）；签名证书 SHA-256 一致，为 `50fe8b357efb2f2397b1f822012a5d7e48a4fc1caf5a0f8a4f0239ffadab3c7e`。输出存于 `signature-23.4.log` / `signature-23.5.log`。
