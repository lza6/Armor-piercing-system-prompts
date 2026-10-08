# Tiger OpenAPI Rust SDK — Market Data / 行情查询

> Rust SDK 行情 API 参考 / Quote API Reference（async / tokio）
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::model::quote_requests::*;
use tigeropen::quote::QuoteClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let qc = QuoteClient::from_config(config.clone());
```

> **命名与入参约定 / Naming & requests**
> - 行情方法统一以 `get_` 开头（`get_market_state`、`get_kline` …）
> - 多数方法接收 `XxxRequest` 结构体；字段是 `Option<T>`，配合 `..Default::default()` 使用
> - 返回**强类型结构体**，不是 `serde_json::Value`
> - `HttpClient` 的正确 use 路径是 `tigeropen::client::http_client::HttpClient`
>   （`tigeropen::client::HttpClient` 没有 re-export）
> - Quote methods are prefixed `get_`; requests are structs with `Option<T>` fields —
>   use `..Default::default()`. Returns are typed structs.

---

## 市场状态 / Market State

```rust
// market: "US" / "HK" / "CN" / "SG"
let states = qc.get_market_state("US").await?;
for s in &states {
    println!("{} {} {} {}", s.market, s.market_status, s.status, s.open_time);
}
```

返回 `Vec<MarketState>`：`market`、`market_status`、`status`、`open_time`。

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```rust
let briefs = qc
    .get_real_time_quote(BriefRequest {
        symbols: Some(vec!["AAPL".to_string(), "TSLA".to_string()]),
        ..Default::default()
    })
    .await?;

for b in &briefs {
    println!(
        "{} latest={:.2} bid={:.2} ask={:.2} vol={} preClose={:.2}",
        b.symbol, b.latest_price, b.bid_price, b.ask_price, b.volume, b.pre_close
    );
}
```

返回 `Vec<Brief>`。常用字段：`symbol`、`latest_price`、`latest_time`、
`open`/`high`/`low`/`close`、`pre_close`、`ask_price`/`ask_size`、`bid_price`/`bid_size`、`volume`。

`get_brief` 与 `get_real_time_quote` 入参、返回一致 / `get_brief` is an equivalent alias.

`BriefRequest` 可选字段：`include_hour_trading: Option<bool>`、`sec_type`、`lang`。

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```rust
// period: "day"/"week"/"month"/"year"/"1min"/"5min"/"15min"/"30min"/"60min"
let klines = qc
    .get_kline(KlineRequest {
        symbols: Some(vec!["AAPL".to_string()]),
        period: Some("day".to_string()),
        limit: Some(30),
        ..Default::default()
    })
    .await?;

for k in &klines {
    for it in &k.items {
        println!(
            "{} {} O={:.2} H={:.2} L={:.2} C={:.2} V={}",
            k.symbol, it.time, it.open, it.high, it.low, it.close, it.volume
        );
    }
}
```

返回 `Vec<Kline>`（`symbol`、`period`、`next_page_token`、`items`）；
`KlineItem`：`time`、`open`、`high`、`low`、`close`、`volume`、`amount`。

`KlineRequest` 支持时间范围（`begin_time`/`end_time`，毫秒）或分页（`begin_index`/`end_index`）。
分页查询另有 `get_kline_by_page`。

---

## 分时 / Timeline

```rust
// 注意：接收 &[&str]，不是 Request 结构体
let timelines = qc.get_timeline(&["AAPL", "TSLA"]).await?;
for t in &timelines {
    println!("{} {} preClose={:.2}", t.symbol, t.period, t.pre_close);
    if let Some(bucket) = &t.intraday {
        println!("  intraday points: {}", bucket.items.len());
    }
}
```

返回 `Vec<Timeline>`。分时按时段分桶：`intraday`、`pre_hours`、`after_hours`，
类型均为 `Option<TimelineBucket>`，取值前需判空。历史分时用 `get_timeline_history`。

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```rust
let depths = qc
    .get_quote_depth(QuoteDepthRequest {
        symbols: Some(vec!["AAPL".to_string()]),
        ..Default::default()
    })
    .await?;

for d in &depths {
    for a in &d.asks {
        println!("ASK {:.2} x{} (orders={})", a.price, a.volume, a.count);
    }
    for b in &d.bids {
        println!("BID {:.2} x{} (orders={})", b.price, b.volume, b.count);
    }
}
```

返回 `Vec<Depth>`（`symbol`、`asks`、`bids`）；`DepthLevel`：`price`、`volume`、`count`。

---

## 逐笔成交 / Trade Ticks

```rust
let ticks = qc
    .get_trade_tick(TradeTickRequest {
        symbols: Some(vec!["AAPL".to_string()]),
        limit: Some(50),
        ..Default::default()
    })
    .await?;

for t in &ticks {
    for it in &t.items {
        println!("{} price={:.2} vol={}", it.time, it.price, it.volume);
    }
}
```

返回 `Vec<TradeTick>`（`symbol`、`begin_index`、`end_index`、`items`）。

---

## 期货 / Futures

```rust
let exchanges = qc.get_future_exchange().await?;

// 按交易所代码查合约（位置参数，不是 Request）
let fcontracts = qc.get_future_contracts("CME").await?;

let fquotes = qc
    .get_future_real_time_quote(FutureRealTimeQuoteRequest {
        contract_codes: Some(vec!["CLmain".to_string()]),
        ..Default::default()
    })
    .await?;

let fklines = qc
    .get_future_kline(FutureKlineRequest {
        contract_codes: Some(vec!["CLmain".to_string()]),
        period: Some("day".to_string()),
        ..Default::default()
    })
    .await?;
```

另有 `get_future_depth`、`get_future_trade_ticks`、`get_future_trading_times`、
`get_future_continuous_contracts`、`get_current_future_contract`、`get_all_future_contracts`。

---

## 资金流 / Capital Flow

```rust
// 返回 Option<T>，需判空
if let Some(flow) = qc.get_capital_flow("AAPL", "US", "day").await? {
    println!("{:?}", flow);
}
if let Some(dist) = qc.get_capital_distribution("AAPL", "US").await? {
    println!("{:?}", dist);
}
```

> `get_capital_flow` / `get_capital_distribution` 返回 `Result<Option<T>, TigerError>`。

---

## 公司行为 / Corporate Actions

```rust
use tigeropen::model::quote::CorporateActionRequest;

// 注意：CorporateActionRequest 的字段是 String / Vec<String>，不是 Option
// action_type 由方法内部设置，调用方无需填
let changes = qc
    .get_corporate_symbol_change(CorporateActionRequest {
        market: "US".to_string(),
        begin_date: "2026-01-01".to_string(),
        end_date: "2026-08-01".to_string(),
        ..Default::default()
    })
    .await?;

let delistings = qc
    .get_corporate_delisting(CorporateActionRequest {
        market: "US".to_string(),
        ..Default::default()
    })
    .await?;

let ipos = qc
    .get_corporate_ipo(CorporateActionRequest {
        market: "US".to_string(),
        ..Default::default()
    })
    .await?;
```

拆股/派息/财报日历分别用 `get_corporate_split`、`get_corporate_dividend`、
`get_corporate_earnings_calendar`。

---

## 其他行情能力 / Other Quote APIs

```rust
// 全量代码 / All symbols
let symbols = qc
    .get_symbols(SymbolsRequest {
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;

// 行情权限 / Quote permissions
let perms = qc
    .get_quote_permission(QuotePermissionRequest::default())
    .await?;
let grabbed = qc.grab_quote_permission().await?;
```

还封装了基金（`get_fund_symbols` / `get_fund_contracts` / `get_fund_quote`）、
窝轮（`get_warrant_briefs` / `get_warrant_filter`）、行业（`get_industry_list` /
`get_industry_stocks`）、财务（`get_financial_daily` / `get_financial_report`）、
选股器（`market_scanner` / `get_market_scanner_tags`）、夜盘（`get_quote_overnight`）
与交易日历（`get_trading_calendar`）。

---

## 直接调用 API / Raw API Call

```rust
use tigeropen::client::http_client::HttpClient;

let http_client = HttpClient::new(config.clone());
let raw = http_client
    .execute("quote_real_time", r#"{"symbols":["AAPL","TSLA"]}"#)
    .await?;
println!("{}", raw);
```

第二个参数是 **JSON 字符串**，返回 **String**；**没有** `execute_raw` 方法。
Second arg is a **JSON string**, returns a **String**; there is no `execute_raw`.

---

## 注意事项 / Notes

- 所有方法都是 `async`，需在 tokio 运行时中 `.await`
- 请求结构体字段是 `Option<T>`，务必用 `..Default::default()` 补齐
- `get_timeline` 接收 `&[&str]`；`get_future_contracts`、`get_capital_flow` 等接收位置参数
- 资金流类接口返回 `Option<T>`，需判空
- 分时数据分 `intraday`/`pre_hours`/`after_hours` 三桶，均为 `Option`
- `HttpClient` 的正确 use 路径是 `tigeropen::client::http_client::HttpClient`
- 行情数据需要对应市场的行情权限；期权行情需要单独开通期权行情权限
