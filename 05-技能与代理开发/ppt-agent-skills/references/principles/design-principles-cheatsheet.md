# 设计原则速查表 -- Step 4 字段级操作手册

> 用途不是讲理论，而是告诉页面策划师：当 JSON 某个字段写得不对，应该改哪一项。

> CARP（Contrast / Alignment / Repetition / Proximity）在这个 workflow 里是**版式 guardrail**，不是新风格系统，也不是新的模板库。
> 它不能覆盖 `scene_mode`、`density_contract`、`style.json.design_soul` 或 `variation_strategy`，只能约束这些合同如何被更清楚地执行。

---

## CARP 接入原则

### 冲突优先级

当 CARP 与既有预设看起来冲突时，按这个顺序判断：

1. `requirements / outline / planning` 真源优先
2. `scene_mode` 与 `density_contract` 优先
3. CARP 只管版式秩序，不管风格灵魂
4. `variation_strategy` 保证“同宗不同脸”，防止 repetition 退化成模板复制

### 四句短定义

- **Contrast**：决定谁先被看到，谁必须退后
- **Alignment**：决定元素归属于哪套共同骨架，而不是随意漂浮
- **Repetition**：决定同语义角色是否说同一种视觉语言，而不是页页重造系统
- **Proximity**：决定相关信息是否自然成组，而不是靠文字解释关系

### Scene Mode 力度表

| scene_mode | Contrast | Alignment | Repetition | Proximity |
|-----------|----------|-----------|------------|-----------|
| `launch` | 强，可做 hero-stage 压差 | 中，允许更自由重力 | 中，保语法不保长相 | 中，避免过度把页面收死 |
| `business` | 中强，服务判断先后 | 中高 | 中高 | 高，判断/依据/行动必须成组 |
| `report` | 中，服务指标与管理判读 | 高 | 高 | 高，比较轴和说明必须聚合 |
| `academic` | 中，服务论证主次，不做宣传冲击 | 很高 | 很高 | 很高，定义/证据/边界必须成组 |
| `technical` | 中，服务结构/机制/约束扫描 | 很高 | 高 | 很高，模块/步骤/限制条件必须成组 |
| `training` | 中，服务步骤和提醒先后 | 高 | 高 | 很高，步骤/警示/检查点必须贴近 |

### 字段映射速览

| CARP | 优先落在哪些字段 |
|------|------------------|
| Contrast | `visual_weight` / `design_intent.contrast_strategy` / `cards[].role` / `cards[].card_style` |
| Alignment | `layout_hint` / `focus_zone` / `director_command.spatial_strategy` / `layout_variation_note` |
| Repetition | `variation_guardrails.same_gene_as_deck` / `cards[].card_style` / `director_command.techniques` |
| Proximity | `page_text_strategy` / `cards[].content_focus` / `content_budget` / `compression_priority` |

---

## 原则 1. 视觉层级

核心问题：这一页谁是主角，谁必须退后？

优先影响字段：
- `visual_weight`
- `layout_hint`
- `cards[].role`
- `cards[].card_style`
- `director_command.anchor_treatment`

修正手法：
- 如果整页没有焦点：提高 anchor 卡片 `visual_weight`，改成 `accent` 或 `elevated`
- 如果所有卡片一样重：重写 `cards[].role`，只保留 1 个 `anchor`
- 如果标题和内容都在平均发力：在 `must_avoid` 明写“禁止等高等宽平均分配”
- 如果 `contrast_strategy` 只写成“突出重点”：改成具体对比轴，例如“靠标题尺度断层”“靠主图 vs 支撑卡面积差”“靠深浅对比而非装饰噪声”

危险信号：
- 3 张以上卡片都是 `filled`
- 没有 `anchor`
- `director_command.anchor_treatment` 只有“突出显示”这类空话
- dense scene 仍然把所有重点压成相近字号和相近字重

---

## 原则 2. 认知负荷

核心问题：观众这一页需要消化多少东西？

优先影响字段：
- `visual_weight`
- `density_label`
- `cards[].body`
- `cards[].chart`
- `rhythm_action`

修正手法：
- 如果一页塞太多：拆成 2 张 card 或下沉到下一页
- 如果必须保留高信息量：把 `rhythm_action` 标成“爆发”，下一页安排“缓冲”
- 如果信息少却做得很满：改成 `single-focus` 或 `free-section`
- 如果信息并不多但看起来很乱：先检查 `Proximity` 是否失败，而不是先删内容

危险信号：
- content 页 5 张以上 card 且都想当主角
- 一页同时放 3 种图表
- `visual_weight` 低，但 `cards` 内容非常密

---

## 原则 3. 构图与留白

核心问题：空间是在说话，还是只是装内容？

优先影响字段：
- `layout_hint`
- `layout_variation_note`
- `director_command.spatial_strategy`
- `decoration_hints.background`

修正手法：
- 如果布局描述落回“左边一块右边两块”：改写为重力关系，而不是像素切块
- 如果连续两页结构像克隆：在 `variation_guardrails.different_from_previous` 至少写 2 个反差维度
- 如果是金句或章节封面仍然很拥挤：把 `visual_weight` 下调，并用 `free-section`
- 如果元素像“差不多对齐”：把 `director_command.spatial_strategy` 改写成明确基线、列、带状区或中心轴，而不是模糊方位词
- `academic / technical / report` 场景下，默认要求更强 alignment；不要拿 launch 的自由重力去做 dense board

危险信号：
- “三栏均分”“上下两块”这类网页式描述
- 没有 `layout_variation_note`
- `background.feel` 为空或只有“简洁”
- 相关卡片距离和无关卡片距离差不多，看不出分组骨架

---

## 原则 4. 色彩与装饰克制

核心问题：这一页的装饰是在服务信息，还是在抢戏？

优先影响字段：
- `decoration_hints.*`
- `cards[].card_style`
- `variation_guardrails.same_gene_as_deck`

修正手法：
- 如果装饰很多却无主次：保留 1 个页面级手法 + 1 个卡片级手法
- 如果页与页完全不像同一套 deck：补 `same_gene_as_deck`
- 如果太稳太像模板：加强 `background.feel` 或 `page_accent.feel`，但写明 `restraint`
- `Repetition` 只要求“同类角色讲同一种语法”，不要求每页长一样；要重复的是标题角色、标签角色、注释角色、边框语法，不是构图结果

危险信号：
- 三层 `decoration_hints` 都写成“轻微点缀”
- 每页 weapon 组合完全一样
- `accent` 卡片超过 1 张

---

## 原则 5. 数据表达诚实

核心问题：数据是证据，不是贴纸。

优先影响字段：
- `cards[].data_points`
- `cards[].chart`
- `page_goal`
- `audience_takeaway`

修正手法：
- 没有具体数据时，不要假装做数据页，改成 `framework` 或 `quote`
- 有核心 KPI 时，至少安排 1 张 `data_highlight` 或带图表的 `data`
- 图表存在只是装饰时，删掉 `chart`

危险信号：
- 图表只是“为了看起来专业”
- `page_goal` 是判断句，但没有任何证据卡承接
- `data_points.source` 大量空白

---

## 原则 6. 节奏与变奏

核心问题：翻页时有呼吸和推进吗？

优先影响字段：
- `visual_weight`
- `rhythm_action`
- `director_command.techniques`
- `variation_guardrails.different_from_previous`

修正手法：
- 连续两页已经高压，第三页必须降压或视觉降密
- 同一布局复用时，必须换掉至少 2 个维度：重心 / card_style / 技法 / 留白
- 相邻页技术组合雷同，直接重写 `director_command.techniques`

危险信号：
- 3 页连续 `visual_weight >= 7`
- 3 页连续出现同一 `layout_hint`
- 相邻页 techniques 完全一致

---

## CARP 反模式速查

- **Contrast 失效**：所有字号、字重、卡片面积都差不多，观众不知道先看哪里
- **Alignment 失效**：元素像“差不多对齐”，但没有共同列线、边界线或中心轴
- **Repetition 失效**：同样是标题 / 标签 / 注释 / 指标块，却每页都换语法
- **Proximity 失效**：标题、说明、指标、注释互相散开，只能靠文字解释关系
- **误用 CARP**：把 `academic / technical / report` 做成 launch hero；或者把 `launch` 做成过于死板的表格页

---

## 字段纠偏表

| 发现的问题 | 先改哪个字段 | 再看哪个字段 |
|-----------|------------|------------|
| 页面对但不惊艳 | `director_command` | `decoration_hints` |
| 页面像网页 | `layout_hint` | `layout_variation_note` |
| 卡片都一样 | `cards[].card_style` | `cards[].role` |
| 节奏过平 | `visual_weight` | `rhythm_action` |
| 装饰乱 | `decoration_hints.*.restraint` | `variation_guardrails` |
| 论点强但证据弱 | `cards[].data_points` | `chart` |

---

## 逐页 8 项体检单

- 这页的 `page_goal` 是否是一句可判断的完整论点？
- `cards[]` 是否存在清晰的主次，而不是平均铺开？
- `layout_hint` 是否真的匹配内容结构，而不是习惯性套版？
- `director_command` 是否给出明确镜头感？
- 三层 `decoration_hints` 是否各司其职？
- 与上一页是否至少有 2 个维度不同？
- `must_avoid` 是否说中了这页最危险的模板化风险？
- 设计层拿到这页 JSON 后，是否能知道什么不能改、什么可以自由发挥？
