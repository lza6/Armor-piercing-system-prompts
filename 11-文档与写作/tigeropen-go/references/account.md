
# Tiger OpenAPI Go SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/client"
    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/model"
    "github.com/tigerfintech/openapi-go-sdk/trade"
)

cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

> **入参约定 / Request convention**: Go 是静态类型语言，所有方法接收 `model.XxxRequest` **结构体**，
> 不能传 `map[string]interface{}`；返回**强类型结构体**，不需要 `json.Unmarshal`。
> All methods take a typed `model.XxxRequest` struct (not a map) and return typed structs.

---

## 账户列表 / Account List

```go
accounts, err := tc.ManagedAccounts(model.ManagedAccountsRequest{})
// 不传 Account 返回当前用户可见的所有账号（综合、环球、模拟）
// Omit Account to return every account visible to the user

for _, a := range accounts {
    fmt.Printf("%s status=%s type=%s capability=%s\n",
        a.Account, a.Status, a.AccountType, a.Capability)
}
```

字段说明 / Fields:
- `Account` — 账户号（综合 5~10 位数字，模拟 17 位，环球以 U 开头）
- `Capability` — `CASH`（现金）/ `RegTMargin`（保证金）/ `PMGRN`（组合保证金）
- `Status` — `Funded` / `Open` / `Pending` / `Rejected` / `Closed`
- `AccountType` — `STANDARD` / `GLOBAL` / `PAPER`

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```go
globalAssets, err := tc.Assets(model.AssetsRequest{
    Account:     "DU000001",
    Segment:     true,  // 按证券/期货分类
    MarketValue: true,  // 按市场分市值（仅环球账户）
})

for _, a := range globalAssets {
    fmt.Printf("%s %s netLiq=%.2f cash=%.2f buyingPower=%.2f unrealPnl=%.2f\n",
        a.Account, a.Currency, a.NetLiquidation, a.CashValue,
        a.BuyingPower, a.UnrealizedPnL)
    for _, seg := range a.Segments {
        fmt.Printf("  segment %s netLiq=%.2f\n", seg.Category, seg.NetLiquidation)
    }
}
```

返回 `[]model.Asset`。主要字段：`NetLiquidation`（净清算值）、`CashValue`（现金）、
`BuyingPower`（购买力）、`RealizedPnL`、`UnrealizedPnL`、`Segments`（按品种分类）。

### 综合/模拟账号 / Standard/Paper Account

```go
consolidated := true
prime, err := tc.PrimeAssets(model.AssetsRequest{
    Account:      "123456",
    BaseCurrency: "USD",
    Consolidated: &consolidated,  // 指针类型：SEC+FUND 聚合显示
})

fmt.Println("account:", prime.AccountID, "updated:", prime.UpdateTimestamp)
for _, seg := range prime.Segments {
    fmt.Printf("%s %s cashBalance=%.2f netLiq=%.2f initMargin=%.2f maintainMargin=%.2f\n",
        seg.Category, seg.Currency, seg.CashBalance, seg.NetLiquidation,
        seg.InitMargin, seg.MaintainMargin)
}
```

返回 `*model.PrimeAsset`（`AccountID`、`UpdateTimestamp`、`Segments`）。
`Segments []model.PrimeAssetSegment` 主要字段：
- `Category` — `S`（证券）/ `C`（期货）/ `F`（基金）/ `D`（数字货币）
- `Capability` — `RegTMargin` / `Cash`
- `CashBalance`、`CashAvailableForTrade`、`GrossPositionValue`、`EquityWithLoan`
- `NetLiquidation`、`InitMargin`、`MaintainMargin`、`OvernightMargin`

> `Consolidated` 是 `*bool`，需先声明变量再取地址 / `Consolidated` is a `*bool`.

---

## 账户持仓 / Account Positions

```go
positions, err := tc.Positions(model.PositionsRequest{
    Account:  "123456",
    SecType:  "STK",  // STK/OPT/FUT，默认 STK
    Currency: "ALL",  // ALL/USD/HKD/CNH
    Market:   "ALL",  // ALL/US/HK/CN
})

for _, p := range positions {
    fmt.Printf("%s qty=%.2f salable=%.2f cost=%.4f mktValue=%.2f unrealPnl=%.2f\n",
        p.Symbol, p.PositionQty, p.SalableQty, p.AverageCost,
        p.MarketValue, p.UnrealizedPnl)
}
```

返回 `[]model.Position`。主要字段：`Symbol`、`SecType`、`Market`、`Currency`、
`PositionQty`（持仓数量，支持碎股）、`SalableQty`（可卖数量）、`AverageCost`（平均成本）、
`MarketValue`、`RealizedPnl`、`UnrealizedPnl`、`UnrealizedPnlPercent`。

### 期权持仓 / Option Positions

```go
optPositions, err := tc.Positions(model.PositionsRequest{
    Account: "123456",
    SecType: "OPT",
})
```

期权持仓通过 `Identifier`（期权标准代码）和 `Multiplier` 表达合约要素，响应结构体上
**没有** 独立的 `Strike`/`Expiry`/`Right` 字段；这三项可作为 `PositionsRequest` 的查询过滤条件。
Option positions expose the contract via `Identifier` + `Multiplier`; `Strike`/`Expiry`/`Right`
exist only as request-side filters.

---

## 历史资产分析 / Asset Analytics (PnL History)

```go
analytics, err := tc.AnalyticsAsset(model.AnalyticsAssetRequest{
    Account:   "123456",
    StartDate: "2026-01-01",  // yyyy-MM-dd
    EndDate:   "2026-01-31",
    SegType:   "SEC",         // SEC / FUT
    Currency:  "USD",
})
```

> 方法名是 `AnalyticsAsset`，**不是** `PrimeAnalyticsAsset` / The method is `AnalyticsAsset`.

返回 `[]model.AnalyticsAsset`，是**扁平的按日列表，没有汇总对象**。
Returns a flat `[]model.AnalyticsAsset` — one entry per day, no summary object.

| 字段 Field | 类型 | 说明 |
|-----------|------|------|
| `Date` | string | 日期 |
| `HoldingValue` | float64 | 持仓价值 |
| `CashBalance` | float64 | 现金余额 |
| `Pnl` | float64 | 盈亏 |
| `PnlRate` | float64 | 收益率 |
| `NetValueIndex` | float64 | 净值指数 |
| `Currency` | string | 币种 |
| `SegType` | string | 分类（SEC / FUT）|

---

## 最大可交易数量 / Estimate Tradable Quantity

```go
qty, err := tc.EstimateTradableQuantity(model.EstimateTradableQuantityRequest{
    Account:    "123456",
    Symbol:     "AAPL",
    SecType:    "STK",
    Action:     "BUY",
    OrderType:  "LMT",
    LimitPrice: 150.0,
})
```

返回 `*model.EstimateTradableQuantity`，含现金可买/卖数量、融资融券可买/卖数量、
持仓数量与持仓可交易数量。期权可额外传 `Expiry`/`Strike`/`Right`。

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```go
avail, err := tc.SegmentFundAvailable(model.SegmentFundRequest{
    Account:     "123456",
    FromSegment: "SEC",  // SEC / FUT
    Currency:    "USD",
})
```

### 发起转账 / Transfer

```go
transferred, err := tc.TransferSegmentFund(model.SegmentFundRequest{
    Account:     "123456",
    FromSegment: "SEC",
    ToSegment:   "FUT",
    Currency:    "USD",
    Amount:      1000.0,
})
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

> 方法名是 `TransferSegmentFund`（动词在前），**不是** `SegmentFundTransfer`。
> The method is `TransferSegmentFund`.

### 撤销转账 / Cancel Transfer

```go
cancelled, err := tc.CancelSegmentFund(model.SegmentFundRequest{
    Account: "123456",
    ID:      "transfer_id",
})
```

### 转账历史 / Transfer History

```go
history, err := tc.SegmentFundHistory(model.SegmentFundRequest{
    Account: "123456",
    Limit:   20,
})
```

---

## 出入金记录 / Funding Records

SDK 没有 `DepositWithdraw` 方法，用以下两个接口 / There is no `DepositWithdraw`; use:

```go
// 出入金流水 / Funding history
funding, err := tc.FundingHistory(model.FundingHistoryRequest{
    Account: "123456",
    SegType: "SEC",
})

// 资金明细 / Fund details
details, err := tc.FundDetails(model.FundDetailsRequest{
    Account:   "123456",
    SegTypes:  []string{"SEC"},
    Currency:  "USD",
    StartDate: 1767225600000,  // 毫秒时间戳
    EndDate:   1769904000000,
    Limit:     50,
})
```

---

## 聚合资产 / Aggregate Assets

```go
agg, err := tc.AggregateAssets(model.AggregateAssetsRequest{
    Account: "123456",
})
```

---

## 直接调用 API / Raw API Call

未封装的接口用 `ExecuteRaw`，第二个参数是 **JSON 字符串** / Second arg is a **JSON string**:

```go
raw, err := httpClient.ExecuteRaw("accounts", `{}`)
fmt.Println(raw)
```

---

## 注意事项 / Notes

- 环球账户(Global)用 `Assets()`，综合/模拟账户(Standard/Paper)用 `PrimeAssets()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 所有方法接收 `model.XxxRequest` 结构体，返回强类型结构体；只有 `ExecuteRaw` 返回 JSON 字符串
- 持仓使用 `PositionQty` 字段，旧字段 `Position`+`PositionScale` 已废弃
- `MaintainMargin` 低于 0 时会触发强制平仓
- 机构用户在请求结构体上设置 `SecretKey` 字段，或调用 `tc.SetSecretKey(key)` 全局设置
