# Phase 0 路由评测（探索性，20 任务 × 3 版本）

- 任务集: `task-set-v1.json`（20 条，5 类）
- 评测方式: **静态路由判定 + 主流程自评**——依据各版本入口 description 与路由表，判断「哪个入口会触发、最终由哪张能力卡/哪个文件承接」。与方案 v1.3 §10.2 的定位一致：探索性、单次、不做统计推断；正式结论需盲测 runner 与真实宿主触发（Phase 2）。
- 宿主锁定: 本轮为静态判定，未涉及真实宿主触发。真实宿主交叉试用（读者测试）待用户执行，须记录宿主名与版本。
- 评测日期: 2026-08-25；评测者: 主流程 Agent 自评（已知全部原型内容，存在乐观偏差，见文末声明）

## 判定结果

图例: ✅ 命中预期 | 🟡 命中 acceptable 或部分承接 | ❌ 无承接/错误承接 | ⛔ 正确判停（超范围题的期望行为）

| 任务 | 类别 | A 原版 19 入口 | B single 单入口 | C compact pack (7 入口) |
|---|---|---|---|---|
| T01 作者/整书 | 书查询 | ❌ 无任何入口覆盖书级查询 | ✅ overview | ✅ 路由入口→overview |
| T02 第一章观点 | 书查询 | 🟡 wealth-structure 部分承接 | ✅ 核心原则+overview | ✅ 路由入口→overview |
| T03 特殊知识定义 | 书查询 | 🟡 productize-yourself 内容含定义 | ✅ glossary | ✅ 路由入口→glossary |
| T04 杠杆分类 | 书查询 | 🟡 wealth-structure 内容含分类 | ✅ glossary | ✅ 路由入口→glossary |
| T05 越赚越焦虑 | 主题咨询 | ✅ happiness-skill | ✅ →happiness 卡 | ✅ 路由→happiness 卡 |
| T06 升职妒忌 | 主题咨询 | ✅ game-selection | ✅ →game-selection 卡 | ✅ 路由→game-selection 卡 |
| T07 活着没意义 | 主题咨询 | ✅ life-meaning | ✅ →life-meaning 卡 | ✅ 路由→life-meaning 卡 |
| T08 身心灵该不该信 | 主题咨询 | ✅ rational-buddhism | ✅ →rational-buddhism 卡 | ✅ 路由→rational-buddhism 卡 |
| T09 磨洋工 vs 拼命 | 主题咨询 | ✅ principal-agent | ✅ →principal-agent 卡 | ✅ 路由→principal-agent 卡 |
| T10 高薪 vs 期权 offer | 可执行 | ✅ wealth-structure | ✅ →wealth-structure 卡 | ✅ 晋级 wealth-structure |
| T11 副业找优势 | 可执行 | ✅ productize-yourself | ✅ →productize 卡 | ✅ 晋级 productize-yourself |
| T12 搬城决定 | 可执行 | ✅ decision-heuristics | ✅ →decision 卡 | ✅ 晋级 decision-heuristics |
| T13 琐事外包 | 可执行 | ✅ hourly-rate-time | ✅ →hourly-rate 卡 | ✅ 晋级 hourly-rate-time |
| T14 入门经济学 | 可执行 | ✅ reading-metaskill | ✅ →reading 卡 | ✅ 晋级 reading-metaskill |
| T15 戒短视频 | 可执行 | ✅ screen-detox | ✅ →screen-detox 卡 | ✅ 晋级 screen-detox |
| T16 忍还是搬（近邻） | 近邻 | ✅ acceptance（self-liberation 有混淆风险） | ✅ 路由表意图行精确→acceptance 卡 | ✅ 路由→acceptance 卡 |
| T17 控制不住发火（近邻） | 近邻 | ✅ self-liberation | ✅ →self-liberation 卡 | ✅ 路由→self-liberation 卡 |
| T18 睡前反刍（近邻） | 近邻 | ✅ monkey-mind-meditation | ✅ →monkey-mind 卡 | ✅ 路由→monkey-mind 卡 |
| T19 股票买入分析 | 超范围 | ⛔ 无触发（各卡「不适用」明确排除） | ⛔ 入口 out_of_scope 判停 | ⛔ 同 B |
| T20 失眠心悸问药 | 超范围 | ⛔ 无触发（临床排除） | ⛔ 入口 out_of_scope 判停 | ⛔ 同 B |

## 汇总

| 版本 | ✅ | 🟡 | ❌ | ⛔ 正确判停 | 说明 |
|---|---|---|---|---|---|
| A 原版 | 14 | 3 | 1 | 2 | 书级查询是结构性盲区：INDEX/GLOSSARY/OVERVIEW 不随 Skill 安装 |
| B single | 18 | 0 | 0 | 2 | 静态判定全命中；**真实触发率是最大未知**（见下） |
| C compact | 18 | 0 | 0 | 2 | 六个高频可执行意图走晋级 Skill，负载与 A 持平 |

## 必须诚实声明的三个偏差

1. **评测者偏差**：路由表由我编写、评测也由我判定，B/C 的全命中含乐观成分。正式评测需要盲测 runner（Phase 2 的 run_eval.py）与未参与编写的出题人。
2. **静态判定测不出真实触发率**：A 的 19 条 description 合计约 3.9k token 触发面，B 只有 340 token。对不含「纳瓦尔」关键词的裸意图（如 T05、T17），B 的入口在真实宿主里**是否会被激活**无法静态验证——这是 single 模式最大的实证风险，正是读者测试要回答的问题。
3. **单次判定无统计效力**：20 条、单次、无重复采样，只能用于粗筛与证伪「明显更差」，不能证明「不劣于」。
