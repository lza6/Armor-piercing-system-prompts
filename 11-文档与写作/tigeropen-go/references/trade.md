# Tiger OpenAPI Go SDK — Trading / 交易

> Go SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**每步均为必须，缺少任何步骤不得下单**：
1. 调用 `PreviewOrder()` 查看预估佣金和保证金，展示给用户
2. 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止等待用户明确确认**；未收到确认前**禁止调用 `PlaceOrder()`**
3. 用户确认后调用 `PlaceOrder()`
4. 下单后通过 `Orders()` 确认订单状态

---

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/client"
    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/model"
    "github.com/tigerfintech/openapi-go-sdk/trade"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

> **入参约定 / Request convention**: 查询类方法接收 `model.XxxRequest` 结构体（Go 是静态类型语言，
> 不能传 `map[string]interface{}`），返回**强类型结构体**。
> Query methods take a typed `model.XxxRequest` struct — not a map — and return typed structs.

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 创建订单 / Create Order

所有 helper 的前 4 个参数固定为 `account, symbol, secType, action`。
Every helper starts with `account, symbol, secType, action`.

```go
account := cfg.Account

// 限价单 / Limit order
limit := model.LimitOrder(account, "AAPL", "STK", "BUY", 100, 150.0)

// 市价单 / Market order
market := model.MarketOrder(account, "AAPL", "STK", "BUY", 100)

// 止损单 / Stop order (auxPrice = 触发价)
stop := model.StopOrder(account, "AAPL", "STK", "SELL", 100, 145.0)

// 止损限价单 / Stop-limit order (limitPrice, auxPrice)
stopLimit := model.StopLimitOrder(account, "AAPL", "STK", "SELL", 100, 145.0, 148.0)

// 跟踪止损单 / Trailing-stop order (trailingPercent)
trail := model.TrailOrder(account, "AAPL", "STK", "SELL", 100, 5.0)
```

数量参数是 `int64`，价格是 `float64` / Quantity is `int64`, prices are `float64`.

### 其他订单类型 / Other Order Types

```go
// 竞价单 / Auction orders
auctionLmt := model.AuctionLimitOrder(account, "00700", "STK", "BUY", 100, 380.0)
auctionMkt := model.AuctionMarketOrder(account, "00700", "STK", "BUY", 100)

// 按金额下单 / Order by amount (碎股)
amtMkt := model.MarketOrderByAmount(account, "AAPL", "STK", "BUY", 1000.0)
amtLmt := model.LimitOrderByAmount(account, "AAPL", "STK", "BUY", 1000.0, 150.0)

// 冰山单 / Iceberg order (displaySize = 每次展示数量)
iceberg := model.IcebergOrder(account, "AAPL", "STK", "BUY", 1000, 150.0, 100)

// 算法单 / Algo order (TWAP/VWAP)
algo := model.AlgoOrder(account, "AAPL", "STK", "BUY", 1000, 150.0, "TWAP",
    model.AlgoParamsRequest{
        StartTime:         "09:30:00",
        EndTime:           "16:00:00",
        ParticipationRate: 0.1,
    })

// OCA 一篮子互斥单 / OCA (one-cancels-all)
leg1 := model.LimitOrder(account, "AAPL", "STK", "SELL", 100, 160.0)
leg2 := model.StopOrder(account, "AAPL", "STK", "SELL", 100, 140.0)
oca := model.OcaOrder(account, "AAPL", "STK", "SELL", 100, []*model.OrderRequest{&leg1, &leg2})

// 附加止盈/止损腿 / Attached legs
withLegs, err := model.LimitOrderWithLegs(account, "AAPL", "STK", "BUY", 100, 150.0,
    []model.OrderLegRequest{
        model.NewOrderLeg("PROFIT", 170.0, "DAY"),
        model.NewOrderLeg("LOSS", 140.0, "DAY"),
    })
```

### 组合单（多腿期权）/ Combo (Multi-leg Option) Order

```go
legs := []model.ContractLegRequest{
    model.NewContractLeg("AAPL", "OPT", "BUY", 1, "2026-08-21", "150", "CALL"),
    model.NewContractLeg("AAPL", "OPT", "SELL", 1, "2026-08-21", "160", "CALL"),
}
combo := model.ComboOrder(account, "BUY", "LMT", 1, legs, "VERTICAL", 2.5, 0, 0)
comboResult, err := tc.PlaceOrder(combo)
```

> 组合单通过 `model.ComboOrder` + `PlaceOrder` 提交，**没有** `PlaceComboOrder` 方法。
> Combo orders go through `model.ComboOrder` + `PlaceOrder`; there is no `PlaceComboOrder`.

### 预览订单 / Preview Order

```go
preview, err := tc.PreviewOrder(limit)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("pass=%v commission=%.2f %s initMargin=%.2f maintMargin=%.2f\n",
    preview.IsPass, preview.Commission, preview.CommissionCurrency,
    preview.InitMargin, preview.MaintMargin)
```

返回 `*model.PreviewResult`：`IsPass`、`Commission`、`CommissionCurrency`、
`InitMargin`/`InitMarginBefore`、`MaintMargin`/`MaintMarginBefore`、`EquityWithLoan`、`MarginCurrency`。

### 提交下单 / Submit Order

```go
placed, err := tc.PlaceOrder(limit)
if err != nil {
    log.Fatal(err)
}
fmt.Println("order id:", placed.ID)
```

返回 `*model.PlaceOrderResult`：`ID`、`OrderID`、`SubIDs`（附加/OCA 子单）、`Orders`。

### 修改订单 / Modify Order

```go
limit.LimitPrice = 155.0
modified, err := tc.ModifyOrder(placed.ID, limit)  // id int64
fmt.Println(modified.ID)
```

### 取消订单 / Cancel Order

```go
cancelled, err := tc.CancelOrder(placed.ID)  // id int64
fmt.Println(cancelled.ID)
```

`ModifyOrder` / `CancelOrder` 返回 `*model.OrderIDResult`（仅含 `ID`）。

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

四个查询方法都接收 `model.OrdersRequest` / All four take a `model.OrdersRequest`:

```go
// 所有订单 / All orders
allOrders, err := tc.Orders(model.OrdersRequest{Limit: 50})

// 待成交订单 / Active (pending) orders
active, err := tc.ActiveOrders(model.OrdersRequest{})

// 已成交订单 / Filled orders
filled, err := tc.FilledOrders(model.OrdersRequest{Limit: 20})

// 已撤销/失效订单 / Inactive orders
inactive, err := tc.InactiveOrders(model.OrdersRequest{})

// 单个订单 / Single order
one, err := tc.GetOrder(model.GetOrderRequest{Id: 12345678})

// 订单成交明细 / Order transactions
txns, err := tc.OrderTransactions(model.OrderTransactionsRequest{OrderId: 12345678})
```

`model.OrdersRequest` 常用字段：`Account`、`Market`、`SecType`、`SegType`、`Symbol`、
`StartDate`/`EndDate`（**毫秒时间戳**）、`Limit`、`States []string`、`ParentId`、`IsBrief`。

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```go
positions, err := tc.Positions(model.PositionsRequest{})
for _, p := range positions {
    fmt.Printf("%s qty=%.2f cost=%.4f mktValue=%.2f unrealPnl=%.2f (%.2f%%)\n",
        p.Symbol, p.PositionQty, p.AverageCost, p.MarketValue,
        p.UnrealizedPnl, p.UnrealizedPnlPercent)
}
```

返回 `[]model.Position`。常用字段：`Symbol`、`SecType`、`Market`、`Currency`、
`Position`（int64）、`PositionQty`（float64，支持碎股）、`SalableQty`、`AverageCost`、
`MarketValue`、`RealizedPnl`、`UnrealizedPnl`、`UnrealizedPnlPercent`。

可按 `Symbol`、`SecType`、`Market`、`Currency`、`SubAccounts`、期权的 `Expiry`/`Strike`/`Right` 过滤。

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```go
// 普通账户资产 / Standard account assets
assets, err := tc.Assets(model.AssetsRequest{})
for _, a := range assets {
    fmt.Printf("%s %s netLiq=%.2f cash=%.2f buyingPower=%.2f\n",
        a.Account, a.Currency, a.NetLiquidation, a.CashValue, a.BuyingPower)
}

// 综合账户（Prime）资产 / Prime account assets
prime, err := tc.PrimeAssets(model.AssetsRequest{})
for _, seg := range prime.Segments {
    fmt.Printf("%s %s netLiq=%.2f cashBalance=%.2f initMargin=%.2f\n",
        seg.Category, seg.Currency, seg.NetLiquidation, seg.CashBalance, seg.InitMargin)
}
```

`Assets` 返回 `[]model.Asset`（`NetLiquidation`、`CashValue`、`BuyingPower`、
`RealizedPnL`、`UnrealizedPnL`、`Segments`）。
`PrimeAssets` 返回 `*model.PrimeAsset`（`AccountID`、`UpdateTimestamp`、`Segments`），
明细在 `Segments []model.PrimeAssetSegment` 中，按 `Category`（securities/commodities）分段。

---

## 合约查询 / Contract Query

```go
contracts, err := tc.Contract("AAPL", "STK")
batch, err := tc.Contracts([]string{"AAPL", "TSLA"}, "STK")
optContract, err := tc.QuoteContract("AAPL", "OPT", "2026-08-21")

// secType 值: "STK"(股票)/"OPT"(期权)/"FUT"(期货)/"CASH"(外汇)
```

---

## 可交易数量 / Estimate Tradable Quantity

```go
est, err := tc.EstimateTradableQuantity(model.EstimateTradableQuantityRequest{
    Account: account, Symbol: "AAPL", SecType: "STK", Action: "BUY", OrderType: "MKT",
})
```

---

## 期权行权 / Option Exercise

```go
check, err := tc.OptionExerciseCheck(model.OptionExerciseCheckRequest{Account: account})
exercisable, err := tc.OptionExercisePositions(model.OptionExercisePositionRequest{Account: account})
ok, err := tc.OptionExerciseSubmit(model.OptionExerciseSubmitRequest{Account: account})
records, err := tc.OptionExerciseRecords(model.OptionExercisePageRequest{Account: account})
cancelled2, err := tc.OptionExerciseCancel(model.OptionExerciseCancelRequest{Account: account})
```

`OptionExerciseSubmit` / `OptionExerciseCancel` 返回 `(bool, error)`。

---

## model.OrderRequest 字段说明 / Order Fields

| 字段 | 类型 | 说明 |
|-----|------|------|
| `Account` | string | 账户 |
| `Symbol` | string | 标的代码，如 `"AAPL"` |
| `SecType` | string | `"STK"` / `"OPT"` / `"FUT"` / `"CASH"` |
| `Action` | string | `"BUY"` / `"SELL"` |
| `OrderType` | string | `"LMT"` / `"MKT"` / `"STP"` / `"STP_LMT"` / `"TRAIL"` |
| `TotalQuantity` | **int64** | 数量 |
| `LimitPrice` | float64 | 限价（LMT/STP_LMT 必填） |
| `AuxPrice` | float64 | 止损触发价（STP/STP_LMT 必填） |
| `TrailingPercent` | float64 | 跟踪止损百分比 |
| `TimeInForce` | string | `"DAY"` / `"GTC"` / `"GTD"` |
| `OutsideRth` | bool | 是否允许盘前盘后交易 |
| `Expiry` / `Strike` / `Right` | string | 期权合约要素 |
| `Identifier` | string | 期权/期货标准代码 |
| `DisplaySize` / `MinDisplaySize` | int64 | 冰山单展示数量 |
| `OrderLegs` | `[]OrderLegRequest` | 附加止盈/止损腿 |
| `ContractLegs` / `ComboType` | — | 组合单腿与类型 |
| `OcaOrders` | `[]*OrderRequest` | OCA 互斥子单 |
| `AlgoParams` | `*AlgoParamsRequest` | 算法单参数 |
| `ID` / `OrderId` | int64 | 订单 ID（修改/取消时使用） |

---

## 注意事项 / Notes

- `PlaceOrder()` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `Orders()` 或推送回调确认最终成交状态
- `TotalQuantity` 是 `int64`；按金额下单请用 `MarketOrderByAmount` / `LimitOrderByAmount`
- 期权下单需先通过 `QuoteClient.GetOptionChain` 获取 identifier，或设置 `Expiry`/`Strike`/`Right`
- 查询类方法必须传 `model.XxxRequest` 结构体，不接受 map
- 封装方法返回强类型结构体；只有 `httpClient.ExecuteRaw` 返回 JSON 字符串
