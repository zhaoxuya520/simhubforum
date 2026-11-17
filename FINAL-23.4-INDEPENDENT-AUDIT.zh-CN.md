# 23.4 独立发布范围审计

结果：通过。全部操作只读，不修改生产代码。

- 基线 APK SHA-256：`4a83e74a5e207fc1ab23085ba4a9558375d2758c3fdf5b3f8dc6c3cc287d805e`，与预先保存的 23.3 清单一致。
- 23.4 签名 APK SHA-256：`77774c46dbf8d828c7daa7f895306f172fa96e30f630ac83c95c5dd4a11d0eac`。
- 原 6,618 个 Smali 类中，6,610 个逐字节相同。恰好 8 个预期已有类变化，没有额外变化或删除。
- 新增恰好 6 个预期 helper/内部类。
- 两个 APK 各有 1,711 个 ZIP 条目；1,709 个内容相同，仅 `AndroidManifest.xml` 与 `classes.dex` 变化，无新增/删除条目。
- Manifest 仅 versionCode 12→13、versionName 23.3→23.4；权限、组件、网络及其它属性未变。
- 23.4 签名包的每个 ZIP payload 与构建 unsigned 包完全一致；其 classes.dex 与本次审计 Smali 组装产物完全一致。
- 所有 native 库（16 项）、resources（991 项）及 assets（579 项）均与 23.3 逐字节相同。

已有类变化：

- `au0.smali`
- `cc/forserein/simhub/ui/call/CallActivity.smali`
- `cc/forserein/simhub/vowifi/calling/CallActionReceiver.smali`
- `ej.smali`
- `k10.smali`
- `o82.smali`
- `r75.smali`
- `sj5.smali`

新增类：

- `cc/forserein/simhub/internal/CallNotificationGuard$Action.smali`
- `cc/forserein/simhub/internal/CallNotificationGuard.smali`
- `cc/forserein/simhub/internal/DiagnosticShareSanitizer.smali`
- `cc/forserein/simhub/internal/SipForkProtection$Bucket.smali`
- `cc/forserein/simhub/internal/SipForkProtection$Entry.smali`
- `cc/forserein/simhub/internal/SipForkProtection.smali`

这验证了发布变更范围：未发现预期外的 UI、配置编辑、卡片切换、AMR/音频媒体或 native 库修改。允许清单中的 CallActivity/通知动作类确有预期行为修改，因此不能笼统称所有 UI 相关类未变。这不是功能、ART 或实机通话验证；实际行为测试由对应回归报告提供。

完整 JSON：`final-23.4-independent-audit.json`；当前全部类 hash：`current-23.4-classes.json`；两版 aapt manifest 解码：`final-audit-manifest-23.3.txt` / `final-audit-manifest-23.4.txt`。
