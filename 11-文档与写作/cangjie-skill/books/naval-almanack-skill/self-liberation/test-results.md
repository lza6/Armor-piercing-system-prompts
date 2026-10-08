# 测试结果 — self-liberation

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「爸妈催婚/压力大」命中期望边界场景 |
| should-trigger-02 | should_trigger | PASS | 「容易生气/爆炸」命中愤怒解体场景 |
| should-trigger-03 | should_trigger | PASS | 「讨好别人/建立边界」命中核心场景 |
| should-not-trigger-01 | should_not_trigger | PASS | 「答应了做不到怎么说」被 honesty-communication 捕获 (表达问题) |
| should-not-trigger-02 | should_not_trigger | PASS | 订票任务不触发 |
| edge-01 | edge_case | PASS | 房贷+老板 PUA: 先算就业自由缓冲再谈辞职, 符合现实边界 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证「自由 vs 现实约束」的处理已写入 E 步骤 4 与 B 段。
