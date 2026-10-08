# 测试结果 — hourly-rate-time

- **测试方式**: 降级自测 — 主流程对照 `test-prompts.json` 逐条判定 (独立 sub-agent 盲测因环境消息投递故障不可用, 本结果可信度低于盲测)
- **测试时间**: 2026-08-01
- **用例数**: 6 | **通过**: 6 | **通过率**: 100%

| id | 类型 | 判定 | 说明 |
|---|---|---|---|
| should-trigger-01 | should_trigger | PASS | 「比价/省几十块」命中时薪/琐事 trigger |
| should-trigger-02 | should_trigger | PASS | 「外包/请人」命中 delegation trigger |
| should-trigger-03 | should_trigger | PASS | 「财务自由/退休」命中 retirement trigger |
| should-not-trigger-01 | should_not_trigger | PASS | offer 选择场景被 decision-heuristics 捕获 |
| should-not-trigger-02 | should_not_trigger | PASS | 科普查询不触发 |
| edge-01 | edge_case | PASS | 陪家人=非浪费, description 的「做想做的事就不是浪费」边界生效 |

## 分析与回炉记录

- 无失败用例。
- edge-01 验证了「时间资产」与「享受生活」的边界已写入 description, 避免把休息误判为低效。
