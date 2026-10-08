# 数据表Schema说明

## ads_wxb_aide_itm_idx_1d（商品指标表）

### 基础信息
- `item_id`: 商品ID
- `user_id`: 用户ID
- `member_id`: 会员ID
- `title`: 商品标题
- `itm_status`: 商品状态
- `itm_stock`: 商品库存

### 类目信息
- `cate_id`: 叶子类目ID
- `cate_name`: 叶子类目名称
- `cate_level1_name`: 一级类目名称

### 销售指标（最近1天）
- `pay_ord_amt_1d`: 支付金额（分）
- `pay_ord_qty_1d`: 支付件数
- `pay_ord_byr_cnt_1d`: 支付买家数
- `pay_ord_amt_1d_931`: 新买家支付金额
- `pay_ord_byr_cnt_1d_931`: 新买家数
- `pay_mord_cnt_1d`: 支付订单数
- `suc_rfd_amt_1d`: 成功退款金额
- `suc_rfd_qty_1d`: 成功退款数量

### 流量指标（最近1天）
- `imps_cnt_1d`: 曝光量
- `ipv_1d`: IPV（商品详情页浏览量）
- `ipv_uv_1d`: 访客数（UV）
- `collect_uv_1d`: 收藏UV
- `cart_uv_1d`: 购物车UV
- `cart_itm_qty_1d`: 加购商品件数

### 营销指标（最近1天）
- `ad_cost_1d`: 广告成本消耗（分）
- `ad_imps_cnt_1d`: 广告曝光数
- `ad_clk_cnt_1d`: 广告点击数

### 价格信息
- `price`: 批发面价
- `avg_post_fee`: 平均邮费

### 商品能力标签
- `is_pwp`: 是否潜力品（1是/0否）
- `is_hqp`: 是否优品（1是/0否）
- `gyp_growth_level`: 工业品商品成长分层

### 服务能力标签
- `is_no_reason_to_return_7d`: 是否支持七天无理由退货（1是/0否）
- `is_24hour_send`: 是否支持24小时发货（1是/0否）
- `is_48hour_send`: 是否支持48小时发货（1是/0否）
- `is_72hour_send`: 是否支持72小时发货（1是/0否）
- `is_15day_free_refund`: 是否支持15天包换（1是/0否）

### 特殊标签
- `is_yx`: 是否严选（1是/0否）
- `is_zdzb`: 是否镇店之宝（1是/0否）
- `is_sjp`: 是否商机品（1是/0否）

## ads_aidata_aide_slr_idx_1d（商家指标表）

用于获取店铺整体情况作为对比基准：

### 店铺整体指标（最近1天）
- `pay_ord_amt_1d_001`: 店铺支付金额（分）
- `pay_ord_byr_cnt_1d_001`: 店铺支付买家数
- `uv_1d_001`: 店铺访客数
- `pv_1d_001`: 店铺浏览量
- `itm_cnt_1d`: 在线商品数
- `pwp_itm_cnt_1d`: 潜力品商品数
- `hqp_itm_cnt_1d`: 优品商品数

## 数据使用说明

1. **金额单位**：所有金额字段单位为"分"，展示时需除以100转换为元
2. **日期范围**：指标均为最近1天数据，业务日期需从用户输入获取
3. **空值处理**：缺失值用0或NULL表示，计算时需妥善处理
4. **Join关联**：通过member_id关联商家表和商品表
