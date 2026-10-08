# 测试结果 — long-term-compounding

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「长期合伙/判断合作」命中 description (长期/合作/trust) |
| should-trigger-02 | should_trigger | PASS | 「快钱/伤口碑」命中复利/长期 trigger |
| should-trigger-03 | should_trigger | PASS | 「机会主动找我/声誉」命中 A2 场景 3 |
| should-not-trigger-01 | should_not_trigger | PASS | 生活朋友抱怨场景被 peer-selection 捕获, 不激活本 skill |
| should-not-trigger-02 | should_not_trigger | PASS | 技术查询不触发 |
| edge-01 | edge_case | PASS | 三个月好感场景, description 的「一辈子测试」给出审慎判定, 符合预期 |

## 分析与回炉记录

- 无失败用例。
- 跨 skill 诱饵 (生活同伴 vs 商业伙伴) 验证了与 peer-selection 的分工边界有效。
