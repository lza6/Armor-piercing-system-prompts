# 交互组件详细规范

本文档定义了 1688-item-selection Skill 中所有交互组件的具体数据结构与映射规则。大模型在调用 `show_interaction` 前需查阅本文档，确保数据结构正确。

---

## 1. select_products_from_scoring (Table 组件)

### 组件类型

`type: table` — 用于展示评分圈选结果，让用户从中勾选要重点运营的商品。

### 数据槽位定义

- **`title`**:
  - 类型: `String`
  - 说明: 表格标题
  - 固定值: `"重点品圈选结果"`

- **`columns`**:
  - 类型: `Array<Object>`
  - 说明: 表格列定义
  - 每列包含 `key`（字段名）、`label`（列标题）、`width`（可选，列宽 px）

- **`rows`**:
  - 类型: `Array<Object>`
  - 映射规则: 从 `score_and_select` 返回的 `data.data.products` 数组逐项转换
  - 必须字段: `id`, `title`, `level`, `levelName`, `totalScore`, `payAmount`, `buyerCount`, `uv`

### 映射规则

从 `score_and_select` 返回的每个 product 对象：

| 源字段路径 | 目标字段 | 说明 |
|-----------|---------|------|
| `item_id` | `id` | 商品 ID |
| `title` | `title` | 商品标题 |
| `classification.level` | `level` | 分层等级，如 S级、A级 |
| `classification.name` | `levelName` | 分层名称，如 重点推广品 |
| `scores.total_score` | `totalScore` | 综合得分 |
| `key_metrics.pay_ord_amt_1d` | `payAmount` | 支付金额 |
| `key_metrics.pay_ord_byr_cnt_1d` | `buyerCount` | 支付买家数 |
| `key_metrics.ipv_uv_1d` | `uv` | 访客数 |

### columns 定义

```json
[
  { "key": "id", "label": "商品ID", "width": 100 },
  { "key": "title", "label": "商品标题" },
  { "key": "level", "label": "等级", "width": 70 },
  { "key": "levelName", "label": "分层", "width": 100 },
  { "key": "totalScore", "label": "综合得分", "width": 90 },
  { "key": "payAmount", "label": "支付金额", "width": 100 },
  { "key": "buyerCount", "label": "买家数", "width": 80 },
  { "key": "uv", "label": "访客数", "width": 80 }
]
```

### 完整数据示例

```json
{
  "title": "重点品圈选结果",
  "columns": [
    { "key": "id", "label": "商品ID", "width": 100 },
    { "key": "title", "label": "商品标题" },
    { "key": "level", "label": "等级", "width": 70 },
    { "key": "levelName", "label": "分层", "width": 100 },
    { "key": "totalScore", "label": "综合得分", "width": 90 },
    { "key": "payAmount", "label": "支付金额", "width": 100 },
    { "key": "buyerCount", "label": "买家数", "width": 80 },
    { "key": "uv", "label": "访客数", "width": 80 }
  ],
  "rows": [
    {
      "id": "728930501038",
      "title": "2024新款纯棉T恤男士短袖",
      "level": "S级",
      "levelName": "重点推广品",
      "totalScore": 84.0,
      "payAmount": 156000,
      "buyerCount": 23,
      "uv": 460
    },
    {
      "id": "728930501039",
      "title": "夏季薄款休闲裤男宽松直筒",
      "level": "A级",
      "levelName": "潜力培育品",
      "totalScore": 72.5,
      "payAmount": 89000,
      "buyerCount": 15,
      "uv": 320
    }
  ],
  "totalCount": 2
}
```

---

## 2. select_products_from_search (Table 组件)

### 组件类型

`type: table` — 用于展示关键词搜索结果，让用户从中勾选目标商品。

### 数据槽位定义

- **`title`**:
  - 类型: `String`
  - 说明: 表格标题
  - 示例值: `"搜索结果: {keyword}"`，其中 `{keyword}` 为用户输入的搜索关键词

- **`columns`**:
  - 类型: `Array<Object>`
  - 说明: 表格列定义

- **`rows`**:
  - 类型: `Array<Object>`
  - 映射规则: 从 `search_offer_by_keyword` 返回的 `data.data.items` 数组逐项转换
  - 必须字段: `id`, `title`, `imageUrl`, `minPrice`, `maxPrice`, `status`

### 映射规则

从 `search_offer_by_keyword` 返回的每个 item 对象：

| 源字段路径 | 目标字段 | 说明 |
|-----------|---------|------|
| `itemId` | `id` | 商品 ID |
| `title` | `title` | 商品标题 |
| `mainImage` | `imageUrl` | 商品主图 URL |
| `minPrice` | `minPrice` | 最低价（元） |
| `maxPrice` | `maxPrice` | 最高价（元） |
| `status` | `status` | 商品状态，需转换为中文：`PUBLISHED` → `上架中`，其他值 → `未上架` |

### columns 定义

```json
[
  { "key": "imageUrl", "label": "图片", "width": 80 },
  { "key": "id", "label": "商品ID", "width": 140 },
  { "key": "title", "label": "商品标题" },
  { "key": "minPrice", "label": "最低价(元)", "width": 100 },
  { "key": "maxPrice", "label": "最高价(元)", "width": 100 },
  { "key": "status", "label": "状态", "width": 90 }
]
```

### 完整数据示例

```json
{
  "title": "搜索结果: 女装连衣裙",
  "columns": [
    { "key": "imageUrl", "label": "图片", "width": 80 },
    { "key": "id", "label": "商品ID", "width": 140 },
    { "key": "title", "label": "商品标题" },
    { "key": "minPrice", "label": "最低价(元)", "width": 100 },
    { "key": "maxPrice", "label": "最高价(元)", "width": 100 },
    { "key": "status", "label": "状态", "width": 90 }
  ],
  "rows": [
    {
      "id": "1001683074980",
      "title": "xs 测试女装连衣裙",
      "imageUrl": "https://cbu01.alicdn.com/img/ibank/O1CN01y16eab1igwhk43Zbb_!!2215135204443-0-cib.jpg",
      "minPrice": 0.1,
      "maxPrice": 0.1,
      "status": "上架中"
    },
    {
      "id": "1001683074981",
      "title": "2024新款气质通勤连衣裙",
      "imageUrl": "https://cbu01.alicdn.com/img/ibank/xxx.jpg",
      "minPrice": 59.9,
      "maxPrice": 89.9,
      "status": "上架中"
    }
  ],
  "totalCount": 2
}
```
