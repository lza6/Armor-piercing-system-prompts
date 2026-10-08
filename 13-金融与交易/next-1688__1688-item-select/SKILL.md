---
name: 1688-item-selection
description: |
  1688 商家重点品圈选 —— 基于多维度商品评分智能识别值得重点运营的商品或搜索商品。
  工具能力：五维度评分（销售贡献、流量效率、成长潜力、营销ROI、商品健康度），商品分层（S/A/B/C级），搜索商品。
  触发词：重点品查看、圈选重点品、圈选运营商品、今日运营重点、选品、推荐商品、商品分层、商品优先级、搜索商品。
metadata:
  openclaw:
    emoji: "🎯"
    requires:
      bins:
        - python
  interactions:
    - name: select_products_from_scoring
      type: table
      selectionType: product
      description: "从评分圈选结果中选择要重点运营的商品"
      required_data:
        title: "表格标题，固定为'重点品圈选结果'"
        columns: "列定义数组，包含商品ID、标题、等级、分层、综合得分、支付金额、买家数、访客数"
        rows: "数据行数组，每行包含 id, title, level, levelName, totalScore, payAmount, buyerCount, uv"
        totalCount: "数据总条数"
    - name: select_products_from_search
      type: table
      selectionType: product
      description: "从关键词搜索结果中选择商品"
      required_data:
        title: "表格标题，如'搜索结果: {keyword}'"
        columns: "列定义数组，包含图片、商品ID、标题、最低价、最高价、状态"
        rows: "数据行数组，每行包含 id, title, imageUrl, minPrice, maxPrice, status"
        totalCount: "数据总条数"
---

# 1688-item-selection — 商家重点品圈选

## 技能概述

智能识别商家应该重点运营的商品，基于五大维度进行综合评分和分层，输出商品清单；也可以通过关键词搜索商品。

**输出原则**：

- 仅输出商品清单（商品名称、ID、标签、选择原因）
- 不展示综合得分、不输出优化动作、不输出行动建议
- 商品优化动作由独立的"商品诊断skill"负责
- 默认推荐 3 款商品，用户可指定推荐数量（1-10 款）
- 当没有圈出重点商品时可以建议商家输入关键词搜索商品

## 评分维度

1. **销售贡献度**（30%）- 商品对店铺GMV和买家数的贡献
2. **流量效率**（25%）- 流量转化率、曝光转化、加购率
3. **成长潜力**（20%）- 平台标签（优品/潜力品）、成长分层
4. **营销ROI**（15%）- 广告投入产出比
5. **商品健康度**（10%）- 服务能力、库存、退款率

## 分层标准

- **S级（≥80分）**：重点推广品 - 加大投入抢占流量
- **A级（60-80分）**：潜力培育品 - 针对性优化短板
- **B级（40-60分）**：维持运营品 - 保持现状作为辅助
- **C级（<40分）**：优化调整品 - 诊断问题考虑下架

## CLI 命令

### configure — 配置 AK
```bash
# 查看 AK 状态
python {baseDir}/cli.py configure
# 设置 AK
python {baseDir}/cli.py configure YOUR_AK
```
配置网关鉴权所需的 AK。所有操作命令都依赖 AK，首次使用前需先配置。

### get_item_overview — 获取商品概览统计
```bash
python {baseDir}/cli.py get_item_overview
```
返回：商品总数、有销售商品数、总销售额、总买家数、总UV，以及按销售额分段的商品数量。

### get_shop_data — 获取店铺维度数据
```bash
python {baseDir}/cli.py get_shop_data
```
返回：店铺支付金额、支付买家数、在线商品数。用作商品评分的对比基准。

### score_and_select — 商品评分与圈选
```bash
python {baseDir}/cli.py score_and_select \
  --shop_total '{"pay_ord_amt_1d_001": 100000, "pay_ord_byr_cnt_1d_001": 50}' \
  --strategy comprehensive \
  --limit 100 \
  --top_n 3
```

参数说明：

| 参数 | 必填 | 说明 |
|------|------|------|
| `--shop_total` | 是 | 店铺维度数据 JSON（由 get_shop_data 获取） |
| `--strategy` | 否 | 查询策略: comprehensive(默认) / sales / all |
| `--limit` | 否 | 获取商品数量上限，默认100 |
| `--top_n` | 否 | 输出排名前N的商品，默认10 |

**⚠️ 交互渲染（必须执行）**：当此命令返回的 `data.data.products` 包含 **≥2 个商品**时，**禁止**直接用 Markdown 表格输出商品数据，必须通过交互组件渲染：
1. **先读取** `{baseDir}/references/interaction-specs.md` 中的 `select_products_from_scoring` 章节，获取交互组件的完整数据结构定义
2. **再触发** `metadata.interactions` 中声明的 `select_products_from_scoring` 交互，严格按 specs 中的字段映射构造参数

### search_offer_by_keyword — 通过关键词搜索店铺商品
```bash
python {baseDir}/cli.py search_offer_by_keyword --keyword "测试" --page 1 --page_size 10
```

参数说明：

| 参数 | 必填 | 说明 |
|------|------|------|
| `--keyword` | 否 | 搜索关键词（可选）。如果传入关键词后搜索无结果，**让大模型尝试换几个简短的相似词来搜索，最多尝试 3 次** |
| `--page` | 否 | 页码，默认 1 |
| `--page_size` | 否 | 每页数量，默认 10 |

**⚠️ 交互渲染（必须执行）**：当此命令返回的 `data.data.items` 包含 **≥2 个商品**时，**禁止**直接用 Markdown 表格输出商品数据，必须通过交互组件渲染：
1. **先读取** `{baseDir}/references/interaction-specs.md` 中的 `select_products_from_search` 章节，获取交互组件的完整数据结构定义
2. **再触发** `metadata.interactions` 中声明的 `select_products_from_search` 交互，严格按 specs 中的字段映射构造参数

## 生成流程

### 第1步：收集信息

从用户获取以下信息：

1. **推荐数量**（可选）：需要推荐的商品数量，默认为 3 款
2. **筛选条件**（可选）：类目范围、商品数量限制等

### 第2步：查询概览数据（可并行执行）

```bash
python {baseDir}/cli.py get_item_overview
python {baseDir}/cli.py get_shop_data
```

根据概览结果中的商品总数决定下一步查询策略：

- **≤200个商品**：使用 `--strategy all` 直接查询全部
- **201-500个商品**：使用 `--strategy comprehensive` 默认筛选
- **>500个商品**：使用 `--strategy comprehensive --limit 200` 限制数量

### 第3步：评分与圈选

将第2步获取的店铺数据通过 `--shop_total` 传入评分命令：

```bash
python {baseDir}/cli.py score_and_select \
  --shop_total '{第2步返回的店铺数据 JSON}' \
  --strategy <策略> \
  --limit <N> \
  --top_n <推荐数量>
```

### 第4步：生成报告与交互选择

基于评分结果，按综合得分从高到低排序，选取前 N 款商品（默认 3 款），输出结构化的 Markdown 报告。

**交互触发**：当圈选结果包含多个商品（≥2 款）时，在输出 Markdown 报告的同时，调用 `show_interaction` 并设置 `name='select_products_from_scoring'`，将 `score_and_select` 返回的 `products` 数组映射到交互数据槽位，让用户通过表格勾选要重点运营的商品。具体的字段映射规则与组件渲染数据结构请查阅 [`references/interaction-specs.md`](./references/interaction-specs.md) 中对应交互的章节。

## 报告模板

```markdown
# 重点品圈选结果

基于{策略总结}，推荐以下 {N} 款商品：

1. **{商品标题}** (ID: {item_id}) —— 【{商品标签}】
    - {选择原因，1-2句话说明核心数据和优势}

2. **{商品标题}** (ID: {item_id}) —— 【{商品标签}】
    - {选择原因}

3. ...
```

**格式要求**：

- **商品标签**：根据商品特征标注角色，如引流款、利润款、潜力款、爆款、动销款等
- **选择原因**：1-2句话说明核心数据和优势
- **禁止内容**：不展示综合得分、不输出优化动作、不输出行动建议

## 交互能力

本 Skill 支持通过 Newton Agent 的客户端交互组件，在输出多个商品时提供可视化的表格选择体验。

### 触发规则

| 交互名称 | 触发时机 | 数据来源 |
|----------|---------|---------|
| `select_products_from_scoring` | `score_and_select` 返回 ≥2 个商品时 | `score_and_select` 返回的 `products` 数组 |
| `select_products_from_search` | `search_offer_by_keyword` 返回 ≥2 个商品时 | `search_offer_by_keyword` 返回的 `items` 数组 |

### 数据填充

- **`select_products_from_scoring`**：设置 `title` 为表格标题，`columns` 为列定义数组，将 `score_and_select` 返回的 `data.data.products` 数组逐项转换后赋值给 `rows` 槽位。每行需包含 `id`、`title`、`level`、`levelName`、`totalScore`、`payAmount`、`buyerCount`、`uv`。
- **`select_products_from_search`**：设置 `title` 为表格标题，`columns` 为列定义数组，将 `search_offer_by_keyword` 返回的 `data.data.items` 数组逐项转换后赋值给 `rows` 槽位。每行需包含 `id`（源字段 `itemId`）、`title`、`imageUrl`（源字段 `mainImage`）、`minPrice`、`maxPrice`、`status`。

**具体的字段映射规则与组件渲染数据结构请查阅 [`references/interaction-specs.md`](./references/interaction-specs.md) 中对应交互的章节。**

### 注意事项

- 调用 `table` 类型交互前，务必确保 `rows` 有真实数据，否则前端会报错
- 当结果仅有 1 个商品时，无需触发交互，直接输出 Markdown 报告即可
- 交互组件与 Markdown 报告可同时输出，互不冲突

## 安全声明

| 风险级别 | 命令 | Agent 行为 |
|---------|------|----------|
| 只读 | configure | 可直接执行，无需确认 |
| 只读 | get_item_overview | 可直接执行，无需确认 |
| 只读 | get_shop_data | 可直接执行，无需确认 |
| 只读 | score_and_select | 可直接执行，无需确认 |
| 只读 | search_offer_by_keyword | 可直接执行，无需确认 |

## 异常处理

任何命令输出 `success: false` 时：

1. **先输出 `markdown` 字段**（已包含用户可读的错误描述）
2. **再根据关键词追加引导**：

| markdown 关键词 | Agent 额外动作 |
|----------------|--------------|
| "AK 未配置" 或 "签名无效" 或 "401" | 提示用户补充有效 AK 或检查鉴权配置后重试 |
| "限流" 或 "429" | 建议用户等待 1-2 分钟后重试 |
| 其他 | 仅输出 markdown 即可 |

## 环境变量（.env）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `SKILL_NAME` | `1688-item-selection` | skill 名称 |
| `SKILL_VERSION` | `1.0.0` | skill 版本号 |
| `SKILL_CHANNEL` | `clawhub` | 发布渠道 |

## 埋点上报

每次 CLI 命令执行时，自动向 skill 网关上报一次调用记录，用于统计 skill 调用次数。

## 输出格式

采用标准 JSON 输出：

```json
{
  "success": true,
  "markdown": "商品评分与圈选成功",
  "data": {
    "data": {
      "total_scored": 50,
      "returned_count": 3,
      "products": [...],
      "summary": {"S级": 2, "A级": 5, "B级": 20, "C级": 23}
    }
  }
}
```

## 评分规则

详细的评分规则和权重说明见 [scoring_rules.md](references/scoring_rules.md)。

## 数据表Schema

数据表字段定义见 [table_schema.md](references/table_schema.md)。

## 使用原则

1. **必须查询真实数据**：通过 CLI 命令获取真实数据，不要编造
2. 概览和店铺数据脚本可以并行执行以提高效率
3. 根据概览结果的商品数量决定明细查询策略
4. 此技能为只读操作，不会修改任何数据

## 免责声明：
1、您理解并同意，技能运行结果和输出内容可能因适用的AI agent、大模型不同而产生差异或幻觉，请您对重要的信息进行甄别核实。
2、您应妥善保管您的Access Key（AK），这是您运行1688技能的身份凭证，请勿提供给第三人，避免身份凭证泄露造成损失。
3、您下载安装1688技能运行时应始终保持其完整性，不得擅自篡改技能的代码、相关文件或其他内容，否则1688不对技能运行结果和输出内容承担任何法律责任。
4、受限于当前技术发展，我们无法保证技能所有运行结果、输出内容的准确性、真实性、时效性，亦不代表我们的态度、观点和推荐，请您谨慎核实技能运行结果和输出内容，除法律规定由我们承担赔偿责任的场景外，我们不承担其他赔偿责任。