# 测试结果 — principal-agent

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「大公司没劲/小团队拼」命中委托代理核心场景 |
| should-trigger-02 | should_trigger | PASS | 「分成/激励设计」命中 incentive trigger |
| should-trigger-03 | should_trigger | PASS | 「为谁的利益工作/代理人」命中职业诊断场景 |
| should-not-trigger-01 | should_not_trigger | PASS | 「怎么致富」被 wealth-structure 捕获, 不激活本 skill |
| should-not-trigger-02 | should_not_trigger | PASS | 概念查询不触发 |
| edge-01 | edge_case | PASS | 「外包偷工减料」判定为结构解法 (改激励) 而非纯监督, 符合预期 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证 B 段「先改结构再谈管理」的边界有效。
