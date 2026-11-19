# SIMHub Android 24.2

[下载 24.2 安装包](SIMHub-24.2-signed.apk)

可直接覆盖同签名的 24.1，原有逐卡配置继续保留。

- 防止已接通通话被迟到的失败、振铃或会话进度响应误改状态；真正的忙线等失败保留原状态码。
- 接通前，重复或乱序的可靠临时响应不再反复覆盖媒体协商结果。
- 原生流程在 TCP 建连失败后改用 UDP 时，后续取消和失败确认沿用原请求实际使用的传输信息，并恢复 UDP 重传节奏，保留原截止时间。

主机回归通过 5852 条断言。Android 11 模拟器覆盖安装及冷启动通过，275 项离线 ART 检查通过，设备侧 APK 哈希与交付文件一致，未发现应用崩溃或 VerifyError，详见 `emulator-validation.json`。

一加 13 与真实运营商通话尚未验证；以上为离线测试，运营商导致的失败仍需实卡排查。

[补丁源码](patch-source.zip) · [版本对比](previous-delivery-comparison.json) · [签名验证](longterm-signed-validation.json) · [呼叫状态回归](signed-invite-failure.json) · [传输回归](signed-invite-transport.json)

eSTK ARA-M 证书 SHA-1 为 `CAEAEE369CA8933EC6A80050ECAF22D87B155006`。

APK SHA-256 为 `40471d65d554e0a76214efc98f1810d577246a0dcfd75c9766b349155927e0d9`。
