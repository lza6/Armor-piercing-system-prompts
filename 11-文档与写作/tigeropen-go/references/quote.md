# Tiger OpenAPI Go SDK — Market Data / 行情查询

> Go SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/client"
    "github.com/tigerfintech/openapi-go-sdk/config"
    "github.com/tigerfintech/openapi-go-sdk/model"
    "github.com/tigerfintech/openapi-go-sdk/quote"
)

cfg, _ := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
httpClient := client.NewHttpClient(cfg)
qc := quote.NewQuoteClient(httpClient)
```

> **命名约定 / Naming**: 行情方法统一以 `Get` 开头（`GetMarketState`、`GetKline` …）。
> 入参多为 `model.XxxRequest` 结构体，返回**强类型结构体**，无需 `json.Unmarshal`。
> Quote methods are prefixed with `Get`; most take a `model.XxxRequest` struct and return typed structs.

---

## 市场状态 / Market State

```go
// market: "US" / "HK" / "CN" / "SG"
states, err := qc.GetMarketState("US")
for _, s := range states {
    fmt.Println(s.Market, s.MarketStatus, s.Status, s.OpenTime)
}
```

返回 `[]model.MarketState`，字段：`Market`、`MarketStatus`、`Status`、`OpenTime`。

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```go
briefs, err := qc.GetRealTimeQuote(model.BriefRequest{
    Symbols: []string{"AAPL", "TSLA"},
})
for _, b := range briefs {
    fmt.Printf("%s latest=%.2f bid=%.2f ask=%.2f vol=%d change=%.2f rate=%.4f\n",
        b.Symbol, b.LatestPrice, b.BidPrice, b.AskPrice, b.Volume, b.Change, b.ChangeRate)
}
```

返回 `[]model.Brief`。常用字段：`Symbol`、`LatestPrice`、`LatestTime`、`Open`/`High`/`Low`/`Close`、
`PreClose`、`AskPrice`/`AskSize`、`BidPrice`/`BidSize`、`Volume`、`Change`、`ChangeRate`、`Status`。

`GetBrief` 与 `GetRealTimeQuote` 入参、返回一致，可互换 / `GetBrief` is an equivalent alias.

`model.BriefRequest` 可选字段：`IncludeHourTrading *bool`（盘前盘后）、`SecType`、`Lang`。

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```go
// period: "day"/"week"/"month"/"year"/"1min"/"3min"/"5min"/"10min"/"15min"/"30min"/"60min"
klines, err := qc.GetKline(model.KlineRequest{
    Symbols: []string{"AAPL"},
    Period:  "day",
    Limit:   30,
})
for _, k := range klines {
    for _, it := range k.Items {
        fmt.Printf("%s %d O=%.2f H=%.2f L=%.2f C=%.2f V=%d\n",
            k.Symbol, it.Time, it.Open, it.High, it.Low, it.Close, it.Volume)
    }
}
```

返回 `[]model.Kline`，每个元素含 `Symbol`、`Period`、`NextPageToken`、`Items []model.KlineItem`。
`KlineItem` 字段：`Time`、`Open`、`High`、`Low`、`Close`、`Volume`。

`model.KlineRequest` 支持**时间范围**（`BeginTime`/`EndTime`，毫秒）或**分页**（`BeginIndex`/`EndIndex`、`PageToken`），
两者二选一；另有 `Right`（复权）、`TradeSession`、`Date`、`WithFundamental`、`SecType`、`Lang`。

`GetBars` 是 `GetKline` 的等价别名 / `GetBars` is an equivalent alias.

---

## 分时 / Timeline

```go
timelines, err := qc.GetTimeline([]string{"AAPL", "TSLA"})
for _, t := range timelines {
    fmt.Println(t.Symbol, t.Period, t.PreClose)
    if t.Intraday != nil {
        // t.Intraday / t.PreHours / t.AfterHours 为 *model.TimelineBucket
        fmt.Printf("intraday points: %d\n", len(t.Intraday.Items))
    }
}
```

返回 `[]model.Timeline`。注意分时按时段分桶：`Intraday`（盘中）、`PreHours`（盘前）、`AfterHours`（盘后），
均为 `*model.TimelineBucket`，可能为 `nil`，取值前需判空。

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```go
depths, err := qc.GetQuoteDepth(model.DepthQuoteRequest{
    Symbols: []string{"AAPL"},
})
for _, d := range depths {
    for _, a := range d.Asks {
        fmt.Printf("ASK %.2f x%d (orders=%d)\n", a.Price, a.Volume, a.Count)
    }
    for _, b := range d.Bids {
        fmt.Printf("BID %.2f x%d (orders=%d)\n", b.Price, b.Volume, b.Count)
    }
}
```

返回 `[]model.Depth`（`Symbol`、`Asks`、`Bids`）；每档 `model.DepthLevel` 含 `Price`、`Volume`、`Count`。

---

## 逐笔成交 / Trade Ticks

```go
ticks, err := qc.GetTradeTick(model.TradeTickRequest{
    Symbols: []string{"AAPL"},
    Limit:   50,
})
for _, t := range ticks {
    for _, it := range t.Items {
        fmt.Printf("%d price=%.2f vol=%d\n", it.Time, it.Price, it.Volume)
    }
}
```

返回 `[]model.TradeTick`（`Symbol`、`BeginIndex`、`EndIndex`、`Items`）。

---

## 期权到期日 / Option Expirations

```go
// 第一个参数是 symbol 切片；market 为可变参数，可省略
exps, err := qc.GetOptionExpiration([]string{"AAPL"})
```

---

## 期权链 / Option Chain

```go
// items 是 [][2]string，每项为 {symbol, expiry}
chains, err := qc.GetOptionChain([][2]string{{"AAPL", "2026-08-21"}})
```

需要 Greeks 或按条件过滤时，用 `GetOptionChainByReq` / For Greeks or filters use `GetOptionChainByReq`:

```go
returnGreek := true
chains, err := qc.GetOptionChainByReq(model.OptionChainRequest{
    // 注意 OptionQueryItem.Expiry 是 int64 毫秒时间戳
    OptionBasic:      []model.OptionQueryItem{{Symbol: "AAPL", Expiry: 1787356800000}},
    ReturnGreekValue: &returnGreek,
})
```

---

## 期权报价 / Option Brief

```go
briefs, err := qc.GetOptionBrief([]string{"AAPL  260821C00150000"})
// 也可用 GetOptionQuote(identifiers, timezone...)
```

期权代码格式：标的（6 位，右侧空格填充）+ YYMMDD + C/P + 行权价×1000（8 位）。
返回 `[]model.Brief`，期权相关字段：`Strike`、`Right`、`Expiry`（**int64 毫秒时间戳**）、`Multiplier`、`OpenInterest`。

---

## 期货 / Futures

```go
exchanges, err := qc.GetFutureExchange()
contracts, err := qc.GetFutureContracts("CME")
futureQuotes, err := qc.GetFutureRealTimeQuote(model.FutureBriefRequest{
    ContractCodes: []string{"CLmain"},
})
futureKlines, err := qc.GetFutureKline(model.FutureKlineRequest{
    ContractCodes: []string{"CLmain"},
    Period:        "day",
})
```

---

## 资金流 / Capital Flow

```go
flow, err := qc.GetCapitalFlow("AAPL", "US", "day")
dist, err := qc.GetCapitalDistribution("AAPL", "US")
```

---

## 公司行为 / Corporate Actions

```go
changes, err := qc.GetCorporateSymbolChange(model.CorporateActionRequest{
    Market: "US", BeginDate: "2026-01-01", EndDate: "2026-08-01",
})
delistings, err := qc.GetCorporateDelisting(model.CorporateActionRequest{Market: "US"})
ipos, err := qc.GetCorporateIPO(model.CorporateActionRequest{Market: "US"})
```

---

## 直接调用 API / Raw API Call

当以上方法不满足需求时，用 `ExecuteRaw` 直接调用。
第二个参数是 **JSON 字符串**，返回值是 **字符串** / Second arg is a **JSON string**; returns a **string**.

```go
result, err := httpClient.ExecuteRaw("quote_real_time", `{"symbols":["AAPL","TSLA"]}`)
fmt.Println(result)
```

---

## 注意事项 / Notes

- 封装方法返回**强类型结构体**，直接取字段；只有 `ExecuteRaw` 返回 JSON 字符串
- 行情数据需要对应市场的行情权限；期权行情需要单独开通期权行情权限
- `model.Brief.Expiry` 是 `int64` 毫秒时间戳，不是字符串
- 分时数据分 `Intraday`/`PreHours`/`AfterHours` 三个桶，均可能为 `nil`
- 港股期权代码格式与美股不同，例：`TCH.HK 260616C00550000`
