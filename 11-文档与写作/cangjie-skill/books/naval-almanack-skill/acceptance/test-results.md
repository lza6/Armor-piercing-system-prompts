# 测试结果 — acceptance

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「改不了又走不了/内耗」命中三选项场景 |
| should-trigger-02 | should_trigger | PASS | 「放不下过去」命中接受/重释场景 |
| should-trigger-03 | should_trigger | PASS | 「该忍还是该走」命中三选项筛查 |
| should-not-trigger-01 | should_not_trigger | PASS | 可改变的选择 (offer) 被 decision-heuristics 捕获 |
| should-not-trigger-02 | should_not_trigger | PASS | 写作任务不触发 |
| edge-01 | edge_case | PASS | 暴力场景: 明确安全优先、不接受, 符合安全红线 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证「接受不适用于人身安全场景」已写入 B 段, 是重要安全边界。
