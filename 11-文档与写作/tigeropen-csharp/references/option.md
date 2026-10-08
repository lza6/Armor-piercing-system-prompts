
# Tiger OpenAPI C# SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

1. **查到期日 Get expirations**: `QuoteApiService.OPTION_EXPIRATION` + `OptionExpirationModel`
2. **查期权链 Get chain**: `QuoteApiService.OPTION_CHAIN` + `OptionChainV3Model`
3. **查行情 Get quotes**: `QuoteApiService.OPTION_BRIEF` + `OptionBasicModel`

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股：`00700` → `TCH`（腾讯）
- 用 `QuoteApiService.ALL_HK_OPTION_SYMBOLS` 查询港股期权代码映射

---

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Common.Enum;
using TigerOpenAPI.Common.Util;
using TigerOpenAPI.Config;
using TigerOpenAPI.Model;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;
using TigerOpenAPI.Trade;
using TigerOpenAPI.Trade.Model;
using TigerOpenAPI.Trade.Response;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/"
};
QuoteClient quoteClient = new QuoteClient(config);
TradeClient tradeClient = new TradeClient(config);
```

> **关键约定 / Key conventions**
> - `Market` 是 `Market` 枚举（`Market.US`），不是字符串
> - **`Expiry` 是 `long` 毫秒时间戳**，用 `DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE)` 转换
> - `Strike` / `Right` 是 `string`
> - 筛选条件用 `OptionChainFilterModel` + `Range<Double>`，不是扁平的 `XxxMin`/`XxxMax` 字段
> - `Expiry` is a `long` ms timestamp; filters go through `OptionChainFilterModel`.

---

## 期权到期日 / Option Expirations

```csharp
var expirationRequest = new TigerRequest<OptionExpirationResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_EXPIRATION,
    ModelValue = new OptionExpirationModel()
    {
        Symbols = new List<string> { "AAPL" },
        Market = Market.US        // 枚举
    }
};
OptionExpirationResponse expirations = await quoteClient.ExecuteAsync(expirationRequest);
// Data 是 List<OptionExpirationItem>
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
        },
        OptionFilter = new OptionChainFilterModel()
        {
            InTheMoney = true,
            ImpliedVolatility = new Range<Double>() { Min = 0.15, Max = 0.8 },
            Greeks = new Greeks()
            {
                Delta = new Range<Double>() { Min = 0.2, Max = 0.8 }
            }
        }
    }
};
OptionChainResponse chain = await quoteClient.ExecuteAsync(chainRequest);
// Data 是 List<OptionChainItem>
```

> 模型是 `OptionChainV3Model`；`Market` / `ReturnGreekValue` / `OptionFilter` 在
> **外层模型**上，`OptionChainModel` 只有 `Symbol` 与 `Expiry` 两个字段。
> 筛选用 `OptionChainFilterModel` + `Range<Double>` / `Greeks`，**没有**
> `ImpliedVolatilityMin` / `DeltaMax` / `OpenInterestMin` 这类扁平字段。

---

## 港股期权代码映射 / HK Option Symbol Mapping

```csharp
var symbolRequest = new TigerRequest<OptionSymbolResponse>()
{
    ApiMethodName = QuoteApiService.ALL_HK_OPTION_SYMBOLS,
    ModelValue = new OptionModel()
    {
        Market = Market.HK,
        Lang = Language.en_US
    }
};
OptionSymbolResponse optSymbols = await quoteClient.ExecuteAsync(symbolRequest);
// Data 是 List<OptionSymbolItem>
```

> 常量是 `ALL_HK_OPTION_SYMBOLS`（**不是** `OPTION_SYMBOL`），
> 模型是基类 `OptionModel`（**没有** `OptionSymbolModel`）。

---

## 期权实时行情 / Option Brief

```csharp
var briefRequest = new TigerRequest<OptionBriefResponse>()
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
OptionBriefResponse optBriefs = await quoteClient.ExecuteAsync(briefRequest);
// Data 是 List<OptionBriefItem>
```

> 模型是 `OptionBasicModel`（**没有** `OptionBriefModel`）。
> 传入非 `OptionBasicModel` 的模型时，SDK 会自动把 `ApiVersion` 降到 `"1.0"`。

---

## 期权深度行情 / Option Depth

```csharp
var depthRequest = new TigerRequest<OptionDepthResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_DEPTH,
    ModelValue = new OptionBasicModel()
    {
        Market = Market.US,
        OptionBasic = new List<OptionCommonModel>()
        {
            new OptionCommonModel()
            {
                Symbol = "AAPL",
                Right = "PUT",
                Strike = "210.0",
                Expiry = DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE)
            }
        }
    }
};
OptionDepthResponse optDepth = await quoteClient.ExecuteAsync(depthRequest);
```

> 深度行情与 Brief 共用 `OptionBasicModel`（**没有** `OptionDepthModel`）。

---

## 期权逐笔成交 / Option Trade Ticks

```csharp
// 逐笔用批量模型 BatchApiModel<OptionCommonModel>
var tickRequest = new TigerRequest<OptionTradeTickResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_TRADE_TICK,
    ModelValue = new BatchApiModel<OptionCommonModel>()
    {
        Items = new List<OptionCommonModel>()
        {
            new OptionCommonModel()
            {
                Symbol = "AAPL",
                Right = "PUT",
                Strike = "185.0",
                Expiry = DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE)
            }
        }
    }
};
OptionTradeTickResponse optTicks = await quoteClient.ExecuteAsync(tickRequest);
```

> 逐笔成交用 `BatchApiModel<OptionCommonModel>`（字段名 `Items`），
> **没有** `OptionTradeTickModel`。

---

## 期权 K 线 / Option K-line

```csharp
var klineRequest = new TigerRequest<OptionKlineResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_KLINE,
    ModelValue = new OptionKlineV2Model()
    {
        Market = Market.US,
        OptionQuery = new List<OptionKlineModel>()
        {
            new OptionKlineModel()
            {
                Symbol = "AAPL",
                Right = "CALL",
                Strike = "170.0",
                Expiry = DateUtil.ConvertTimestamp("2026-08-21", CustomTimeZone.NY_ZONE),
                BeginTime = DateUtil.ConvertTimestamp("2026-06-01", CustomTimeZone.NY_ZONE),
                EndTime = DateUtil.ConvertTimestamp("2026-06-15", CustomTimeZone.NY_ZONE),
                Period = OptionKType.day.Value,
                Limit = 10,
                SortDir = SortDir.SortDir_Descend
            }
        }
    }
};
OptionKlineResponse optKlines = await quoteClient.ExecuteAsync(klineRequest);
```

> 模型是 `OptionKlineV2Model`，合约列表字段名是 `OptionQuery`，
> 元素类型是 `OptionKlineModel`（继承 `OptionCommonModel`，所以有
> `Symbol`/`Right`/`Strike`/`Expiry`）。
> `BeginTime`/`EndTime` 是 `Int64` 毫秒；`SortDir` 是 `SortDir` 枚举
> （`SortDir_No` / `SortDir_Ascend` / `SortDir_Descend`）。
> 传入非 `OptionKlineV2Model` 的模型时，SDK 会把 `ApiVersion` 降到 `"1.0"`。

---

## 期权分析 / Option Analysis

```csharp
var analysisRequest = new TigerRequest<OptionAnalysisResponse>()
{
    ApiMethodName = QuoteApiService.OPTION_ANALYSIS,
    ModelValue = new OptionAnalysisModel()
    {
        Market = Market.US,
        // Symbols 是 List<OptionAnalysisSymbolModel>，不是 List<string>
        Symbols = new List<OptionAnalysisSymbolModel>()
        {
            new OptionAnalysisSymbolModel()
            {
                Symbol = "AAPL",
                Period = "52week",              // 3year/52week/26week/13week
                RequireVolatilityList = true
            }
        }
    }
};
OptionAnalysisResponse analysis = await quoteClient.ExecuteAsync(analysisRequest);
```

> `OptionAnalysisModel` 只有 `Symbols` 与继承来的 `Market`；`Period` 与
> `RequireVolatilityList` 在**每个** `OptionAnalysisSymbolModel` 上。
> 该接口在 SDK 示例中未被调用过，字段以源码为准。

---

## 期权分时 / Option Timeline

`QuoteApiService.OPTION_TIMELINE` 常量存在，但 SDK **尚未提供**对应的请求模型与响应类型。
需要时用通用响应 `TigerListResponse` + 自行构造模型，或直接走 HTTP。
The `OPTION_TIMELINE` constant exists but the SDK ships no model/response for it.

---

## 单腿期权下单 / Single-leg Option Order

```csharp
// 用 ContractItem 工厂方法构造期权合约
ContractItem optContract = ContractItem.BuildOptionContract("AAPL", "20260821", 150.0, "CALL");
// 也可用 identifier: ContractItem.BuildOptionContract("AAPL  260821C00150000")

PlaceOrderModel optOrder = PlaceOrderModel.BuildLimitOrder(
    config.DefaultAccount, optContract, ActionType.BUY, 1L, 5.0);

var placeRequest = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PLACE_ORDER,
    ModelValue = optOrder
};
PlaceOrderResponse placed = await tradeClient.ExecuteAsync(placeRequest);
```

> 合约类型是 `ContractItem`（**没有** `Contract` 类），
> 用 `ContractItem.BuildOptionContract(...)` 构造。

---

## 多腿组合策略 / Multi-leg Combo Strategies

组合单用 `PlaceOrderModel.BuildMultiLegOrder` + `TradeApiService.PLACE_ORDER`。
**没有** `PLACE_COMBO_ORDER` 常量。
Combo orders use `BuildMultiLegOrder` + `PLACE_ORDER`; there is no `PLACE_COMBO_ORDER`.

```csharp
var legs = new List<ContractLeg>()
{
    new ContractLeg() { Symbol = "AAPL", SecType = "OPT", Action = "BUY", Ratio = 1,
                        Expiry = "20260821", Strike = "145.0", Right = "CALL" },
    new ContractLeg() { Symbol = "AAPL", SecType = "OPT", Action = "SELL", Ratio = 1,
                        Expiry = "20260821", Strike = "155.0", Right = "CALL" }
};

// BuildMultiLegOrder(account, legs, comboType, action, quantity, orderType,
//                    limitPrice, auxPrice, trailingPercent, totalQuantityScale = 0)
PlaceOrderModel comboOrder = PlaceOrderModel.BuildMultiLegOrder(
    config.DefaultAccount, legs, ComboType.VERTICAL, ActionType.BUY, 1,
    OrderType.LMT, 3.0, null, null);

var comboRequest = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PLACE_ORDER,
    ModelValue = comboOrder
};
PlaceOrderResponse comboPlaced = await tradeClient.ExecuteAsync(comboRequest);
```

### 组合策略类型 / Combo Strategy Types

`ComboType` 枚举值对应下表策略 / `ComboType` enum values:

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

这是 SDK 中**唯一**封装成便捷方法的一组接口（内部自行构造请求并 `ExecuteAsync`）。
These are the only convenience wrappers in the SDK.

```csharp
// type: "Exercise"（提前行权）| "Expire"（提前放弃行权）
var exCheck = await tradeClient.CheckOptionExerciseAsync(
    contractId: 123456789L, type: "Exercise", quantity: 1);

var exPositions = await tradeClient.GetOptionExercisePositionsAsync(type: "Exercise");

var exSubmit = await tradeClient.SubmitOptionExerciseAsync(
    contractId: 123456789L, type: "Exercise", quantity: 1,
    executingDate: "2026-08-21", isForce: false);

var exRecords = await tradeClient.GetOptionExerciseRecordsAsync(page: 1, size: 20);

var exCancel = await tradeClient.CancelOptionExerciseAsync(exerciseId: 987654321L);
```

- `Exercise` 时 `executingDate`（yyyy-MM-dd）与 `isForce` 必填
- `itmRate`（0-10）为 `Expire` 专用
- 每个方法都有可选的 `account` 参数，省略时用 `TigerConfig.DefaultAccount`

---

## 查询期权持仓 / Query Option Positions

```csharp
var optPositionsRequest = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionsModel()
    {
        SecType = SecType.OPT,
        Right = "CALL",
        Strike = 150.0
    }
};
PositionsResponse optPositions = await tradeClient.ExecuteAsync(optPositionsRequest);
// 列表在 Data.Items
```

---

## 注意事项 / Notes

- **`Expiry` 是 `long` 毫秒时间戳**，用 `DateUtil.ConvertTimestamp(date, timeZone)` 转换；
  `Strike` / `Right` 是 `string`
- `Market` / `SecType` / `ActionType` / `OrderType` / `SortDir` / `Language` 都是枚举
- 各接口的模型不同：`OptionChainV3Model`（chain）、`OptionBasicModel`（brief/depth）、
  `OptionKlineV2Model`（kline）、`BatchApiModel<OptionCommonModel>`（trade tick）、
  `OptionModel`（HK symbols）、`OptionAnalysisModel`（analysis）
- **SDK 没有 `OPTION_INDICATOR` 常量**；单合约 Greeks 通过
  `OPTION_CHAIN` + `ReturnGreekValue = true` 获取
- **SDK 没有 `PLACE_COMBO_ORDER` 常量**；组合单用 `BuildMultiLegOrder` + `PLACE_ORDER`
- **SDK 没有 `OPTION_TIMELINE` 的模型/响应类型**（仅有常量）
- 合约类型是 `ContractItem`，不是 `Contract`
- 港股期权需先用 `ALL_HK_OPTION_SYMBOLS` 获取代码映射
- 期权每张合约通常代表 100 股标的；行权价小数位须和期权链一致
- `Execute`/`ExecuteAsync` 不抛异常，检查 `IsSuccess()` / `Code`
