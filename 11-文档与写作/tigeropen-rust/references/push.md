# Tiger OpenAPI Rust SDK — Real-time Push / 实时推送

> Rust SDK 实时推送 API 参考 / Push API Reference（async / tokio）
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 关键约定 / Key Conventions

- `PushClient` 必须包在 `Arc` 中：`connect` 是**自由函数** `connect(&Arc<PushClient>)`，不是方法
- 回调是 `Option<Arc<dyn Fn(pb::XxxData) + Send + Sync>>`，数据类型来自 **protobuf** 的 `pb` 模块
- 订阅统一走 `subscribe(&SubjectType, symbols, account, market)`，后三个参数是 `Option`
- `PushClient` must live in an `Arc`; `connect` is a **free function**, not a method.
  Callbacks take protobuf `pb::*` types. Subscribing goes through one generic
  `subscribe(&SubjectType, symbols, account, market)`.

---

## 完整示例 / Full Example

```rust
use std::sync::Arc;
use tigeropen::config::ClientConfig;
use tigeropen::push::*;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = ClientConfig::builder()
        .properties_file("tiger_openapi_config.properties")
        .build()?;
    let account = config.account.clone();

    // PushClient 必须包在 Arc 中 / must be wrapped in an Arc
    let pc = Arc::new(PushClient::new(config, None));

    pc.set_callbacks(Callbacks {
        on_quote: Some(Arc::new(|data| {
            println!(
                "[行情] {} price={:?} volume={:?}",
                data.symbol, data.latest_price, data.volume
            );
        })),
        on_order: Some(Arc::new(|data| {
            println!(
                "[订单] {} status={} filled={}",
                data.symbol, data.status, data.filled_quantity
            );
        })),
        on_asset: Some(Arc::new(|data| {
            println!(
                "[资产] netLiq={} buyingPower={}",
                data.net_liquidation, data.buying_power
            );
        })),
        on_position: Some(Arc::new(|data| {
            println!(
                "[持仓] {} position={} mktValue={}",
                data.symbol, data.position, data.market_value
            );
        })),
        on_connect: Some(Arc::new(|| println!("推送连接成功"))),
        on_disconnect: Some(Arc::new(|| println!("推送连接断开"))),
        on_error: Some(Arc::new(|e| println!("推送错误: {}", e))),
        on_kickout: Some(Arc::new(|m| println!("被踢下线: {}", m))),
        ..Default::default()
    });

    // connect 是自由函数，接收 &Arc<PushClient> / free function taking &Arc<PushClient>
    connect(&pc).await?;

    // 连接成功后订阅 / Subscribe after connecting
    pc.subscribe(&SubjectType::Quote, Some("AAPL,TSLA"), None, None);
    pc.subscribe(&SubjectType::Asset, None, Some(&account), None);
    pc.subscribe(&SubjectType::Order, None, Some(&account), None);
    pc.subscribe(&SubjectType::Position, None, Some(&account), None);

    tokio::time::sleep(std::time::Duration::from_secs(30)).await;

    pc.disconnect();
    Ok(())
}
```

---

## 回调函数 / Callbacks

`Callbacks` 派生了 `Default`，只需填用到的字段，其余用 `..Default::default()`。
`Callbacks` derives `Default` — set only what you need.

| 回调 Callback | 数据类型 |
|--------------|---------|
| `on_quote` | `pb::QuoteData` |
| `on_quote_bbo` | `pb::QuoteData` |
| `on_tick` | `pb::TradeTickData` |
| `on_full_tick` | `pb::TickData` |
| `on_depth` | `pb::QuoteDepthData` |
| `on_kline` | `pb::KlineData` |
| `on_option` | `pb::QuoteData` |
| `on_future` | `pb::QuoteData` |
| `on_stock_top` | `pb::StockTopData` |
| `on_option_top` | `pb::OptionTopData` |
| `on_asset` | `pb::AssetData` |
| `on_position` | `pb::PositionData` |
| `on_order` | `pb::OrderStatusData` |
| `on_transaction` | `pb::OrderTransactionData` |
| `on_connect` / `on_disconnect` | 无参数 |
| `on_error` / `on_kickout` | `String` |

每个回调的类型是 `Option<Arc<dyn Fn(...) + Send + Sync>>`，因此要写成
`Some(Arc::new(|data| { ... }))`。
Each is `Option<Arc<dyn Fn(...) + Send + Sync>>`, so wrap closures in `Some(Arc::new(...))`.

> 部分 protobuf 字段是 `Option<T>`（如 `latest_price`、`volume`），打印时用 `{:?}`
> 或先 `unwrap_or_default()`；`AssetData` / `PositionData` 的数值字段多为值类型。

---

## 订阅 / Subscribe

```rust
use std::sync::Arc;
use tigeropen::config::ClientConfig;
use tigeropen::push::*;

let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
let account = config.account.clone();
let pc = Arc::new(PushClient::new(config, None));

// 行情类：symbols 传逗号分隔字符串 / Quote-family: comma-separated symbols
pc.subscribe(&SubjectType::Quote, Some("AAPL,TSLA,00700"), None, None);
pc.subscribe(&SubjectType::Tick, Some("AAPL"), None, None);
pc.subscribe(&SubjectType::Depth, Some("AAPL"), None, None);
pc.subscribe(&SubjectType::Kline, Some("AAPL"), None, None);
pc.subscribe(&SubjectType::Option, Some("AAPL"), None, None);
pc.subscribe(&SubjectType::Future, Some("CL2506"), None, None);
pc.subscribe(&SubjectType::QuoteBbo, Some("AAPL"), None, None);
pc.subscribe(&SubjectType::FullTick, Some("AAPL"), None, None);

// 榜单类：用 market 参数 / Ranking lists use the market arg
pc.subscribe(&SubjectType::StockTop, None, None, Some("US"));
pc.subscribe(&SubjectType::OptionTop, None, None, Some("US"));

// 账户类：用 account 参数 / Account-family uses the account arg
pc.subscribe(&SubjectType::Asset, None, Some(&account), None);
pc.subscribe(&SubjectType::Position, None, Some(&account), None);
pc.subscribe(&SubjectType::Order, None, Some(&account), None);
pc.subscribe(&SubjectType::Transaction, None, Some(&account), None);

// 退订 / Unsubscribe — 同样的四参数形式
pc.unsubscribe(&SubjectType::Quote, Some("TSLA"), None, None);
pc.unsubscribe(&SubjectType::Asset, None, Some(&account), None);

// 数字货币与全市场有专用方法 / Dedicated helpers for crypto and whole-market
pc.subscribe_cc(&["BTC/USD"])?;
pc.unsubscribe_cc(Some(&["BTC/USD"]))?;
pc.subscribe_market("US")?;
pc.unsubscribe_market("US")?;

// 查询当前订阅 / Inspect subscriptions
let subs = pc.get_subscriptions();
let acct_subs = pc.get_account_subscriptions();

// 连接状态 / Connection state
let st = pc.state();

// 断开 / Disconnect
pc.disconnect();
```

`SubjectType` 变体：`Quote`、`Tick`、`Depth`、`Option`、`Future`、`Kline`、
`StockTop`、`OptionTop`、`FullTick`、`QuoteBbo`、`Asset`、`Position`、`Order`、
`Transaction`、`Cc`、`Market`。

---

## 注意事项 / Notes

- `PushClient` 必须包在 `Arc` 中；`connect(&pc).await?` 是自由函数调用，不是 `pc.connect()`
- 回调数据类型是 protobuf 生成的 `pb::*`，**不存在** `QuotePushData` / `AssetPushData` 之类的类型
- 首次订阅在 `connect` 成功之后执行
- ✅ **断线重连后 SDK 自动恢复订阅**（`resubscribe()` 见 `src/push/push_client.rs:729`）。请勿在 `on_connect` 中重复订阅，否则会产生重复订阅 / SDK auto-restores subscriptions on reconnect; do NOT re-subscribe in `on_connect`
- 心跳保活由 SDK 自动维护
- `subscribe` 的 `symbols` 是**逗号分隔的字符串**，不是切片
