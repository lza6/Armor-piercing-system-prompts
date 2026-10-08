# ResearchSynth Phase 2 Playbook -- 数据格式化、整理与自审

## 目标与审查态度（拒绝自以为是的走过场）

作为质量守门员，接管上一阶段的无序原始碎片，清洗并提炼为高密度的 `search-brief.txt`。
**严厉警告**：绝不允许看一遍资料后，就自视甚高地得出几句宏观套话（比如“市场很大”、“前景广阔”）然后通过！
你必须审视：摘要中是否有血有肉？是否有具体的痛点案例？是否有精确到个位数的数据支撑？
如果没有深度修改和提纯颗粒度，请立刻驳回自己重写！

---

## 核心任务流

### 1. 数据对齐与清洗

1. **去重合并**：如果同一事实（如市占率）在多处提及，合并为一条记录，并在 `[来源]` 中保留出处的交集。
2. **矛盾对抗**：如果 A 说增长了 15%，B 说 18%，保留两者，明确标注**数据冲突（Conflict）**及对应来源。
3. **可信度贴标**：按权威度（high: 官方白皮书/财报 / medium: 行业媒体 / low: SEO凑字文章）为关键数据贴置信度标签。

### 2. 构建结构化数据包（核心）

这页报告最重要的资产，是喂给 PPTX 版面的**组装弹药**。你必须根据搜索中的原始数据，严格按照下面这 11 种格式转化：

| 结构化类型 | 输出格式规范 | 对应 PPT 组件 |
|-----------|-------------|---------------|
| `metrics` | `{value} {unit} ({trend}) [来源: {source}] [可信度: high]` | `kpi` / `metric-row` |
| `data_tables` | 多行多列的 md table，附 `[来源]` | `table` 卡片 |
| `trend_series`| `{time_1}: {value} \n {time_2}: {value}...` 换行平铺，带 `[来源]` | `sparkline` 折线 |
| `ranked_list` | `1. {name}: {value} \n 2. {name}...` | `list` / `data_highlight` |
| `before_after`| `Before: {x} -> After: {y} (差值: {diff}) [来源]` | `comparison` 卡片 |
| `funnel_data` | `{label1}: {val} -> {label2}: {val} (流失: {rate})` | `funnel` 图 |
| `pie_data`    | `{seg_1}: {percentage}%, {seg_2}... [来源]` | `ring` / `treemap` |
| `timelines`   | `{year/date}: {milestone_desc} [来源]` | `timeline` 区块 |
| `expert_quotes`| `"{quote_text}" -- {person/title}, {org} [来源]` | `quote` 大字引言 |
| `team_profiles`| `{name} ({title}): {desc}` | `people` 人物卡 |
| `process_flows`| `Step 1: {x} -> Step 2: {y}...` | `process` 流程图 |

---

## search-brief.txt 内容骨架合同

输出的简报必须严格按照下方层级格式，不得残缺：

```text
# Research Brief
主题：{topic}
素材总数：{n}
可信度分布：high={h} / medium={m} / low={l}

---

## 核心发现
1. {一句话发现} [来源: {source}] [可信度: high]
2. ...

## 关键数据
- {数据点} [来源: {source}]
- ...

## 覆盖缺口
- {dimension/类型}: {什么是缺失的，如：未获取到具体的竞品财报}
- ...

## 分维度摘要
### 核心定义
{summary_text}
### 市场数据
...（六大维度按序摘要）

## PPTX 结构化数据包
### metrics
- 47.3 % (上升) [来源: ...]
### timelines
- 2024: 突破 10 万 DAU [来源: ...]
... （列出提取出来的所有类型数据包，至少 3 种不同的大块）

### 数据覆盖评估
- metrics: {count} 个可用 / 缺口: {what_is_missing}
- timelines: {count} 个可用 / 缺口: ...
```

---

## 质量门卫自审

在宣告完成前，问自己：
1. **数字是否有依据？** 绝不能写出"市占率大"，必须是"34% [来源]"。
2. **结构包是否超额达成？** 你的目标是提取出**至少 3 种完全跨类型**（比如有 metric，有 timeline，有 quote）的数据给 Planning Subagent 用作卡片设计。
3. **缺口是否坦诚暴露？** 没有拿到营收图表，要明确在`覆盖缺口`写出"缺失营收趋势"，禁止伪造。
