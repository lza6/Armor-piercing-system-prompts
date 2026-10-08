
# Tiger OpenAPI Rust SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

1. **查到期日 Get expirations**: `qc.get_option_expiration(&["AAPL"], Some("US"))`
2. **查期权链 Get chain**: `qc.get_option_chain(OptionChainRequest { .. })`
3. **查行情 Get quotes**: `qc.get_option_brief(&[..])` / `qc.get_option_quote(OptionQuoteRequest { .. })`

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股：`00700` → `TCH`（腾讯）
- 用 `qc.get_option_symbols()` 查询港股期权代码映射

---

## 初始化 / Initialize

```rust
use tigeropen::config::ClientConfig;
use tigeropen::model::order::*;
use tigeropen::model::quote_requests::*;
use tigeropen::model::trade_requests::*;
use tigeropen::quote::QuoteClient;
use tigeropen::trade::TradeClient;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let qc = QuoteClient::from_config(config.clone());
let tc = TradeClient::from_config(config.clone());
let account = config.account.clone();
```

> **关键约定 / Key conventions**
> - 期权方法统一以 `get_option_` 开头
> - **到期日 `expiry` 是 `i64` 毫秒时间戳**，不是日期字符串
> - 请求结构体中的 `option_basic` / `option_query` / `contracts` 字段名各接口不同，注意区分
> - `expiry` is an **`i64` millisecond timestamp**, not a date string.

---

## 期权到期日 / Option Expirations

```rust
// 位置参数：symbols 切片 + Option<market>
let expirations = qc.get_option_expiration(&["AAPL"], Some("US")).await?;
for e in &expirations {
    println!("{} {:?} {:?}", e.symbol, e.dates, e.timestamps);
}
```

返回 `Vec<OptionExpiration>`：`symbol`、`option_symbols`、`dates`（日期字符串）、
`timestamps`（毫秒）、`periods`（`m`=月 / `w`=周 / `q`=季）、`counts`。

---

## 期权链 / Option Chain

```rust
// OptionChainItem.symbol 是 String，expiry 是 i64 毫秒
let chains = qc
    .get_option_chain(OptionChainRequest {
        option_basic: Some(vec![OptionChainItem {
            symbol: "AAPL".to_string(),
            expiry: 1787356800000,
        }]),
        return_greek_value: Some(true),
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;
```

可选 `option_filter: Option<OptionChainFilter>` 按隐含波动率、Delta、持仓量等筛选。

返回 `Vec<OptionChain>`，含 call/put 合约的 identifier、行权价、买卖价、成交量、
持仓量、隐含波动率与希腊字母（需 `return_greek_value: Some(true)`）。

---

## 港股期权代码映射 / HK Option Symbol Mapping

```rust
let opt_symbols = qc
    .get_option_symbols(OptionSymbolsRequest {
        market: Some("HK".to_string()),
        lang: Some("en_US".to_string()),
        ..Default::default()
    })
    .await?;
// 返回 Vec<OptionSymbol>：期权 symbol（如 "TCH.HK"）、标的名称、正股代码
```

---

## 期权实时行情 / Option Brief

### 按 identifier 查询（最简）/ By identifier

```rust
let opt_briefs = qc.get_option_brief(&["AAPL  260821C00150000"]).await?;
```

期权代码格式：标的（6 位，右侧空格填充）+ YYMMDD + C/P + 行权价×1000（8 位）。
SDK 内部会用 `OptionContractItem::from_occ()` 解析。

### 按合约要素查询 / By contract fields

```rust
let opt_quotes = qc
    .get_option_quote(OptionQuoteRequest {
        // OptionContractItem 的四个字段都是必填（非 Option）
        option_basic: Some(vec![OptionContractItem {
            symbol: "AAPL".to_string(),
            expiry: 1787356800000,      // i64 毫秒
            right: "CALL".to_string(),
            strike: "150.0".to_string(), // 小数位须和期权链一致
        }]),
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;
```

两者都返回 `Vec<OptionBrief>`。

---

## 期权深度行情 / Option Depth

```rust
// 字段名是 option_basic，元素类型是 OptionQueryItem（字段均为 Option）
let opt_depth = qc
    .get_option_depth(OptionDepthRequest {
        option_basic: Some(vec![OptionQueryItem {
            symbol: Some("AAPL".to_string()),
            expiry: Some(1787356800000),
            right: Some("PUT".to_string()),
            strike: Some("210.0".to_string()),
            ..Default::default()
        }]),
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 期权逐笔成交 / Option Trade Ticks

```rust
// 仅美股期权；注意字段名是 contracts
let opt_ticks = qc
    .get_option_trade_ticks(OptionTradeTicksRequest {
        contracts: Some(vec![OptionQueryItem {
            symbol: Some("AAPL".to_string()),
            expiry: Some(1787356800000),
            right: Some("PUT".to_string()),
            strike: Some("185.0".to_string()),
            ..Default::default()
        }]),
        ..Default::default()
    })
    .await?;
```

---

## 期权 K 线 / Option K-line

```rust
// 字段名是 option_query，元素类型是 OptionKlineItem
// OptionKlineItem 不实现 Default，所有字段都必须显式给出
// OptionKlineItem does NOT implement Default — list every field
let opt_klines = qc
    .get_option_kline(OptionKlineRequest {
        option_query: Some(vec![OptionKlineItem {
            symbol: "AAPL".to_string(),
            expiry: 1787356800000,
            right: "CALL".to_string(),
            strike: "170.0".to_string(),
            period: "1min".to_string(),   // day/1min/5min/30min/60min
            begin_time: None,
            end_time: None,
            limit: Some(10),
            sort_dir: Some("DESC".to_string()),
        }]),
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 期权分时 / Option Timeline

```rust
// 目前仅支持港股期权；字段名是 option_query
let opt_timeline = qc
    .get_option_timeline(OptionTimelineRequest {
        option_query: Some(vec![OptionQueryItem {
            symbol: Some("ALB.HK".to_string()),
            expiry: Some(1753878054000),
            right: Some("CALL".to_string()),
            strike: Some("117.50".to_string()),
            ..Default::default()
        }]),
        market: Some("HK".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 期权分析 / Option Analysis

```rust
let analysis = qc
    .get_option_analysis(OptionAnalysisRequest {
        symbols: Some(vec!["AAPL".to_string()]),
        period: Some("52week".to_string()),  // 3year/52week/26week/13week
        require_volatility_list: Some(true),
        market: Some("US".to_string()),
        ..Default::default()
    })
    .await?;
```

需要为每个标的单独设置 period 时，用 `symbol_items: Option<Vec<OptionAnalysisSymbol>>`。
返回 30 日隐含波动率、历史波动率、IV/HV 比率、Call/Put 比率、IV 百分位与排名。

---

## 单腿期权下单 / Single-leg Option Order

```rust
let mut opt_order = limit_order(&account, "AAPL", "OPT", "BUY", 1, 5.0);
opt_order.expiry = Some("20260821".to_string());  // 下单用 YYYYMMDD 字符串
opt_order.strike = Some("150.0".to_string());
opt_order.right = Some("CALL".to_string());
opt_order.currency = Some("USD".to_string());

let opt_preview = tc.preview_order(opt_order.clone()).await?;
let opt_placed = tc.place_order(opt_order.clone()).await?;
```

也可直接用 identifier / Or set the standard identifier:

```rust
let mut by_identifier = limit_order(&account, "AAPL", "OPT", "BUY", 1, 5.0);
by_identifier.identifier = Some("AAPL  260821C00150000".to_string());
```

> 注意：**行情接口**的 `expiry` 是 `i64` 毫秒，**下单结构体**的 `expiry` 是 `Option<String>`（YYYYMMDD）。
> Quote APIs use `i64` ms for expiry; the order struct uses `Option<String>` (YYYYMMDD).

---

## 多腿组合策略 / Multi-leg Combo Strategies

```rust
// combo_order(account, action, quantity, order_type, legs,
//             combo_type, limit_price, aux_price, trailing_percent)
let spread_legs = vec![
    contract_leg("AAPL", "OPT", "BUY", 1, Some("2026-08-21"), Some("145.0"), Some("CALL")),
    contract_leg("AAPL", "OPT", "SELL", 1, Some("2026-08-21"), Some("155.0"), Some("CALL")),
];
let spread = combo_order(
    &account, "BUY", 1, "LMT", spread_legs,
    Some("VERTICAL"), Some(3.0), None, None,
);
let spread_result = tc.place_order(spread).await?;
```

> 组合单通过 `combo_order` + `place_order` 提交，**没有** `place_combo_order` 方法。
> Combo orders go through `combo_order` + `place_order`; there is no `place_combo_order`.

### 组合策略类型 / Combo Strategy Types

| ComboType | 策略 Strategy | 说明 |
|-----------|--------------|------|
| `VERTICAL` | 垂直价差 | 同到期日不同行权价 |
| `STRADDLE` | 跨式 | 同行权价同到期日 Call+Put |
| `STRANGLE` | 宽跨式 | 不同行权价同到期日 |
| `CALENDAR` | 日历价差 | 同行权价不同到期日 |
| `DIAGONAL` | 对角线价差 | 不同行权价不同到期日 |
| `COVERED` | 备兑 | 持有股票+卖 Call |
| `PROTECTIVE` | 保护性 | 持有股票+买 Put |
| `SYNTHETIC` | 合成 | 合成多/空头 |
| `CUSTOM` | 自定义 | 4 条腿组合（Iron Condor 等） |

---

## 期权行权 / Option Exercise

```rust
let ex_check = tc.option_exercise_check(OptionExerciseCheckRequest::default()).await?;
let ex_positions = tc
    .get_option_exercise_positions(OptionExercisePositionRequest::default())
    .await?;
let ex_submitted = tc.submit_option_exercise(OptionExerciseSubmitRequest::default()).await?;
let ex_records = tc
    .get_option_exercise_records(OptionExerciseRecordsRequest::default())
    .await?;
let ex_cancelled = tc.cancel_option_exercise(OptionExerciseCancelRequest::default()).await?;
```

---

## 查询期权持仓 / Query Option Positions

```rust
let opt_positions = tc
    .get_positions(PositionsRequest {
        sec_type: Some("OPT".to_string()),
        ..Default::default()
    })
    .await?;
```

---

## 注意事项 / Notes

- 所有方法都是 `async`，请求结构体字段多为 `Option<T>`，用 `..Default::default()` 补齐
- **行情接口的 `expiry` 是 `i64` 毫秒时间戳；下单结构体的 `expiry` 是 `Option<String>`（YYYYMMDD）**
- 各接口的合约字段名不同：`option_basic`（chain/quote/depth）、`option_query`（kline/timeline）、
  `contracts`（trade_ticks）
- `OptionChainItem` / `OptionContractItem` / `OptionKlineItem` 的字段是**必填**（非 Option），
  `OptionQueryItem` 的字段是 `Option`
- **SDK 没有 `option_indicator` 方法**；单合约 Greeks 通过
  `get_option_chain(return_greek_value: Some(true))` 或 `get_option_brief` 获取
- **SDK 没有 `place_combo_order` 方法**；组合单用 `combo_order` + `place_order`
- 港股期权需先用 `get_option_symbols()` 获取代码映射
- 期权每张合约通常代表 100 股标的；行权价小数位须和期权链一致
- 期权行情需要期权行情权限
