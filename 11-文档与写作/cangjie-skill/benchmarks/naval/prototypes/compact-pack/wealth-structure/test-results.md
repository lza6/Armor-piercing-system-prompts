# 测试结果 — wealth-structure

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「期权/股权」命中 description trigger (equity/期权) |
| should-trigger-02 | should_trigger | PASS | 「不靠出卖时间/被动收入」命中核心 trigger |
| should-trigger-03 | should_trigger | PASS | 「All in/积蓄全押」命中避免出局红线, description 含风险边界 |
| should-not-trigger-01 | should_not_trigger | PASS | 定位/适合做什么场景被 productize-yourself 捕获, 不激活本 skill |
| should-not-trigger-02 | should_not_trigger | PASS | 总结/信息查询不触发 |
| edge-01 | edge_case | PASS | description 明确「不推荐具体标的」, 判定符合预期 |

## 分析与回炉记录

- 无失败用例。
- 阶段 2 特意在 B 段写明「不提供投资标的建议」, 防止把结构框架误用于选股。
