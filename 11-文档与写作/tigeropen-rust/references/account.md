
# Tiger OpenAPI Rust SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::model::trade_requests::*;
use tigeropen::trade::TradeClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let tc = TradeClient::from_config(config.clone());
```

> **约定 / Conventions**
> - 方法接收 `XxxRequest` 结构体（字段为 `Option<T>`），用 `..Default::default()` 补齐
> - 返回**强类型结构体**；单值接口返回 `Option<T>`，需判空
> - `account` 留空时 SDK 自动填入客户端账户
> - Methods take typed request structs with `Option<T>` fields; single-value APIs return `Option<T>`.

---

## 账户列表 / Account List

```rust
let accounts = tc
    .get_managed_accounts(ManagedAccountsRequest::default())
    .await?;
```

返回 `Vec<ManagedAccount>`。字段说明 / Fields:
- 账户号（综合 5~10 位数字，模拟 17 位，环球以 U 开头）
- `capability` — `CASH`（现金）/ `RegTMargin`（保证金）/ `PMGRN`（组合保证金）
- `status` — `Funded` / `Open` / `Pending` / `Rejected` / `Closed`
- 账户类型 — `STANDARD` / `GLOBAL` / `PAPER`

> 方法名是 `get_managed_accounts`，**没有** `accounts()` 方法。
> The method is `get_managed_accounts`; there is no `accounts()`.

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```rust
let assets = tc
    .get_assets(AssetsRequest {
        segment: Some(true),      // 按证券/期货分类
        market_value: Some(true), // 按市场分市值（仅环球账户）
        ..Default::default()
    })
    .await?;
```

返回 `Vec<Asset>`。主要字段：净清算值、可用资金、购买力、现金、
初始/维持保证金要求、浮动与已实现盈亏、按品种分类的 segments。

### 综合/模拟账号 / Standard/Paper Account

```rust
// 返回 Option，需判空
if let Some(prime) = tc
    .get_prime_assets(AssetsRequest {
        segment: Some(true),
        ..Default::default()
    })
    .await?
{
    println!("{:?}", prime);
}
```

`segments` 主要字段：
- `category` — S（证券）/ C（期货）/ F（基金）/ D（数字货币）
- `capability` — RegTMargin / Cash
- 购买力、可用资金、现金余额、净清算值
- 初始保证金、维持保证金（低于此值会强平）
- 浮动盈亏、按币种（USD/HKD/SGD/CNH）细分

---

## 账户持仓 / Account Positions

```rust
let positions = tc
    .get_positions(PositionsRequest {
        sec_type: Some("STK".to_string()),  // STK/OPT/FUT，默认 STK
        currency: Some("ALL".to_string()),  // ALL/USD/HKD/CNH
        market: Some("ALL".to_string()),    // ALL/US/HK/CN
        ..Default::default()
    })
    .await?;

for p in &positions {
    // 字段均为 Option<T>
    println!(
        "{:?} position={:?} cost={:?} mktValue={:?}",
        p.symbol, p.position, p.average_cost, p.market_value
    );
}
```

返回 `Vec<Position>`，字段均为 `Option<T>`：`symbol`、`sec_type`、`market`、
`currency`、`position`、`average_cost`、`market_value` 等。

### 期权持仓 / Option Positions

```rust
let opt_positions = tc
    .get_positions(PositionsRequest {
        sec_type: Some("OPT".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```rust
let analytics = tc
    .get_analytics_asset(AnalyticsAssetRequest {
        start_date: Some("2026-01-01".to_string()),
        end_date: Some("2026-01-31".to_string()),
        seg_type: Some("SEC".to_string()),  // SEC / FUT
        ..Default::default()
    })
    .await?;
```

> 方法名是 `get_analytics_asset`，**不是** `prime_analytics_asset`。
> The method is `get_analytics_asset`.

返回内容包含汇总（盈亏金额、收益率、年化收益率）与按日历史
（日期毫秒时间戳、总资产、当日盈亏、现金余额、持仓市值、入金、出金）。

---

## 最大可交易数量 / Estimate Tradable Quantity

```rust
let qty = tc
    .get_estimate_tradable_quantity(EstimateTradableQuantityRequest {
        symbol: Some("AAPL".to_string()),
        sec_type: Some("STK".to_string()),
        action: Some("BUY".to_string()),
        order_type: Some("LMT".to_string()),
        limit_price: Some(150.0),
        ..Default::default()
    })
    .await?;
```

返回现金可买/卖数量、融资融券可买/卖数量、持仓数量与持仓可交易数量。

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```rust
let avail = tc
    .get_segment_fund_available(SegmentFundRequest {
        from_segment: Some("SEC".to_string()),  // SEC / FUT
        currency: Some("USD".to_string()),
        ..Default::default()
    })
    .await?;
```

### 发起转账 / Transfer

```rust
// 返回 Option，需判空
if let Some(result) = tc
    .transfer_segment_fund(SegmentFundRequest {
        from_segment: Some("SEC".to_string()),
        to_segment: Some("FUT".to_string()),
        currency: Some("USD".to_string()),
        amount: Some(1000.0),
        ..Default::default()
    })
    .await?
{
    println!("{:?}", result);
}
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

> 方法名是 `transfer_segment_fund`（动词在前），**不是** `segment_fund_transfer`。
> The method is `transfer_segment_fund`.

### 撤销转账与历史 / Cancel & History

```rust
let cancelled = tc
    .cancel_segment_fund(SegmentFundRequest {
        id: Some("transfer_id".to_string()),
        ..Default::default()
    })
    .await?;

let history = tc
    .get_segment_fund_history(SegmentFundRequest {
        limit: Some(20),
        ..Default::default()
    })
    .await?;
```

---

## 出入金记录 / Funding Records

SDK **没有** `deposit_withdraw` 方法，用以下两个接口 / There is no `deposit_withdraw`; use:

```rust
let funding = tc
    .get_funding_history(FundingHistoryRequest {
        seg_type: Some("SEC".to_string()),
        ..Default::default()
    })
    .await?;

let details = tc
    .get_fund_details(FundDetailsRequest {
        currency: Some("USD".to_string()),
        limit: Some(50),
        ..Default::default()
    })
    .await?;
```

---

## 聚合资产与持仓转移 / Aggregate Assets & Position Transfer

```rust
// 聚合资产（返回 Option）
if let Some(agg) = tc
    .get_aggregate_assets(AggregateAssetsRequest::default())
    .await?
{
    println!("{:?}", agg);
}

// 持仓转移记录
let transfer_records = tc
    .get_position_transfer_records(PositionTransferRecordsRequest::default())
    .await?;
```

---

## 注意事项 / Notes

- 环球账户(Global)用 `get_assets()`，综合/模拟账户(Standard/Paper)用 `get_prime_assets()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 所有方法都是 `async`，请求结构体字段是 `Option<T>`，用 `..Default::default()` 补齐
- 单值接口（`get_prime_assets`、`transfer_segment_fund`、`get_aggregate_assets`）返回 `Option<T>`，需判空
- 维持保证金低于 0 时会触发强制平仓
- 机构用户用 `TradeClient::with_secret_key(...)` 构造客户端，或在请求上设置 `secret_key`
