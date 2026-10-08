# 测试结果 — productize-yourself

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「不可替代性/独特优势」命中 description trigger (moat/special knowledge/找方向) |
| should-trigger-02 | should_trigger | PASS | 「副业方向」命中 A2 场景 1, description 含「副业/自由职业」 |
| should-trigger-03 | should_trigger | PASS | 英文 prompt 命中中英双写 trigger (moat/productize) |
| should-not-trigger-01 | should_not_trigger | PASS | 投资/All in 场景被 description 的 wealth-structure 边界捕获, 不激活本 skill |
| should-not-trigger-02 | should_not_trigger | PASS | 信息查询, description 明确「不适用于执行细节/查询」 |
| edge-01 | edge_case | PASS | description 区分「定位」与「简历执行」, 判定符合预期 |

## 分析与回炉记录

- 无失败用例, 无需回炉。
- 阶段 2 已把「何时不调用」(纯求职/已有明确方向) 写入 description 与 B 段, 防止过度激活。
