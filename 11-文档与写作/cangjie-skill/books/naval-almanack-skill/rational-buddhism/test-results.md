# 测试结果 — rational-buddhism

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「能量疗愈该不该信」命中可证伪测试场景 |
| should-trigger-02 | should_trigger | PASS | 「科学和冥想冲突吗」命中理性佛教立场 |
| should-trigger-03 | should_trigger | PASS | 「建立该信什么的标准」命中验证流程 |
| should-not-trigger-01 | should_not_trigger | PASS | 「专家可不可信」被 judgment-training 捕获 |
| should-not-trigger-02 | should_not_trigger | PASS | 寺庙开放时间查询不触发 |
| edge-01 | edge_case | PASS | 临终信仰场景: 尊重他人信仰, 验证标准用于自己而非审判他人 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证 B 段「尊重信仰场景」边界, 防止理性主义被误用为审判工具。
