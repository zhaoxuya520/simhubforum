# Wi-Fi 通话媒体协议审查

本轮对照 [RFC 3550 Appendix A.1](https://www.rfc-editor.org/rfc/rfc3550#appendix-A.1)、[RFC 4867](https://www.rfc-editor.org/rfc/rfc4867)，以及 PJSIP 固定提交 `0e62a9d6498d91c475bcc9f98329da329836ef35` 的 [RTP 序号处理](https://github.com/pjsip/pjproject/blob/0e62a9d6498d91c475bcc9f98329da329836ef35/pjmedia/src/pjmedia/rtp.c) 和 [AMR 负载解析](https://github.com/pjsip/pjproject/blob/0e62a9d6498d91c475bcc9f98329da329836ef35/pjmedia/include/pjmedia-codec/amr_helper.h)。参考文件及 SHA-256 记录在 `validation/native-amr/protocol-review.json`，没有复制其实现源码进应用。

## 发现与修复

1. **坏语音包改变当前发送封装。** 23.2 的 AMR 检测先更新 SSRC 和序号，再验证负载。带其他 SSRC 的截断包会让已确认的封装退回 SDP；同 SSRC 下带异常大序号的坏包也可能占用序号位置。原生抖动缓冲并不接受这些坏包，造成两边状态不一致。现在先校验负载，失败不会换来源或推进序号；同来源的坏包只中断尚未完成的格式确认。

2. **同 SSRC 重启后自适应可能长时间停滞。** 原来的半区间判断将大幅回退序号视为迟到包，原生抖动缓冲却已经接收重启后的新流。如果实际封装改变，接收能解出语音，但发送封装无法及时跟进。现在采用 RFC 3550/PJSIP 的 3000 包正常跨度、100 包乱序窗口，在大幅跳变时以两个连续新序号确认重启；之后仍需三个有效语音观察才调整发送封装。一个离群包、其副本或普通迟到包都不能单独触发调整。

修复位于 `AmrFraming` 辅助类，沿用原 RTP 解析、抖动缓冲、AMR 编解码器和发送器；没有修改 SDP 或卡片身份参数。两个连续重启序号仅决定哪些包可计入封装确认，不改变原生抖动缓冲的接收、丢弃、重排或重启策略；可验证的语音负载仍立即交给原生路径。

## 回归范围

实际 APK 探针增加：已适配通话收到其他 SSRC 的坏包、同 SSRC 的坏序号包、同 SSRC 大幅向前及向后重启，并检查原生抖动队列与最终发送封装。`--stream-legacy` 以已发布 23.2 为反例；默认模式检查修复版。单元测试额外覆盖重复重启候选、原流恢复取消候选及后续重启再确认。

这些测试使用合成语音数据与音频/网络替身，不是运营商通话或手机听感测试。此审查不表示所有媒体问题已找完。
