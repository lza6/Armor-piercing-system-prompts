# Phase 1 静态路由评测 — 50 条任务集（task-set-v2）

> 日期：2026-08-25 ｜ 方法：**静态自评、探索性**。逐条判断 expected 能力在各版本的
> 路由面上是否存在可达路径与匹配意图；不是宿主实测（宿主实测须锁定同一宿主同一版本，
> 由读者交叉试用与后续盲测完成）。**不作任何统计非劣声明。**
>
> 版本：A = `books/naval-almanack-skill`（19-Skill 基线，tag `baseline-naval-19skill-20260825`）；
> B = `dist/naval-almanack-single`；C = `dist/naval-almanack-pack`（B/C 均由
> `cangjie.py compile` 从同一份 Capability Bundle 生成，BUILD_MANIFEST 含发布哈希）。

## 汇总（静态判定口径：✅ 有明确可达路径且意图在路由面上；⚠️ 可达但近邻区分/判停质量待宿主实测；❌ 无运行时可达路径）

| 版本 | ✅ hit | ⚠️ 待实测 | ❌ miss | 说明 |
|---|---|---|---|---|
| A: 19-Skill 基线 | 28 | 13 | 9 | 9 条 miss 全部是 book_lookup（overview/glossary 不随 skill 安装）+ out_of_scope 无统一判停 3 条计入 ⚠️ |
| B: single | 42 | 8 | 0 | 8 条 ⚠️ 全部为 near_neighbor 的表内近邻区分 |
| C: compact pack | 42 | 8 | 0 | 同上；13 条 router-only 能力多一跳（路由入口 → 能力卡） |

与 Phase 0 的 20 条结论方向一致：**B/C 相对基线 A 的确定性优势是"非能力类查询与超范围判停有统一入口"**；
A 的 book_lookup 类 9 条在运行时没有任何可达入口（INDEX/GLOSSARY 是 pack 根文件，宿主不加载）。
near_neighbor 8 条（T16–T18、T43–T47）在三个版本中都标 ⚠️——静态分析无法替代宿主实测的近邻混淆率。

## A 类静态指标（正式编译产物，benchmarks/naval/metrics-v2/）

| 版本 | 入口数 | 发现负载 cl100k | 单任务负载 median | 语料总量 |
|---|---|---|---|---|
| A 基线 | 19 | 3,897 | 2,207 | 254,521 |
| B single | 1 | **340** | 4,534（约为 A 的 2 倍，预期内，§0） | 48,669 |
| C compact pack | 7 | 1,575 | 路由 4,761 / 晋级 2,271 | 62,558 |

结论口径不变：single 的确定性收益在**发现目录载荷**（340 vs 3,897）与入口认知成本；
单任务载荷 single 高于 pack 命中，是"主入口 + 能力卡"两跳的预期代价，不作为失败判据。

## 逐条判定

| 任务 | 类别 | expected | A: 19-Skill 基线 | B: single(编译) | C: compact pack(编译) |
|---|---|---|---|---|---|
| T01 | book_lookup | overview | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/overview.md | ✅ 路由入口 → references/overview.md |
| T02 | book_lookup | overview | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/overview.md | ✅ 路由入口 → references/overview.md |
| T03 | book_lookup | glossary | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/glossary.md | ✅ 路由入口 → references/glossary.md |
| T04 | book_lookup | glossary | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/glossary.md | ✅ 路由入口 → references/glossary.md |
| T05 | topic_consult | happiness-skill | ✅ happiness-skill 直接命中 | ✅ 入口 → 能力卡 happiness-skill | ✅ 路由入口 → 能力卡 happiness-skill；两跳 |
| T06 | topic_consult | game-selection | ✅ game-selection 直接命中 | ✅ 入口 → 能力卡 game-selection | ✅ 路由入口 → 能力卡 game-selection；两跳 |
| T07 | topic_consult | life-meaning | ✅ life-meaning 直接命中 | ✅ 入口 → 能力卡 life-meaning | ✅ 路由入口 → 能力卡 life-meaning；两跳 |
| T08 | topic_consult | rational-buddhism | ✅ rational-buddhism 直接命中 | ✅ 入口 → 能力卡 rational-buddhism | ✅ 路由入口 → 能力卡 rational-buddhism；两跳 |
| T09 | topic_consult | principal-agent | ✅ principal-agent 直接命中 | ✅ 入口 → 能力卡 principal-agent | ✅ 路由入口 → 能力卡 principal-agent；两跳 |
| T10 | executable_task | wealth-structure | ✅ wealth-structure 直接命中 | ✅ 入口 → 能力卡 wealth-structure | ✅ 晋级 Skill wealth-structure 直接命中 |
| T11 | executable_task | productize-yourself | ✅ productize-yourself 直接命中 | ✅ 入口 → 能力卡 productize-yourself | ✅ 晋级 Skill productize-yourself 直接命中 |
| T12 | executable_task | decision-heuristics | ✅ decision-heuristics 直接命中 | ✅ 入口 → 能力卡 decision-heuristics | ✅ 晋级 Skill decision-heuristics 直接命中 |
| T13 | executable_task | hourly-rate-time | ✅ hourly-rate-time 直接命中 | ✅ 入口 → 能力卡 hourly-rate-time | ✅ 晋级 Skill hourly-rate-time 直接命中 |
| T14 | executable_task | reading-metaskill | ✅ reading-metaskill 直接命中 | ✅ 入口 → 能力卡 reading-metaskill | ✅ 晋级 Skill reading-metaskill 直接命中 |
| T15 | executable_task | screen-detox | ✅ screen-detox 直接命中 | ✅ 入口 → 能力卡 screen-detox | ✅ 晋级 Skill screen-detox 直接命中 |
| T16 | near_neighbor | acceptance | ⚠️ acceptance 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 acceptance；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 acceptance；两跳 + 近邻区分待实测 |
| T17 | near_neighbor | self-liberation | ⚠️ self-liberation 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 self-liberation；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 self-liberation；两跳 + 近邻区分待实测 |
| T18 | near_neighbor | monkey-mind-meditation | ⚠️ monkey-mind-meditation 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 monkey-mind-meditation；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 monkey-mind-meditation；两跳 + 近邻区分待实测 |
| T19 | out_of_scope | out-of-scope | ⚠️ 只能依赖各 skill 的 B 段边界，无统一判停声明 | ✅ 入口「不适用」清单 + 边界与判停 | ✅ 路由入口「不适用」清单 + 边界与判停 |
| T20 | out_of_scope | out-of-scope | ⚠️ 只能依赖各 skill 的 B 段边界，无统一判停声明 | ✅ 入口「不适用」清单 + 边界与判停 | ✅ 路由入口「不适用」清单 + 边界与判停 |
| T21 | book_lookup | overview | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/overview.md | ✅ 路由入口 → references/overview.md |
| T22 | book_lookup | glossary | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/glossary.md | ✅ 路由入口 → references/glossary.md |
| T23 | book_lookup | glossary | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/glossary.md | ✅ 路由入口 → references/glossary.md |
| T24 | book_lookup | overview | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/overview.md | ✅ 路由入口 → references/overview.md |
| T25 | book_lookup | glossary | ❌ 无运行时入口（INDEX/GLOSSARY 是 pack 根文件，不随任何 skill 安装） | ✅ 入口「非能力类查询」→ references/glossary.md | ✅ 路由入口 → references/glossary.md |
| T26 | book_lookup | peer-selection | ✅ peer-selection 直接命中 | ✅ 入口 → 能力卡 peer-selection | ✅ 路由入口 → 能力卡 peer-selection；两跳 |
| T27 | topic_consult | principal-agent | ✅ principal-agent 直接命中 | ✅ 入口 → 能力卡 principal-agent | ✅ 路由入口 → 能力卡 principal-agent；两跳 |
| T28 | topic_consult | long-term-compounding | ✅ long-term-compounding 直接命中 | ✅ 入口 → 能力卡 long-term-compounding | ✅ 路由入口 → 能力卡 long-term-compounding；两跳 |
| T29 | topic_consult | long-term-compounding | ✅ long-term-compounding 直接命中 | ✅ 入口 → 能力卡 long-term-compounding | ✅ 路由入口 → 能力卡 long-term-compounding；两跳 |
| T30 | topic_consult | identity-work | ✅ identity-work 直接命中 | ✅ 入口 → 能力卡 identity-work | ✅ 路由入口 → 能力卡 identity-work；两跳 |
| T31 | topic_consult | peer-selection | ✅ peer-selection 直接命中 | ✅ 入口 → 能力卡 peer-selection | ✅ 路由入口 → 能力卡 peer-selection；两跳 |
| T32 | topic_consult | judgment-training | ✅ judgment-training 直接命中 | ✅ 入口 → 能力卡 judgment-training | ✅ 路由入口 → 能力卡 judgment-training；两跳 |
| T33 | topic_consult | honesty-communication | ✅ honesty-communication 直接命中 | ✅ 入口 → 能力卡 honesty-communication | ✅ 路由入口 → 能力卡 honesty-communication；两跳 |
| T34 | topic_consult | monkey-mind-meditation | ✅ monkey-mind-meditation 直接命中 | ✅ 入口 → 能力卡 monkey-mind-meditation | ✅ 路由入口 → 能力卡 monkey-mind-meditation；两跳 |
| T35 | executable_task | wealth-structure | ✅ wealth-structure 直接命中 | ✅ 入口 → 能力卡 wealth-structure | ✅ 晋级 Skill wealth-structure 直接命中 |
| T36 | executable_task | wealth-structure | ✅ wealth-structure 直接命中 | ✅ 入口 → 能力卡 wealth-structure | ✅ 晋级 Skill wealth-structure 直接命中 |
| T37 | executable_task | hourly-rate-time | ✅ hourly-rate-time 直接命中 | ✅ 入口 → 能力卡 hourly-rate-time | ✅ 晋级 Skill hourly-rate-time 直接命中 |
| T38 | executable_task | reading-metaskill | ✅ reading-metaskill 直接命中 | ✅ 入口 → 能力卡 reading-metaskill | ✅ 晋级 Skill reading-metaskill 直接命中 |
| T39 | executable_task | screen-detox | ✅ screen-detox 直接命中 | ✅ 入口 → 能力卡 screen-detox | ✅ 晋级 Skill screen-detox 直接命中 |
| T40 | executable_task | decision-heuristics | ✅ decision-heuristics 直接命中 | ✅ 入口 → 能力卡 decision-heuristics | ✅ 晋级 Skill decision-heuristics 直接命中 |
| T41 | executable_task | productize-yourself | ✅ productize-yourself 直接命中 | ✅ 入口 → 能力卡 productize-yourself | ✅ 晋级 Skill productize-yourself 直接命中 |
| T42 | executable_task | judgment-training | ✅ judgment-training 直接命中 | ✅ 入口 → 能力卡 judgment-training | ✅ 路由入口 → 能力卡 judgment-training；两跳 |
| T43 | near_neighbor | game-selection | ⚠️ game-selection 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 game-selection；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 game-selection；两跳 + 近邻区分待实测 |
| T44 | near_neighbor | acceptance | ⚠️ acceptance 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 acceptance；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 acceptance；两跳 + 近邻区分待实测 |
| T45 | near_neighbor | monkey-mind-meditation | ⚠️ monkey-mind-meditation 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 monkey-mind-meditation；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 monkey-mind-meditation；两跳 + 近邻区分待实测 |
| T46 | near_neighbor | self-liberation | ⚠️ self-liberation 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 self-liberation；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 self-liberation；两跳 + 近邻区分待实测 |
| T47 | near_neighbor | happiness-skill | ⚠️ happiness-skill 直接命中；近邻竞争待实测 | ⚠️ 入口 → 能力卡 happiness-skill；表内近邻意图区分待实测 | ⚠️ 路由入口 → 能力卡 happiness-skill；两跳 + 近邻区分待实测 |
| T48 | out_of_scope | out-of-scope | ⚠️ 只能依赖各 skill 的 B 段边界，无统一判停声明 | ✅ 入口「不适用」清单 + 边界与判停 | ✅ 路由入口「不适用」清单 + 边界与判停 |
| T49 | out_of_scope | out-of-scope | ⚠️ 只能依赖各 skill 的 B 段边界，无统一判停声明 | ✅ 入口「不适用」清单 + 边界与判停 | ✅ 路由入口「不适用」清单 + 边界与判停 |
| T50 | out_of_scope | out-of-scope | ⚠️ 只能依赖各 skill 的 B 段边界，无统一判停声明 | ✅ 入口「不适用」清单 + 边界与判停 | ✅ 路由入口「不适用」清单 + 边界与判停 |

## 待宿主实测项（进入读者交叉试用 / 盲测）

1. near_neighbor 8 条的实际路由命中与兄弟混淆率（三版本同一宿主各跑 ≥1 轮）；
2. C 版 router-only 能力的"两跳"是否造成可感知的时延/失败；
3. out_of_scope 3 条在 A 版是否实际乱触发（静态只能标 ⚠️）。
