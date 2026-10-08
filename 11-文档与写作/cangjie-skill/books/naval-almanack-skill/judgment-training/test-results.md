# 测试结果 — judgment-training

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「快速建立判断力/不被忽悠」命中核心 trigger |
| should-trigger-02 | should_trigger | PASS | 「真懂还是包装」命中从基础重建场景 |
| should-trigger-03 | should_trigger | PASS | 「提升决策能力」命中 A2 场景 |
| should-not-trigger-01 | should_not_trigger | PASS | 风水/转运场景被 rational-buddhism 捕获 (验证标准) |
| should-not-trigger-02 | should_not_trigger | PASS | 编码任务不触发 |
| edge-01 | edge_case | PASS | 导师意见场景: description 平衡「尊重权威 vs 独立验证」, 判定符合预期 |

## 分析与回炉记录

- 无失败用例。
- 与 rational-buddhism 的边界 (能力 vs 标准) 在诱饵测试中确认有效。
