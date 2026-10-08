# Phase 0 基线报告 — Naval 试点（single vs pack 低成本证伪）

- 日期: 2026-08-25
- 依据方案: `docs/plans/2026-08-23-cangjie-skill-optimization-plan.md`（v1.3.1）
- 状态: **步骤 1—5、7 完成；读者交叉试用（步骤 6）待用户执行**

## 1. 基线冻结

| 项 | 值 |
|---|---|
| 冻结分支 | `baseline/naval-freeze-20260825`（基于 `origin/main` `a47a604`，独立 worktree `../cangjie-baseline`） |
| 冻结提交 | `f59f64c`（74 文件：19 Skill × 3 文件 + 审计文件 + SHA-256 清单） |
| 本地 tag | `baseline-naval-19skill-20260825`（**仅本地，未推送**） |
| 完整性 | `books/naval-almanack-skill.SHA256SUMS` 73 条校验和全部通过 |
| 静态校验 | `validate_skill_pack.py`: 19 SKILL.md / 54 md，0 errors 0 warnings |

## 2. 产出物

| 产物 | 路径 |
|---|---|
| 人工能力映射（唯一判断产物） | `benchmarks/naval/capability-map.yaml`（19 能力：id/重要度+依据/意图/关键词/一句话规则/晋级判定） |
| single 原型 | `benchmarks/naval/prototypes/single/naval-almanack/`（1 入口 + 19 能力卡 + overview/glossary/cheatsheet/索引 + BUILD_MANIFEST） |
| compact pack 原型 | `benchmarks/naval/prototypes/compact-pack/`（1 路由入口 + 6 晋级 Skill + `capability-destinations.json`） |
| Phase 0 工具（纯确定性） | `scripts/validate_skill_pack.py`、`scripts/compile_single.py`、`scripts/count_tokens.py` |
| A 类静态指标 | `benchmarks/naval/metrics/a-class-metrics.{json,md}` |
| 任务集与路由评测 | `benchmarks/naval/task-set-v1.json`、`benchmarks/naval/phase0-routing-eval.md` |

**晋级门评审结果**（预算 ≤8 入口，实际 7）：晋级 `wealth-structure`、`productize-yourself`、`decision-heuristics`、`hourly-rate-time`、`reading-metaskill`、`screen-detox`；其余 13 能力留路由域（幸福/接受/冥想等近邻域按 §4.6.5 统一路由，等 split-on-evidence）。逐条依据见 capability-map.yaml 的 `promotion_notes`。

## 3. A 类静态指标（tokenizer=cl100k_base；o200k 结论方向一致）

| 指标 | A 原版 19 Skill | B single | C compact pack |
|---|---|---|---|
| 入口数 | 19 | **1** | **7** |
| 发现负载（常驻目录） | 3,897 | **340（−91%）** | 1,575（−60%） |
| 单任务负载 median | 2,207 | 4,730（**+114%**） | 晋级命中 2,247（持平）；路由命中 4,957 |
| 语料总量 | 113,417（含审计文件） | 52,599 | 68,799 |

**解读**：数据坐实了 v1.3 的预判——single 省的是**入口数与常驻发现负载**（产品/认知成本），但**每任务 token 负载翻倍**（入口 SKILL.md ~2.6k + 能力卡 ~2.1k）。若按「常驻每会话都付、任务负载只在命中时付」的模型，B 只有在**每会话命中任务 ≤1 次**时才有 token 净收益；token 角度 C 是更好的折中。**因此 single 的取舍理由只能是认知/管理成本，不能宣传为省 token**——这必须写进后续对外文案。

## 4. 路由评测（探索性，20 任务）

| 版本 | 命中预期 | acceptable/部分 | 无承接 | 正确判停 |
|---|---|---|---|---|
| A 原版 | 14 | 3 | **1** | 2 |
| B single | 18 | 0 | 0 | 2 |
| C compact | 18 | 0 | 0 | 2 |

关键发现：

1. **A 的结构性盲区**：书级查询（作者/章节/术语）没有任何已安装入口承接——INDEX/GLOSSARY/OVERVIEW 是构建产物不随 Skill 安装。B/C 的 overview/glossary 路由直接补上这块。
2. **B 的最大实证风险是真实触发率**：19 条 description 合计 ~3.9k token 的触发面被压缩到 340 token，不含「纳瓦尔」关键词的裸意图（"我总控制不住发火"）在真实宿主里是否激活 single 入口，静态判定测不出来。这正是读者测试的核心问题。
3. 近邻意图（T16—T18）在 B/C 的显式路由表下判定精确度不低于 A 的 description 竞争。
4. 本轮为主流程自评 + 静态判定，含乐观偏差，不能用于「不劣于」结论（详见 phase0-routing-eval.md 的偏差声明）。

## 5. §10.6 终止条件判定

| 终止信号 | 是否触发 | 依据 |
|---|---|---|
| single 路由命中明显差于 A | **否** | 静态判定 B ≥ A（18+2 vs 14+3+2），且补上书级查询盲区 |
| 能力覆盖丢失严重 | **否** | 19 能力全量编译为能力卡，0 丢失；晋级 6 能力双视图可达 |
| token 负担不可接受 | **部分预警** | 每任务负载 +114%；不构成终止，但否决「single 省 token」叙事 |
| 编译流程不可行 | **否** | 纯确定性编译 + 校验全绿，含死链清洗与发布哈希 |

**结论：不触发终止，进入读者交叉试用。** 但基于数据做两点方向修正建议：
1. `auto` 默认策略倾向 **compact pack**（发现负载 −60%、高频任务负载持平、书级查询有承接），single 作为「重视认知成本/单书轻使用」场景的显式选项——待读者测试数据确认后写入 ADR-002 的最终裁决。
2. 对外文案禁止「single 更省 token」表述，统一为「更少入口、更低管理与认知成本」。

## 6. 待用户执行：读者交叉试用协议（步骤 6）

1. 选定**一个宿主并锁版本**（如 Cursor 当前版本），记录宿主名+版本号。
2. 三个版本分别安装到干净的 skills 目录（一次只装一个版本）：
   - A: `books/naval-almanack-skill/` 下 19 个 Skill 目录
   - B: `benchmarks/naval/prototypes/single/naval-almanack/`
   - C: `benchmarks/naval/prototypes/compact-pack/` 下 7 个目录
3. 从 `task-set-v1.json` 抽 5—10 条（务必含 T05/T16/T17 这类**不含书名关键词**的裸意图），观察：入口是否触发、是否加载了正确能力卡、超范围题是否判停。
4. 每条记录：宿主、任务 id、触发的入口、加载的文件、主观质量 1—5 分。
5. 结果回填 `benchmarks/naval/phase0-report.md` 附录，据此做 go/no-go 与 auto 默认策略裁决。

## 7. 遗留与不做清单

- 能力卡未携带 per-capability `test-prompts.json`（Phase 1 的 Capability Bundle 契约再定）。
- 未建 `.cangjie/` 侧车、缓存、依赖图、SourceDocument、CLI——按方案 Phase 0 明确不做。
- 全部改动**未推送**：基线 tag/分支在本地，工作区新增文件未提交（见报告末尾文件清单）。
