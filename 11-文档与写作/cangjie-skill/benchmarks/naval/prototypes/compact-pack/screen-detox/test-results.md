# 测试结果 — screen-detox

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「刷短视频上瘾」命中核心场景 |
| should-trigger-02 | should_trigger | PASS | 「睡前不刷难受」命中习惯替换五步 |
| should-trigger-03 | should_trigger | PASS | 「被算法控制/时间还给自己」命中数字自主场景 |
| should-not-trigger-01 | should_not_trigger | PASS | 「多读点书/读不进去」被 reading-metaskill 捕获 |
| should-not-trigger-02 | should_not_trigger | PASS | 电影排片查询不触发 |
| edge-01 | edge_case | PASS | 靠屏幕工作: 区分工作/消费屏幕, 符合预期 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证「工作屏幕 vs 消费屏幕」边界已写入 B 段。
