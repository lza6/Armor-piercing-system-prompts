# Tiger OpenAPI Rust SDK — Trading / 交易

> Rust SDK 交易 API 参考 / Trade API Reference（async / tokio）
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**每步均为必须，缺少任何步骤不得下单**：
1. 调用 `preview_order()` 查看预估佣金和保证金，展示给用户
2. 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止等待用户明确确认**；未收到确认前**禁止调用 `place_order()`**
3. 用户确认后调用 `place_order()`
4. 下单后通过 `get_orders()` 确认订单状态

---

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::model::order::*;
use tigeropen::model::trade_requests::*;
use tigeropen::trade::TradeClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let tc = TradeClient::from_config(config.clone());
let account = config.account.clone();
```

> **约定 / Conventions**
> - 交易方法统一以 `get_` 开头（查询类）；下单相关是 `place_order` / `preview_order` / `modify_order` / `cancel_order`
> - 查询类接收 `XxxRequest` 结构体（字段为 `Option<T>`，用 `..Default::default()`）
> - 下单/改单/撤单返回 `Result<Option<T>, TigerError>`，需判空
> - Query methods take a typed request struct; order operations return `Option<T>`.

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 创建订单 / Create Order

所有 helper 的前 4 个参数固定为 `account, symbol, sec_type, action`。
Every helper starts with `account, symbol, sec_type, action`.

```rust
// 限价单 / Limit order
let limit = limit_order(&account, "AAPL", "STK", "BUY", 100, 150.0);

// 市价单 / Market order
let market = market_order(&account, "AAPL", "STK", "BUY", 100);

// 止损单 / Stop order (aux_price = 触发价)
let stop = stop_order(&account, "AAPL", "STK", "SELL", 100, 145.0);

// 止损限价单 / Stop-limit order (limit_price, aux_price)
let stop_limit = stop_limit_order(&account, "AAPL", "STK", "SELL", 100, 145.0, 148.0);

// 跟踪止损单 / Trailing-stop order (trailing_percent)
let trail = trail_order(&account, "AAPL", "STK", "SELL", 100, 5.0);
```

数量是 `i64`，价格是 `f64` / Quantity is `i64`, prices are `f64`.

### 其他订单类型 / Other Order Types

```rust
// 竞价单 / Auction orders
let auction_lmt = auction_limit_order(&account, "00700", "STK", "BUY", 100, 380.0);
let auction_mkt = auction_market_order(&account, "00700", "STK", "BUY", 100);

// 按金额下单 / Order by amount（碎股）
let amt_mkt = market_order_by_amount(&account, "AAPL", "STK", "BUY", 1000.0);
let amt_lmt = limit_order_by_amount(&account, "AAPL", "STK", "BUY", 1000.0, 150.0);

// 冰山单 / Iceberg order
// iceberg_order(account, symbol, sec_type, action, quantity, limit_price, display_size,
//               min_display_size, check_intervals, price_type, start_time, end_time)
// 后五个参数是 Option，不需要时传 None
let iceberg = iceberg_order(
    &account, "AAPL", "STK", "BUY", 1000, 150.0, 100,
    None, None, None, None, None,
);

// 附加止盈/止损腿 / Attached legs
let with_legs = limit_order_with_legs(
    &account, "AAPL", "STK", "BUY", 100, 150.0,
    vec![
        new_order_leg("PROFIT", 170.0, "DAY"),
        new_order_leg("LOSS", 140.0, "DAY"),
    ],
);

// OCA 一篮子互斥单 / OCA (one-cancels-all)
let oca = oca_order(
    &account, "AAPL", "STK", "SELL", 100,
    vec![
        Box::new(limit_order(&account, "AAPL", "STK", "SELL", 100, 160.0)),
        Box::new(stop_order(&account, "AAPL", "STK", "SELL", 100, 140.0)),
    ],
);
```

### 组合单（多腿期权）/ Combo (Multi-leg Option) Order

```rust
// combo_order(account, action, quantity, order_type, legs,
//             combo_type, limit_price, aux_price, trailing_percent)
// 注意 quantity 在 order_type 之前，后四个参数是 Option
let legs = vec![
    contract_leg("AAPL", "OPT", "BUY", 1, Some("2026-08-21"), Some("150.0"), Some("CALL")),
    contract_leg("AAPL", "OPT", "SELL", 1, Some("2026-08-21"), Some("160.0"), Some("CALL")),
];
let combo = combo_order(
    &account, "BUY", 1, "LMT", legs,
    Some("VERTICAL"), Some(2.5), None, None,
);
let combo_result = tc.place_order(combo).await?;
```

> 组合单通过 `combo_order` + `place_order` 提交，**没有** `place_combo_order` 方法。
> Combo orders go through `combo_order` + `place_order`; there is no `place_combo_order`.

### 预览订单 / Preview Order

```rust
if let Some(preview) = tc.preview_order(limit.clone()).await? {
    println!("{:?}", preview);
}
```

### 提交下单 / Submit Order

```rust
let placed = tc.place_order(limit.clone()).await?;
let order_id = match &placed {
    Some(r) => r.id,
    None => 0,
};
println!("order id: {}", order_id);
```

### 修改订单 / Modify Order

```rust
let mut modify_req = limit.clone();
modify_req.limit_price = Some(155.0);
let modified = tc.modify_order(order_id, modify_req).await?;
```

### 取消订单 / Cancel Order

```rust
let cancelled = tc.cancel_order(order_id).await?;
```

`place_order` / `preview_order` / `modify_order` / `cancel_order` 均返回 `Option<T>`，需判空。

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```rust
// 所有订单 / All orders
let all_orders = tc.get_orders(OrdersRequest { limit: Some(50), ..Default::default() }).await?;

// 待成交 / Active (pending)
let active = tc.get_active_orders(OrdersRequest::default()).await?;

// 已成交 / Filled
let filled = tc.get_filled_orders(OrdersRequest { limit: Some(20), ..Default::default() }).await?;

// 已撤销/失效 / Inactive
let inactive = tc.get_inactive_orders(OrdersRequest::default()).await?;

// 单个订单 / Single order
let one = tc.get_order(GetOrderRequest { id: Some(order_id), ..Default::default() }).await?;

// 订单成交明细 / Order transactions
let txns = tc
    .get_order_transactions(OrderTransactionsRequest {
        order_id: Some(order_id),
        ..Default::default()
    })
    .await?;
```

`account` 字段留空时 SDK 会自动填入客户端账户 / The SDK fills `account` automatically when unset.

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```rust
let positions = tc.get_positions(PositionsRequest::default()).await?;
for p in &positions {
    // 字段均为 Option<T>
    println!(
        "{:?} position={:?} cost={:?} mktValue={:?}",
        p.symbol, p.position, p.average_cost, p.market_value
    );
}
```

可按 `symbol`、`sec_type`、`market`、`currency` 等过滤 / Filter by symbol/sec_type/market/currency.

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```rust
// 普通/环球账户 / Standard & Global
let assets = tc.get_assets(AssetsRequest::default()).await?;

// 综合账户（Prime）—— 返回 Option
if let Some(prime) = tc.get_prime_assets(AssetsRequest::default()).await? {
    println!("{:?}", prime);
}
```

---

## 合约查询 / Contract Query

```rust
let contracts = tc.get_contract("AAPL", "STK").await?;
let batch = tc.get_contracts(&["AAPL", "TSLA"], "STK").await?;

// sec_type 值: "STK"(股票)/"OPT"(期权)/"FUT"(期货)/"CASH"(外汇)
```

---

## 可交易数量 / Estimate Tradable Quantity

```rust
let est = tc
    .get_estimate_tradable_quantity(EstimateTradableQuantityRequest {
        symbol: Some("AAPL".to_string()),
        sec_type: Some("STK".to_string()),
        action: Some("BUY".to_string()),
        order_type: Some("MKT".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 期权行权 / Option Exercise

```rust
let ex_check = tc.option_exercise_check(OptionExerciseCheckRequest::default()).await?;
let ex_positions = tc.get_option_exercise_positions(OptionExercisePositionRequest::default()).await?;
let ex_submitted = tc.submit_option_exercise(OptionExerciseSubmitRequest::default()).await?;
let ex_records = tc.get_option_exercise_records(OptionExerciseRecordsRequest::default()).await?;
let ex_cancelled = tc.cancel_option_exercise(OptionExerciseCancelRequest::default()).await?;
```

---

## OrderRequest 字段说明 / Order Fields

字段类型均为 `Option<T>` / All fields are `Option<T>`:

| 字段 | 类型 | 说明 |
|-----|------|------|
| `account` | `Option<String>` | 账户（SDK 自动填充） |
| `symbol` | `Option<String>` | 标的代码，如 `"AAPL"` |
| `sec_type` | `Option<String>` | `"STK"` / `"OPT"` / `"FUT"` / `"CASH"` |
| `action` | `Option<String>` | `"BUY"` / `"SELL"` |
| `order_type` | `Option<String>` | `"LMT"` / `"MKT"` / `"STP"` / `"STP_LMT"` / `"TRAIL"` |
| `total_quantity` | `Option<i64>` | 数量 |
| `limit_price` | `Option<f64>` | 限价（LMT/STP_LMT 必填） |
| `aux_price` | `Option<f64>` | 止损触发价（STP/STP_LMT 必填） |
| `trailing_percent` | `Option<f64>` | 跟踪止损百分比 |
| `time_in_force` | `Option<String>` | `"DAY"` / `"GTC"` / `"GTD"` |
| `outside_rth` | `Option<bool>` | 是否允许盘前盘后交易 |
| `expiry` / `strike` / `right` | `Option<String>` | 期权合约要素 |
| `identifier` | `Option<String>` | 期权/期货标准代码 |
| `display_size` | `Option<i64>` | 冰山单展示数量 |
| `contract_legs` / `combo_type` | — | 组合单腿与类型 |
| `oca_orders` | `Option<Vec<Box<OrderRequest>>>` | OCA 互斥子单 |
| `id` / `order_id` | `Option<i64>` | 订单 ID |

---

## 注意事项 / Notes

- 所有方法都是 `async`，需在 tokio 运行时中 `.await`
- 下单/改单/撤单返回 `Option<T>`，务必判空后再取字段
- `combo_order` 的参数顺序是 `(account, action, quantity, order_type, legs, ...)`，
  后四个参数为 `Option`
- 查询类方法接收请求结构体（字段 `Option<T>`），用 `..Default::default()` 补齐
- `place_order()` 成功仅表示订单**已提交**，需通过 `get_orders()` 或推送确认成交
- 期权下单可设置 `expiry`/`strike`/`right`，或直接用 `identifier`
- 机构用户用 `TradeClient::with_secret_key(...)` 构造客户端
