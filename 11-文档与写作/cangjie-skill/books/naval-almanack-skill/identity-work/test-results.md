# 测试结果 — identity-work

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「为立场辩护/不肯松口」命中身份锁死认知场景 |
| should-trigger-02 | should_trigger | PASS | 「下定决心又坚持不下去」命中自我形象替代自律 |
| should-trigger-03 | should_trigger | PASS | 「自欺欺人/面对真相」命中痛苦=真相时刻 |
| should-not-trigger-01 | should_not_trigger | PASS | 老板画饼/外部期望被 self-liberation 捕获 |
| should-not-trigger-02 | should_not_trigger | PASS | U 盘格式化不触发 |
| edge-01 | edge_case | PASS | 「出柜/转行家人不认同」: 明确区分身份清空与身份认同, 保护用户核心身份 |

## 分析与回炉记录

- 无失败用例。
- edge-01 是关键边界: B 段已注明「身份清空针对认知牢笼而非真实身份认同」, 防止误用为否定自我。
