# 测试结果 — decision-heuristics

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「跳槽/拿不定主意/利弊表」命中核心 trigger |
| should-trigger-02 | should_trigger | PASS | 「搬城市/影响十年」命中三个重大决定场景 |
| should-trigger-03 | should_trigger | PASS | 「前期痛苦长期有价值」命中短期痛苦原则 |
| should-not-trigger-01 | should_not_trigger | PASS | 关系/改不了习惯场景被 acceptance 捕获, 不激活本 skill |
| should-not-trigger-02 | should_not_trigger | PASS | 天气查询不触发 |
| edge-01 | edge_case | PASS | 「晚饭吃什么」被 description 明确排除 (日常琐碎选择) |

## 分析与回炉记录

- 无失败用例。
- 阶段 2 在 description/B 段明确「不适用于日常琐碎选择」, edge-01 验证有效。
