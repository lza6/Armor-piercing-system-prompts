# score_and_select — 商品评分与圈选

## 功能说明

对商品进行五维度综合评分和 S/A/B/C 分层，内部自动获取商品明细并完成评分，只返回 Top-N 精简结果，避免大量原始数据进入 LLM 上下文浪费 token。

**五大评分维度**：
1. **销售贡献度**（30%）- 商品对店铺 GMV 和买家数的贡献
2. **流量效率**（25%）- 流量转化率、曝光转化、加购率
3. **成长潜力**（20%）- 平台标签（优品/潜力品）、成长分层
4. **营销ROI**（15%）- 广告投入产出比
5. **商品健康度**（10%）- 服务能力、库存、退款率

**分层标准**：
- **S级**（≥80分）：重点推广品
- **A级**（60-80分）：潜力培育品
- **B级**（40-60分）：维持运营品
- **C级**（<40分）：优化调整品

## 前置条件

- 已配置 AK（通过设置环境变量 `ALI_1688_AK`）
- 已通过 `get_shop_data` 获取店铺维度数据

## CLI 调用

```bash
python {baseDir}/cli.py score_and_select \
  --shop_total '{"pay_ord_amt_1d_001": 100000, "pay_ord_byr_cnt_1d_001": 50}' \
  --strategy comprehensive \
  --limit 100 \
  --top_n 3
```

**参数说明**：

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `--shop_total` | 是 | — | 店铺维度数据 JSON 字符串（由 `get_shop_data` 获取） |
| `--strategy` | 否 | `comprehensive` | 查询策略: `comprehensive`(综合排序) / `sales`(按销售额) / `all`(全部商品) |
| `--limit` | 否 | `100` | 获取商品数量上限 |
| `--top_n` | 否 | `10` | 输出排名前N的商品 |

### 策略选择指南

| 场景 | 推荐策略 |
|------|---------|
| 商品数 ≤200 | `--strategy all` |
| 商品数 201-500 | `--strategy comprehensive`（默认） |
| 商品数 >500 | `--strategy comprehensive --limit 200` |
| 只看销售额排名 | `--strategy sales` |

## 返回数据说明

成功时返回 `data` 对象，包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| total_scored | Number | 参与评分的商品总数 |
| returned_count | Number | 实际返回的商品数 |
| products | Array | Top-N 商品列表（见下方结构） |
| summary | Object | 各分层商品计数（S/A/B/C级） |

### products 数组中每个商品结构

| 字段 | 说明 |
|------|------|
| rank | 排名 |
| item_id | 商品ID |
| title | 商品标题 |
| scores.sales_score | 销售贡献度得分 |
| scores.traffic_score | 流量效率得分 |
| scores.potential_score | 成长潜力得分 |
| scores.roi_score | 营销ROI得分 |
| scores.health_score | 商品健康度得分 |
| scores.total_score | 综合得分 |
| classification.level | 分层等级（S/A/B/C级） |
| classification.name | 分层名称 |
| classification.strategy | 运营建议 |
| key_metrics | 关键指标（支付金额/买家数/UV/广告成本） |

## 输出格式

### 成功输出

```json
{
  "success": true,
  "markdown": "商品评分与圈选成功",
  "data": {
    "data": {
      "total_scored": 85,
      "returned_count": 3,
      "products": [
        {
          "rank": 1,
          "item_id": "123456",
          "title": "商品A",
          "scores": {
            "sales_score": 90,
            "traffic_score": 85,
            "potential_score": 70,
            "roi_score": 100,
            "health_score": 45,
            "total_score": 84.0
          },
          "classification": {
            "level": "S级",
            "name": "重点推广品",
            "strategy": "加大投入，抢占流量，优化详情页，参与营销活动"
          },
          "key_metrics": {
            "pay_ord_amt_1d": 156000,
            "pay_ord_byr_cnt_1d": 23,
            "ipv_uv_1d": 460,
            "ad_cost_1d": 800
          }
        }
      ],
      "summary": {
        "S级": 2,
        "A级": 8,
        "B级": 35,
        "C级": 40
      }
    }
  }
}
```

### 失败输出

```json
{
  "success": false,
  "markdown": "❌ AK 未配置，无法执行商品评分。\n\n请补充有效 AK 或检查鉴权配置后重试",
  "data": {
    "data": {}
  }
}
```

## 典型调用流程

```bash
# 1. 先获取店铺数据
SHOP_DATA=$(python {baseDir}/cli.py get_shop_data | python -c "import sys,json; print(json.dumps(json.loads(sys.stdin.read())['data']['data']))")

# 2. 传入评分命令
python {baseDir}/cli.py score_and_select --shop_total "$SHOP_DATA" --top_n 3
```

## 异常处理

| 异常场景 | 处理方式 |
|---------|---------|
| AK 未配置 | 提示用户配置 AK |
| 签名无效（401） | 提示用户检查 AK 是否有效 |
| 请求被限流（429） | 建议用户等待 1-2 分钟后重试 |
| shop_total JSON 解析失败 | 提示用户检查 JSON 格式 |
| 未获取到商品数据 | 提示用户检查店铺是否有在线商品 |

## 使用注意

1. 此技能为只读操作，不会修改任何数据
2. `--shop_total` 的值来自 `get_shop_data` 的返回结果，不要编造
3. 评分详细规则见 [scoring_rules.md](../scoring_rules.md)
4. 数据表字段定义见 [table_schema.md](../table_schema.md)
