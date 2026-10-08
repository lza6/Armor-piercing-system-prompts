# get_shop_data — 店铺维度数据查询

## 功能说明

获取店铺维度的汇总数据，包含店铺支付金额、支付买家数、在线商品数等，作为商品评分时的对比基准。

## 前置条件

- 已配置 AK（通过设置环境变量 `ALI_1688_AK`）

## CLI 调用

```bash
python {baseDir}/cli.py get_shop_data
```

**参数说明**：无需参数，脚本自动使用最新时间分区。

## 返回数据说明

成功时返回 `data` 对象，包含以下店铺整体指标：

| 字段 | 类型 | 说明 |
|------|------|------|
| pay_ord_amt_1d_001 | Number | 店铺支付金额（分） |
| pay_ord_byr_cnt_1d_001 | Number | 店铺支付买家数 |
| uv_1d_001 | Number | 店铺访客数 |
| pv_1d_001 | Number | 店铺浏览量 |
| itm_cnt_1d | Number | 在线商品数 |
| pwp_itm_cnt_1d | Number | 潜力品商品数 |
| hqp_itm_cnt_1d | Number | 优品商品数 |

> 金额单位为"分"，展示时需除以 100 转换为元。

## 输出格式

### 成功输出

```json
{
  "success": true,
  "markdown": "店铺数据查询成功",
  "data": {
    "data": {
      "pay_ord_amt_1d_001": 1560000,
      "pay_ord_byr_cnt_1d_001": 230,
      "uv_1d_001": 5600,
      "pv_1d_001": 18200,
      "itm_cnt_1d": 320,
      "pwp_itm_cnt_1d": 45,
      "hqp_itm_cnt_1d": 12
    }
  }
}
```

### 失败输出

```json
{
  "success": false,
  "markdown": "❌ AK 未配置，无法查询店铺数据。\n\n请补充有效 AK 或检查鉴权配置后重试",
  "data": {
    "data": {}
  }
}
```

## 数据用途

此命令返回的数据需要作为 `score_and_select` 命令的 `--shop_total` 参数传入，用于计算每个商品的：

- **销售贡献度**：商品支付金额 / `pay_ord_amt_1d_001`，商品买家数 / `pay_ord_byr_cnt_1d_001`
- 作为商品在店铺整体中的占比基准

## 异常处理

| 异常场景 | 处理方式 |
|---------|---------|
| AK 未配置 | 提示用户配置 AK |
| 签名无效（401） | 提示用户检查 AK 是否有效 |
| 请求被限流（429） | 建议用户等待 1-2 分钟后重试 |

## 使用注意

1. 此技能为只读操作，不会修改任何数据
2. 脚本自动使用最新时间分区（max_pt），无需指定日期
3. 可与 `get_item_overview` 并行执行以提高效率
