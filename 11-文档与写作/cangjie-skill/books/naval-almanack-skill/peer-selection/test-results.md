# 测试结果 — peer-selection

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「最好的朋友总抱怨/要不要疏远」命中五只黑猩猩 |
| should-trigger-02 | should_trigger | PASS | 「新城市怎么交朋友」命中主动选择场景 |
| should-trigger-03 | should_trigger | PASS | 「选伴侣最重要是什么」命中价值观一致性 |
| should-not-trigger-01 | should_not_trigger | PASS | 投资人合作被 long-term-compounding 捕获 (商业伙伴) |
| should-not-trigger-02 | should_not_trigger | PASS | 健身房查询不触发 |
| edge-01 | edge_case | PASS | 无法换组的同事: 给边界而非切割, 符合预期 |

## 分析与回炉记录

- 无失败用例。
- 与 long-term-compounding 的「生活同伴 vs 商业伙伴」分工在诱饵测试中确认有效。
