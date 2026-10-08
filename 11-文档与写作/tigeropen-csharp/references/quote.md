# Tiger OpenAPI C# SDK — Market Data / 行情查询

> C# SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Common.Enum;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/",
    Language = Language.en_US,
};
QuoteClient quoteClient = new QuoteClient(config);
```

所有行情 API 均通过 `TigerRequest<TResponse>` 模式调用（下方为模式示意，非可编译代码）：

```text
var request = new TigerRequest<TResponse>()
{
    ApiMethodName = QuoteApiService.XXX,  // API 名称常量
    ModelValue = new XxxModel() { ... }   // 请求参数
};
var response = await quoteClient.ExecuteAsync(request);
```

---

## 市场状态 / Market State

```csharp
using TigerOpenAPI.Quote.Model;

var request = new TigerRequest<MarketStateResponse>()
{
    ApiMethodName = QuoteApiService.MARKET_STATE,
    ModelValue = new QuoteMarketModel() { Market = Market.US }
    // Market 枚举: Market.US / Market.HK / Market.CN / Market.SG
};
var response = await quoteClient.ExecuteAsync(request);
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```csharp
var request = new TigerRequest<QuoteRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,
    ModelValue = new QuoteSymbolModel()
    {
        Symbols = new List<string> { "AAPL", "TSLA", "00700" }
    }
};
var response = await quoteClient.ExecuteAsync(request);

if (response?.Data != null)
{
    foreach (var q in response.Data)
    {
        Console.WriteLine($"{q.Symbol}: latest={q.LatestPrice}, " +
                          $"preClose={q.PreClose}, volume={q.Volume}");
    }
}
```

`BRIEF`（简版报价，含盘前盘后）：

```csharp
var request = new TigerRequest<QuoteRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.BRIEF,
    ModelValue = new QuoteSymbolModel()
    {
        Symbols = new List<string> { "AAPL" },
        IncludeHourTrading = true   // 包含盘前盘后
    }
};
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```csharp
var request = new TigerRequest<QuoteKlineResponse>()
{
    ApiMethodName = QuoteApiService.KLINE,
    ModelValue = new QuoteKlineModel()
    {
        Symbols = new List<string> { "AAPL" },
        Period = "day",    // day/week/month/year/1min/3min/5min/15min/30min/60min
        BeginTime = -1,    // -1 = 最新
        EndTime = -1,
        Rigth = RightOption.br,  // 注意 SDK 源码拼写是 Rigth（非 Right）
        Limit = 251
    }
};
var response = await quoteClient.ExecuteAsync(request);

if (response?.Data != null)
{
    foreach (var k in response.Data)          // KlineItem（每个标的一组）
    {
        foreach (var bar in k.Items)          // KlinePoint
        {
            Console.WriteLine($"{k.Symbol} time={bar.Time}, open={bar.Open}, " +
                              $"high={bar.High}, low={bar.Low}, close={bar.Close}, vol={bar.Volume}");
        }
    }
}
```

---

## 分时 / Timeline

```csharp
// 当日分时 / Current day timeline
var request = new TigerRequest<QuoteTimelineResponse>()
{
    ApiMethodName = QuoteApiService.TIMELINE,
    ModelValue = new QuoteTimelineModel()
    {
        Symbols = new List<string> { "AAPL" },
        IncludeHourTrading = false
    }
};

// 历史分时 / Historical timeline
var histRequest = new TigerRequest<QuoteHistoryTimelineResponse>()
{
    ApiMethodName = QuoteApiService.HISTORY_TIMELINE,
    ModelValue = new QuoteHistoryTimelineModel()
    {
        Symbols = new List<string> { "AAPL" },
        Date = "2024-01-15"  // yyyy-MM-dd
    }
};
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```csharp
var request = new TigerRequest<QuoteDepthResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_DEPTH,
    ModelValue = new QuoteDepthModel()
    {
        Symbols = new List<string> { "AAPL" },
        Market = Market.US   // "US" / "HK"
    }
};
var response = await quoteClient.ExecuteAsync(request);
// 返回 asks/bids，各含 price, volume 的数组
```

---

## 逐笔成交 / Trade Ticks

```csharp
var request = new TigerRequest<QuoteTradeTickResponse>()
{
    ApiMethodName = QuoteApiService.TRADE_TICK,
    ModelValue = new QuoteTradeTickModel()
    {
        Symbols = new List<string> { "AAPL" },
        BeginIndex = -1,   // -1 = 最新
        EndIndex = -1,
        Limit = 100
    }
};
```

---

## 期权到期日 / Option Expirations

```csharp
var request = new TigerRequest<OptionExpirationResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_EXPIRATION,
    ModelValue = new OptionExpirationModel()
    {
        Symbols = new List<string> { "AAPL" }
    }
};
var response = await quoteClient.ExecuteAsync(request);
// 返回 dates（字符串日期列表）, timestamps, periodTag ("m"=月度/"w"=周度)
```

---

## 期权链 / Option Chain

```csharp
var chainRequest = new TigerRequest<OptionChainResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_CHAIN,
    ModelValue = new OptionChainV3Model()
    {
        Market = Market.US,
        ReturnGreekValue = true,
        OptionBasic = new List<OptionChainModel>()
        {
            new OptionChainModel()
            {
                Symbol = "AAPL",
                // Expiry 是 long 毫秒时间戳
                Expiry = DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE)
            }
        }
    }
};
var chainResponse = await quoteClient.ExecuteAsync(chainRequest);
// 每个合约含: identifier, strike, right(CALL/PUT), bid, ask, volume,
//            openInterest, impliedVol, delta, gamma 等
```

筛选条件用 `OptionChainFilterModel` + `Range<Double>`，详见 [option.md](option.md)。

---

## 期权报价 / Option Brief

```csharp
// 按合约要素查询；Expiry 是 long 毫秒时间戳
var optBriefRequest = new TigerRequest<OptionBriefResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_BRIEF,
    ModelValue = new OptionBasicModel()
    {
        Market = Market.US,
        OptionBasic = new List<OptionCommonModel>()
        {
            new OptionCommonModel()
            {
                Symbol = "AAPL",
                Right = "CALL",
                Strike = "150.0",
                Expiry = DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE)
            }
        }
    }
};
var optBriefResponse = await quoteClient.ExecuteAsync(optBriefRequest);
```

期权完整能力（到期日/链/深度/K线/分析/行权）见 [option.md](option.md)。

---

## 期货 / Futures

```csharp
// 期货实时行情 / Future real-time quotes
var request = new TigerRequest<FutureRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.FUTURE_REAL_TIME_QUOTE,
    ModelValue = new FutureContractCodesModel()
    {
        ContractCodes = new List<string> { "CL2509" }
    }
};

// 期货 K 线 / Future kline
var klineRequest = new TigerRequest<FutureKlineResponse>()
{
    ApiMethodName = QuoteApiService.FUTURE_KLINE,
    ModelValue = new FutureKlineModel()
    {
        ContractCodes = new List<string> { "CL2509" },
        Period = "day",
        Limit = 100
    }
};
```

---

## 资金流向 / Capital Flow

```csharp
var request = new TigerRequest<QuoteCapitalFlowResponse>()
{
    ApiMethodName = QuoteApiService.CAPITAL_FLOW,
    ModelValue = new QuoteCapitalFlowModel()
    {
        Symbol = "AAPL",
        Market = Market.US,
        Period = "day"  // day/week/month/intraday
    }
};

// 资金分布 / Capital distribution
var distRequest = new TigerRequest<QuoteCapitalDistributionResponse>()
{
    ApiMethodName = QuoteApiService.CAPITAL_DISTRIBUTION,
    ModelValue = new QuoteCapitalModel()
    {
        Symbol = "AAPL",
        Market = Market.US
    }
};
```

---

## QuoteApiService 常量速查 / Constants

| 常量 Constant | API 说明 |
|--------------|---------|
| `MARKET_STATE` | 市场状态 |
| `BRIEF` | 实时报价（含盘前后） |
| `QUOTE_REAL_TIME` | 实时报价 |
| `KLINE` | K 线 |
| `TIMELINE` | 当日分时 |
| `HISTORY_TIMELINE` | 历史分时 |
| `TRADE_TICK` | 逐笔成交 |
| `QUOTE_DEPTH` | 深度行情 |
| `OPTION_EXPIRATION` | 期权到期日 |
| `OPTION_CHAIN` | 期权链 |
| `OPTION_BRIEF` | 期权报价 |
| `FUTURE_REAL_TIME_QUOTE` | 期货实时行情 |
| `FUTURE_KLINE` | 期货 K 线 |
| `CAPITAL_FLOW` | 资金流向 |
| `CAPITAL_DISTRIBUTION` | 资金分布 |

---

## 注意事项 / Notes

- 所有方法支持同步 `Execute()` 和异步 `ExecuteAsync()`
- `ConfigFilePath` 是目录路径（以 `/` 结尾），不是文件全路径
- 行情数据需要对应市场的行情权限
- 港股期权代码格式与美股不同：`TCH.HK 20241230 410.00 CALL`
- 期权 identifier 格式（美股）：`"AAPL  250829C00150000"`（注意两个空格）
