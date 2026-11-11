# 跨芯片旧配置无法禁用：刷新回退修复

用户确认：切换后 Wi-Fi 通话能读到新卡；故障是另一芯片旧配置仍启用，无法直接禁用，只有启用其同芯片另一配置才会自动关闭旧配置。身份读取不是本次修复重点。此前身份缓存修复没有解决该问题。

对照 EasyEUICC / OpenEUICC b79e03bbf42f4d69b9d9dd21cd6132bb1b9314d7 的 EuiccChannelManagerService.launchProfileSwitchTask（523–537 行）：带刷新切卡失败后，使用同一 slot / port / seId / ICCID 以 refresh=false 重试。底层 lpac 50cdf15ab40942b00f8819a1c2ca19997f1a6d34 的 es10c.c 以 BF31/BF32 + A0/5A + 81 BOOLEAN 构造启用/禁用命令。

当前 SIMHub 的 rf1 固定手机卡槽 refreshFlag=true。新增 InternalSimSwitchRequest 包装原有请求：

1. 首次请求沿用原参数。
2. pg1 解析到 BF31/BF32 的明确非零卡片结果时保留 operation / result，手机卡槽仅重试一次 refreshFlag=false。USB/蓝牙不重试；取消、超时、断连、缺少响应或未知响应不重试。
3. 回退使用同一 backend、ICCID、操作类型，不切换到其他芯片，不启用替代配置，不更改昵称或 IMSI。
4. 沿用原有状态读回；跨芯片操作必须确认旧配置禁用后才能启用目标，并最终核对全部芯片状态。读回不一致不能报告完成。
5. 回退在原日志提示可能需要飞行模式刷新手机，不保证 modem 已刷新。没有新增 UI 控件。

自动验证覆盖同步/挂起首次成功、明确拒绝后一次回退、回退再次失败、传输错误和取消不重试、USB 不重试，以及原有独立禁用确认和切卡事务。APK 使用既有长期证书签名。尚未进行用户设备实测，不能据此确认其卡片故障根因或宣称实际跨芯片切换已解决。
