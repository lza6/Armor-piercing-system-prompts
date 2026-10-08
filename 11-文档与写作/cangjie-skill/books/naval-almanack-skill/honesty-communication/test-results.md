# 测试结果 — honesty-communication

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「批评同事不伤关系」命中具体表扬一般批评 |
| should-trigger-02 | should_trigger | PASS | 「场面话/真诚拒绝」命中诚实沟通场景 |
| should-trigger-03 | should_trigger | PASS | 「骗自己/怎么纠正」命中撒谎先骗自己机制 |
| should-not-trigger-01 | should_not_trigger | PASS | 「讨好别人」被 self-liberation 捕获 (边界问题) |
| should-not-trigger-02 | should_not_trigger | PASS | 压缩视频不触发 |
| edge-01 | edge_case | PASS | 「新发型好不好看」: 诚实+积极可同时成立, 判定符合预期 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证「诚实≠口无遮拦」边界已写入 I 段与 B 段。
