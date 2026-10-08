# Tiger OpenAPI Go SDK — Real-time Push / 实时推送

> Go SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/push"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
pc := push.NewPushClient(cfg)
```

> **回调数据类型 / Callback payload types**: 推送回调的数据是 **protobuf 生成的类型**，
> 位于 `push/pb` 包（`*pb.QuoteData`、`*pb.OrderStatusData` …），不是 `push` 包下的类型。
> 大部分字段是**指针**，请使用生成的 `GetXxx()` 访问器安全取值，避免解引用 nil。
> Payloads are protobuf types in `push/pb`; most fields are pointers, so use the generated
> `GetXxx()` accessors rather than dereferencing directly.

---

## 完整示例 / Full Example

```go
package main

import (
    "fmt"
    "os"
    "os/signal"
    "syscall"

    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/push"
    "github.com/tigerfintech/openapi-go-sdk/push/pb"
)

func main() {
    cfg, _ := config.NewClientConfig(
        config.WithPropertiesFile("tiger_openapi_config.properties"),
    )

    pc := push.NewPushClient(cfg)

    // 设置回调 / Set callbacks
    // ⚠️ 首次订阅在 OnConnect 中执行 / Place the initial subscribe inside OnConnect
    pc.SetCallbacks(push.Callbacks{
        OnQuote: func(data *pb.QuoteData) {
            fmt.Printf("[行情] %s 最新价: %.2f 成交量: %d\n",
                data.GetSymbol(), data.GetLatestPrice(), data.GetVolume())
        },
        OnOrder: func(data *pb.OrderStatusData) {
            fmt.Printf("[订单] %s 状态: %s 已成交: %d\n",
                data.GetSymbol(), data.GetStatus(), data.GetFilledQuantity())
        },
        OnAsset: func(data *pb.AssetData) {
            fmt.Printf("[资产] 净资产: %.2f 购买力: %.2f\n",
                data.GetNetLiquidation(), data.GetBuyingPower())
        },
        OnPosition: func(data *pb.PositionData) {
            fmt.Printf("[持仓] %s 数量: %d 市值: %.2f\n",
                data.GetSymbol(), data.GetPosition(), data.GetMarketValue())
        },
        OnConnect: func() {
            fmt.Println("推送连接成功")
            // 首次连接后订阅 / Subscribe after the first connect.
            // SDK 重连成功后会自动恢复订阅，无需在此重复订阅
            // SDK auto-restores subscriptions on reconnect — do not re-subscribe here.
            pc.SubscribeQuote([]string{"AAPL", "TSLA"})
            pc.SubscribeAsset("")    // 资产变动，"" 表示使用默认账户
            pc.SubscribeOrder("")    // 订单状态变动
            pc.SubscribePosition("") // 持仓变动
        },
        OnDisconnect: func() { fmt.Println("推送连接断开") },
        OnError:      func(err error) { fmt.Printf("推送错误: %v\n", err) },
        OnKickout:    func(msg string) { fmt.Printf("被踢下线: %s\n", msg) },
    })

    // 连接 / Connect
    if err := pc.Connect(); err != nil {
        panic(err)
    }
    defer pc.Disconnect()

    // 等待退出 / Wait for signal
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    <-sig

    // 退订 / Unsubscribe
    pc.UnsubscribeQuote([]string{"AAPL", "TSLA"})
}
```

---

## 回调函数 / Callbacks

全部定义在 `push.Callbacks`（`push/callbacks.go`）/ All defined on `push.Callbacks`:

| 回调 Callback | 触发时机 | 数据类型 |
|--------------|---------|---------|
| `OnQuote` | 基础行情更新 | `*pb.QuoteData` |
| `OnQuoteBBO` | 最优买卖价更新 | `*pb.QuoteData` |
| `OnTick` | 逐笔成交 | `*pb.TradeTickData` |
| `OnFullTick` | 全量逐笔 | `*pb.TickData` |
| `OnDepth` | 盘口深度 | `*pb.QuoteDepthData` |
| `OnKline` | K 线更新 | `*pb.KlineData` |
| `OnOption` | 期权行情 | `*pb.QuoteData` |
| `OnFuture` | 期货行情 | `*pb.QuoteData` |
| `OnStockTop` | 股票榜单 | `*pb.StockTopData` |
| `OnOptionTop` | 期权榜单 | `*pb.OptionTopData` |
| `OnAsset` | 账户资产变动 | `*pb.AssetData` |
| `OnPosition` | 持仓变动 | `*pb.PositionData` |
| `OnOrder` | 订单状态变化 | `*pb.OrderStatusData` |
| `OnTransaction` | 成交明细 | `*pb.OrderTransactionData` |
| `OnConnect` | 连接成功 | - |
| `OnDisconnect` | 连接断开 | - |
| `OnError` | 发生错误 | `error` |
| `OnKickout` | 被服务端踢下线 | `string` |

---

## 订阅方法 / Subscribe Methods

```go
// 行情类：接收 symbol 列表 / Quote-family: take a symbol slice
pc.SubscribeQuote([]string{"AAPL", "TSLA", "00700"})
pc.UnsubscribeQuote([]string{"TSLA"})

pc.SubscribeTick([]string{"AAPL"})
pc.SubscribeDepth([]string{"AAPL"})
pc.SubscribeKline([]string{"AAPL"})
pc.SubscribeOption([]string{"AAPL  260821C00150000"})
pc.SubscribeFuture([]string{"CLmain"})
pc.SubscribeCc([]string{"BTC/USD"})

// 榜单 / Ranking lists
pc.SubscribeStockTop("US", []string{"changeRate"})
pc.SubscribeOptionTop("US", []string{"volume"})

// 全市场 / Whole market
pc.SubscribeMarket("US")

// 账户类：订阅传 account（"" 用默认账户），退订【不带参数】
// Account-family: subscribe takes an account; UNSUBSCRIBE TAKES NO ARGUMENTS
account := cfg.Account
pc.SubscribeAsset(account)
pc.SubscribeOrder(account)
pc.SubscribePosition(account)
pc.SubscribeTransaction(account)

pc.UnsubscribeAsset()
pc.UnsubscribeOrder()
pc.UnsubscribePosition()
pc.UnsubscribeTransaction()

// 查询当前订阅 / Inspect current subscriptions
subs := pc.GetSubscriptions()
acctSubs := pc.GetAccountSubscriptions()

// 连接状态 / Connection state
state := pc.State()

// 断开连接 / Disconnect
pc.Disconnect()
```

> ⚠️ 账户类退订方法**无参数**：`UnsubscribeAsset()`，不是 `UnsubscribeAsset(account)`。
> Account unsubscribe methods take **no** arguments.

---

## pb.QuoteData 常用字段 / Common Fields

指针字段请用 `GetXxx()` 访问 / Use `GetXxx()` for pointer fields:

| 字段 Field | 访问器 | 说明 |
|-----------|-------|------|
| `Symbol` | `GetSymbol()` | 标的代码 |
| `LatestPrice` | `GetLatestPrice()` | 最新价 |
| `Volume` | `GetVolume()` | 成交量 |
| `Amount` | `GetAmount()` | 成交额 |
| `Open`/`High`/`Low` | `GetOpen()` … | 开/高/低 |
| `PreClose` | `GetPreClose()` | 昨收 |
| `AskPrice`/`AskSize` | `GetAskPrice()` … | 卖一价/量 |
| `BidPrice`/`BidSize` | `GetBidPrice()` … | 买一价/量 |
| `MarketStatus` | `GetMarketStatus()` | 市场状态 |
| `Timestamp` | `GetTimestamp()` | 时间戳（毫秒） |

## pb.AssetData 字段 / AssetData Fields

值字段，可直接访问 / Value fields, safe to read directly:

| 字段 Field | 说明 |
|-----------|------|
| `Account` | 账户号 |
| `Currency` / `SegType` | 币种 / 分段 |
| `NetLiquidation` | 净资产 |
| `CashBalance` | 现金余额 |
| `BuyingPower` | 购买力 |
| `AvailableFunds` | 可用资金 |
| `ExcessLiquidity` | 剩余流动性 |
| `EquityWithLoan` | 含借贷权益 |
| `GrossPositionValue` | 持仓总市值 |
| `InitMarginReq` / `MaintMarginReq` | 初始 / 维持保证金 |

## pb.OrderStatusData 字段 / OrderStatusData Fields

| 字段 Field | 说明 |
|-----------|------|
| `Id` | 订单 ID |
| `Symbol` / `Identifier` | 标的 / 标准代码 |
| `Action` / `OrderType` | 方向 / 订单类型 |
| `Status` | 订单状态 |
| `TotalQuantity` / `FilledQuantity` | 总量 / 已成交量 |
| `AvgFillPrice` | 成交均价 |
| `LimitPrice` / `StopPrice` | 限价 / 止损价 |
| `RealizedPnl` | 已实现盈亏 |
| `CanModify` / `CanCancel` | 是否可改单 / 可撤单 |

---

## 注意事项 / Notes

- **首次订阅放在 `OnConnect` 回调中**，连接成功后才能订阅
- SDK 自动处理断线重连与心跳保活，无需手动重连
- ✅ **断线重连后 SDK 自动恢复订阅**（`resubscribe()` 会重放行情与账户订阅，见 `push/push_client.go:468`）。请勿在 `OnConnect` 中重复订阅，否则会产生重复订阅 / SDK auto-restores subscriptions on reconnect; do NOT re-subscribe in `OnConnect` or you will double-subscribe
- 回调数据类型来自 `push/pb` 包，字段多为指针，优先用 `GetXxx()` 访问器
- 账户类退订方法不带参数
- 同一个 `PushClient` 实例只能有一个活跃连接
