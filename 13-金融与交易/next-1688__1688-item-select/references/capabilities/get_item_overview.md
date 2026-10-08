# get_item_overview — 商品概览统计查询

## 功能说明

获取商品总体概况统计，了解数据规模以决定后续的评分与圈选查询策略。返回商品总数、有销售商品数、总销售额、总买家数、总UV，以及按销售额分段的商品数量（高/中/低）。

## 前置条件

- 已配置 AK（通过设置环境变量 `ALI_1688_AK`）

## CLI 调用

```bash
python {baseDir}/cli.py get_item_overview
```

**参数说明**：无需参数，脚本自动使用最新时间分区。

## 返回数据说明

成功时返回 `data` 对象，包含以下指标：

| 字段 | 类型 | 说明 |
|------|------|------|
| total_item_cnt | Number | 在线商品总数 |
| has_sale_item_cnt | Number | 有销售商品数（近1天） |
| total_pay_amt | Number | 总销售额（分） |
| total_buyer_cnt | Number | 总支付买家数 |
| total_uv | Number | 总访客数 |
| high_sale_cnt | Number | 高销售额商品数 |
| mid_sale_cnt | Number | 中销售额商品数 |
| low_sale_cnt | Number | 低销售额商品数 |

> 金额单位为"分"，展示时需除以 100 转换为元。

## 输出格式

### 成功输出

```json
{
  "success": true,
  "markdown": "商品概览查询成功",
  "data": {
    "data": {
      "total_item_cnt": 320,
      "has_sale_item_cnt": 85,
      "total_pay_amt": 1560000,
      "total_buyer_cnt": 230,
      "total_uv": 5600,
      "high_sale_cnt": 10,
      "mid_sale_cnt": 35,
      "low_sale_cnt": 40
    }
  }
}
```

### 失败输出

```json
{
  "success": false,
  "markdown": "❌ AK 未配置，无法查询商品概览。\n\n请补充有效 AK 或检查鉴权配置后重试",
  "data": {
    "data": {}
  }
}
```

## 后续策略决策

根据 `total_item_cnt` 决定 `score_and_select` 的查询策略：

| 商品总数 | 推荐策略 |
|---------|---------|
| ≤200 | `--strategy all` 直接查询全部 |
| 201-500 | `--strategy comprehensive` 默认筛选 |
| >500 | `--strategy comprehensive --limit 200` 限制数量 |

## 异常处理

| 异常场景 | 处理方式 |
|---------|---------|
| AK 未配置 | 提示用户配置 AK |
| 签名无效（401） | 提示用户检查 AK 是否有效 |
| 请求被限流（429） | 建议用户等待 1-2 分钟后重试 |

## 使用注意

1. 此技能为只读操作，不会修改任何数据
2. 脚本自动使用最新时间分区（max_pt），无需指定日期
3. 可与 `get_shop_data` 并行执行以提高效率
