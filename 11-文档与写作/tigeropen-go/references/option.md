
# Tiger OpenAPI Go SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `qc.GetOptionExpiration()` → 获取可选到期日列表
2. **查期权链 Get chain**: `qc.GetOptionChain()` / `qc.GetOptionChainByReq()` → 获取指定到期日的所有合约
3. **查行情 Get quotes**: `qc.GetOptionBrief()` / `qc.GetOptionQuote()` → 获取期权实时行情和 Greeks

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `qc.GetOptionSymbols()` 查询港股期权代码映射

---

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/client"
    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/model"
    "github.com/tigerfintech/openapi-go-sdk/quote"
    "github.com/tigerfintech/openapi-go-sdk/trade"
)

cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
qc := quote.NewQuoteClient(httpClient)
tc := trade.NewTradeClient(httpClient, cfg.Account)
```

> **入参约定 / Request convention**: 方法接收位置参数或 `model.XxxRequest` **结构体**，
> 不能传 `map[string]interface{}`；返回**强类型结构体**。
> Methods take positional args or a typed `model.XxxRequest` struct — not a map.

---

## 期权到期日 / Option Expirations

```go
// 第一个参数是 symbol 切片；market 是可变参数，可省略
expirations, err := qc.GetOptionExpiration([]string{"AAPL"}, "US")
for _, e := range expirations {
    fmt.Println(e.Symbol, e.Dates, e.Timestamps, e.Periods)
}
```

返回 `[]model.OptionExpiration`。字段：`Symbol`、`OptionSymbols`、`Dates`（到期日字符串列表）、
`Timestamps`（毫秒时间戳列表）、`Periods`（周期标签，`m`=月期权 / `w`=周期权 / `q`=季度）、`Counts`。

---

## 期权链 / Option Chain

### 简单查询 / Simple form

```go
// items 是 [][2]string，每项为 {symbol, expiry}
chains, err := qc.GetOptionChain([][2]string{{"AAPL", "2026-08-21"}})
```

### 带 Greeks 与筛选 / With Greeks and filters

```go
returnGreek := true
chainsWithGreeks, err := qc.GetOptionChainByReq(model.OptionChainRequest{
    // 注意 OptionQueryItem.Expiry 是 int64 毫秒时间戳
    OptionBasic:      []model.OptionQueryItem{{Symbol: "AAPL", Expiry: 1787356800000}},
    ReturnGreekValue: &returnGreek,
    Market:           "US",
})
```

筛选条件通过 `OptionFilter *model.OptionChainFilter` 传入（如隐含波动率、Delta、持仓量范围）。

返回 `[]model.OptionChain`，每项含 call/put 合约：`Identifier`、`Strike`、`Right`、
`AskPrice`/`BidPrice`、`LatestPrice`、`Volume`、`OpenInterest`、隐含波动率与
`Delta`/`Gamma`/`Theta`/`Vega`/`Rho`（需 `ReturnGreekValue=true`）。

---

## 港股期权代码映射 / HK Option Symbol Mapping

```go
optSymbols, err := qc.GetOptionSymbols(model.OptionSymbolsRequest{
    Market: "HK",
    Lang:   "en_US",  // en_US / zh_CN / zh_TW
})
// 返回 []model.OptionSymbol：期权 symbol（如 "TCH.HK"）、标的名称、正股代码（如 "00700"）
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

按 identifier 查询 / Query by identifier:

```go
optBriefs, err := qc.GetOptionBrief([]string{"AAPL  260821C00150000"})

// GetOptionQuote 支持可选 timezone 参数
optQuotes, err := qc.GetOptionQuote([]string{"AAPL  260821C00150000"}, "US/Eastern")
```

期权代码格式：标的（6 位，右侧空格填充）+ YYMMDD + C/P + 行权价×1000（8 位）。

返回 `[]model.Brief`，期权相关字段：`Identifier`、`Strike`、`Right`、`Expiry`（**int64 毫秒**）、
`Multiplier`、`OpenInterest`、`BidPrice`/`AskPrice`/`LatestPrice`、`Volume`、
`High`/`Low`/`Open`/`PreClose`、`Change`。

---

## 期权深度行情 / Option Depth Quotes

```go
optDepth, err := qc.GetOptionDepth(model.OptionDepthRequest{
    OptionBasic: []model.OptionQueryItem{
        {Symbol: "AAPL", Right: "PUT", Expiry: 1787356800000, Strike: "210.0"},
    },
    Market: "US",
})
// 返回 []model.Depth：Asks / Bids，每档含 Price、Volume、Count
```

---

## 期权逐笔成交 / Option Trade Ticks

```go
// 仅支持美股期权 / US market only
// 注意字段名是 Contracts（不是 OptionBasic）
optTicks, err := qc.GetOptionTradeTicks(model.OptionTradeTicksRequest{
    Contracts: []model.OptionQueryItem{
        {Symbol: "AAPL", Right: "PUT", Expiry: 1787356800000, Strike: "185.0"},
    },
})
// 返回 []model.TradeTick：Items 中每笔含 Time、Price、Volume、Type
```

---

## 期权K线 / Option K-line

```go
// 位置参数：identifiers, period, beginTime, endTime（毫秒），timezone 可选
optKlines, err := qc.GetOptionKline(
    []string{"AAPL  260821C00150000"},
    "1min",           // day/1min/5min/30min/60min
    1787270400000,    // beginTime 毫秒
    1787356800000,    // endTime 毫秒
)

// 需要 limit / 排序方向时用 WithOpts 变体
optKlines2, err := qc.GetOptionKlineWithOpts(
    []string{"AAPL  260821C00150000"},
    "1min", 1787270400000, 1787356800000, 10, "DESC",
)
```

> `beginTime` / `endTime` 是**必填**位置参数（0.4.6 起）/ They are **required** positional args.

返回 `[]model.Kline`，`Items` 中每根含 `Time`、`Open`、`High`、`Low`、`Close`、`Volume`。

---

## 期权分时数据 / Option Timeline

```go
// 目前仅支持港股期权 / HK market only currently
// 注意字段名是 OptionQuery
optTimeline, err := qc.GetOptionTimeline(model.OptionTimelineRequest{
    OptionQuery: []model.OptionQueryItem{
        {Symbol: "ALB.HK", Right: "CALL", Expiry: 1753878054000, Strike: "117.50"},
    },
    Market: "HK",
})
// 返回 []model.Timeline：PreClose 与按时段分桶的 Intraday / PreHours / AfterHours
```

---

## 期权分析 / Option Analysis

```go
requireVol := true
analysis, err := qc.GetOptionAnalysis(model.OptionAnalysisRequest{
    // Symbols 是 []model.OptionAnalysisSymbol（0.4.7 起，不再是 []string）
    Symbols: []model.OptionAnalysisSymbol{
        {
            Symbol:                "AAPL",
            Period:                "52week",  // 3year/52week/26week/13week
            RequireVolatilityList: &requireVol,
        },
    },
    Market: "US",
})
```

返回 `[]model.OptionAnalysis`，含 30 日隐含波动率、历史波动率、IV/HV 比率、
Call/Put 比率、IV 百分位与排名。

> `RequireVolatilityList` 是 `*bool`，需先声明变量再取地址 / It is a `*bool`.

---

## 单腿期权下单 / Single-leg Option Order

```go
account := cfg.Account

// 买入看涨期权 / Buy call option（1 张 = 100 股）
optOrder := model.LimitOrder(account, "AAPL", "OPT", "BUY", 1, 5.0)
optOrder.Expiry = "20260821"  // YYYYMMDD
optOrder.Strike = "150.0"
optOrder.Right = "CALL"
optOrder.Currency = "USD"

preview, err := tc.PreviewOrder(optOrder)
placed, err := tc.PlaceOrder(optOrder)
```

也可直接用 identifier / Or set the standard identifier:

```go
byIdentifier := model.LimitOrder(account, "AAPL", "OPT", "BUY", 1, 5.0)
byIdentifier.Identifier = "AAPL  260821C00150000"
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

组合单用 `model.ComboOrder` 构造后交给 `PlaceOrder`，**没有** `PlaceComboOrder` 方法。
Build with `model.ComboOrder` and submit via `PlaceOrder`; there is no `PlaceComboOrder`.

```go
// 牛市看涨价差 / Bull Call Spread (VERTICAL)
spreadLegs := []model.ContractLegRequest{
    model.NewContractLeg("AAPL", "OPT", "BUY", 1, "2026-08-21", "145.0", "CALL"),
    model.NewContractLeg("AAPL", "OPT", "SELL", 1, "2026-08-21", "155.0", "CALL"),
}
// ComboOrder(account, action, orderType, quantity, legs, comboType, limitPrice, auxPrice, trailingPercent)
spread := model.ComboOrder(account, "BUY", "LMT", 1, spreadLegs, "VERTICAL", 3.0, 0, 0)
spreadResult, err := tc.PlaceOrder(spread)
```

### 其他策略示例 / Other Strategy Examples

```go
// 跨式策略 / Straddle：同行权价 Call+Put
straddleLegs := []model.ContractLegRequest{
    model.NewContractLeg("AAPL", "OPT", "BUY", 1, "2026-08-21", "150.0", "CALL"),
    model.NewContractLeg("AAPL", "OPT", "BUY", 1, "2026-08-21", "150.0", "PUT"),
}
straddle := model.ComboOrder(account, "BUY", "LMT", 1, straddleLegs, "STRADDLE", 8.0, 0, 0)

// 备兑策略 / Covered Call：正股 + 卖出 Call
coveredLegs := []model.ContractLegRequest{
    model.NewContractLeg("AAPL", "STK", "BUY", 100, "", "", ""),
    model.NewContractLeg("AAPL", "OPT", "SELL", 1, "2026-08-21", "160.0", "CALL"),
}
covered := model.ComboOrder(account, "BUY", "LMT", 1, coveredLegs, "COVERED", 0, 0, 0)
```

### 组合策略类型总览 / Combo Strategy Types

| ComboType | 策略 Strategy | 说明 Description |
|-----------|--------------|-----------------|
| `VERTICAL` | 垂直价差 | 同到期日不同行权价 Same expiry, different strikes |
| `STRADDLE` | 跨式 | 同行权价同到期日 Call+Put, same strike & expiry |
| `STRANGLE` | 宽跨式 | 不同行权价同到期日 Different strikes, same expiry |
| `CALENDAR` | 日历价差 | 同行权价不同到期日 Same strike, different expiries |
| `DIAGONAL` | 对角线价差 | 不同行权价不同到期日 Different strikes & expiries |
| `COVERED` | 备兑 | 持有股票+卖Call Long stock + short call |
| `PROTECTIVE` | 保护性 | 持有股票+买Put Long stock + long put |
| `SYNTHETIC` | 合成 | 合成多/空头 Synthetic long/short |
| `CUSTOM` | 自定义 | 4条腿组合（Iron Condor等） |

---

## 期权行权 / Option Exercise

```go
exercisable, err := tc.OptionExercisePositions(model.OptionExercisePositionRequest{
    Account: account,
})
checked, err := tc.OptionExerciseCheck(model.OptionExerciseCheckRequest{Account: account})
submitted, err := tc.OptionExerciseSubmit(model.OptionExerciseSubmitRequest{Account: account})
exRecords, err := tc.OptionExerciseRecords(model.OptionExercisePageRequest{Account: account})
exCancelled, err := tc.OptionExerciseCancel(model.OptionExerciseCancelRequest{Account: account})
```

`OptionExerciseSubmit` / `OptionExerciseCancel` 返回 `(bool, error)`。

---

## 查询期权持仓 / Query Option Positions

```go
optPositions, err := tc.Positions(model.PositionsRequest{
    Account: account,
    SecType: "OPT",
})
for _, p := range optPositions {
    fmt.Printf("%s identifier=%s multiplier=%.0f qty=%.2f mktValue=%.2f\n",
        p.Symbol, p.Identifier, p.Multiplier, p.PositionQty, p.MarketValue)
}
```

`model.Position` 通过 `Identifier`（期权标准代码）和 `Multiplier` 表达期权合约，
**没有** 独立的 `Strike`/`Expiry`/`Right` 字段——需要这些要素时请解析 `Identifier`，
或用 `PositionsRequest` 的 `Expiry`/`Strike`/`Right` 作为**查询过滤条件**。
`model.Position` carries option contracts via `Identifier` + `Multiplier`; there are no
separate `Strike`/`Expiry`/`Right` fields on the response.

---

## 注意事项 / Notes

- 方法接收位置参数或 `model.XxxRequest` 结构体，不接受 map；返回强类型结构体
- **SDK 没有 `OptionIndicator` 方法**；单合约 Greeks 通过 `GetOptionChainByReq(ReturnGreekValue=true)`
  或 `GetOptionBrief` 获取 / There is no `OptionIndicator` method
- **SDK 没有 `PlaceComboOrder` 方法**；组合单用 `model.ComboOrder` + `PlaceOrder`
- `model.OptionQueryItem.Expiry` 是 `int64` 毫秒时间戳，不是日期字符串
- `GetOptionKline` 的 `beginTime`/`endTime` 是必填位置参数
- `OptionAnalysisRequest.Symbols` 是 `[]model.OptionAnalysisSymbol`，不是 `[]string`
- 港股期权需先用 `GetOptionSymbols()` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表 100 股标的 / Each contract = 100 shares
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 机构用户在请求结构体上设置 `SecretKey`，或调用 `tc.SetSecretKey(key)`
