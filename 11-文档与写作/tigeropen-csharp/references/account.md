
# Tiger OpenAPI C# SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Trade;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/"
};
TradeClient tradeClient = new TradeClient(config);
```

> **约定 / Conventions**
> - 每次调用都是 `new TigerRequest<TResponse>() { ApiMethodName = ..., ModelValue = ... }`
>   再 `ExecuteAsync` / `Execute`
> - `TradeClient.Validate()` 会自动填入 `Account`（除 `ACCOUNTS` 接口）与
>   机构用户的 `SecretKey`，已显式赋值的不会被覆盖
> - 枚举参数用枚举值（`SegmentType.SEC`、`Currency.USD`），**不能传字符串**
> - Every call builds a `TigerRequest<TResponse>`; enum params need enum values, not strings.

---

## 账户列表 / Account List

```csharp
using TigerOpenAPI.Model;
using TigerOpenAPI.Trade;
using TigerOpenAPI.Trade.Response;

// ACCOUNTS 用基类 ApiModel —— 没有 AccountModel 类型
var accountsRequest = new TigerRequest<AccountsResponse>()
{
    ApiMethodName = TradeApiService.ACCOUNTS,
    ModelValue = new ApiModel()
};
AccountsResponse accountsResponse = await tradeClient.ExecuteAsync(accountsRequest);
```

`AccountsResponse.Data` 是 `Dictionary<string, List<AccountItem>>`。字段说明 / Fields:
- 账户号（综合 5~10 位数字，模拟 17 位，环球以 U 开头）
- `capability` — `CASH`（现金）/ `RegTMargin`（保证金）/ `PMGRN`（组合保证金）
- `status` — `Funded` / `Open` / `Pending` / `Rejected` / `Closed`
- 账户类型 — `STANDARD` / `GLOBAL` / `PAPER`

> `ACCOUNTS` 是唯一不自动注入 `Account` 的交易接口。
> `ACCOUNTS` is the only trade API that does not get `Account` auto-injected.

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```csharp
using TigerOpenAPI.Common;
using TigerOpenAPI.Trade.Model;

// 注意：返回类型是通用 TigerDictResponse，模型是 GlobalAssetsModel
var globalAssetsRequest = new TigerRequest<TigerDictResponse>()
{
    ApiMethodName = TradeApiService.ASSETS,
    ModelValue = new GlobalAssetsModel()
    {
        Segment = true,      // 按证券/期货分类
        MarketValue = true   // 按市场分市值（仅环球账户）
    }
};
TigerDictResponse globalAssets = await tradeClient.ExecuteAsync(globalAssetsRequest);
// globalAssets.Data 是 Dictionary<string, object>
```

> `ASSETS` 接口**没有** `AssetsResponse` / `AssetModel` 类型；
> 用 `TigerDictResponse` + `GlobalAssetsModel`。
> There is no `AssetsResponse`/`AssetModel`; use `TigerDictResponse` + `GlobalAssetsModel`.

### 综合/模拟账号 / Standard/Paper Account

```csharp
using TigerOpenAPI.Common.Enum;

var primeRequest = new TigerRequest<PrimeAssetResponse>()
{
    ApiMethodName = TradeApiService.PRIME_ASSETS,
    ModelValue = new PrimeAssetsModel()
    {
        BaseCurrency = Currency.USD.ToString(),
        Consolidated = true   // SEC+FUND 聚合显示
    }
};
PrimeAssetResponse primeAssets = await tradeClient.ExecuteAsync(primeRequest);
PrimeAssetItem primeItem = primeAssets.Data;
```

> 模型是 `PrimeAssetsModel`（复数 Assets），响应是 `PrimeAssetResponse`（单数 Asset）。
> `BaseCurrency` 是 `string`，需 `.ToString()`；`Consolidated` 是 `Boolean`。

`segments` 主要字段：
- `category` — S（证券）/ C（期货）/ F（基金）/ D（数字货币）
- `capability` — RegTMargin / Cash
- 购买力、可用资金、现金余额、净清算值
- 初始保证金、维持保证金（低于此值会强平）
- 浮动盈亏、按币种（USD/HKD/SGD/CNH）细分

---

## 账户持仓 / Account Positions

```csharp
var positionsRequest = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionsModel()
    {
        SecType = SecType.STK,     // 枚举，不是字符串
        Market = Market.US,
        Currency = Currency.USD
    }
};
PositionsResponse positionsResponse = await tradeClient.ExecuteAsync(positionsRequest);

// Data 是 PositionsItem，持仓列表在 Data.Items 中
PositionsItem positionsItem = positionsResponse.Data;
foreach (var p in positionsItem.Items)
{
    Console.WriteLine($"{p.Symbol} {p.PositionQty} {p.AverageCost} {p.MarketValue}");
}
```

> 模型是 `PositionsModel`（复数），**不是** `PositionModel`。
> `PositionsResponse.Data` 是 `PositionsItem`，真正的列表在 `Data.Items`，
> 不能直接遍历 `Data`。
> `PositionDetail` 用 `PositionQty`（**没有** `Quantity`）与 `LatestPrice`
> （**没有** `MarketPrice`）。

`PositionDetail` 常用字段：`Symbol`、`PositionQty`、`SalableQty`、`AverageCost`、
`MarketValue`、`LatestPrice`、`UnrealizedPnl`、`UnrealizedPnlPercent`、
`RealizedPnl`、`TodayPnl`。

### 期权持仓 / Option Positions

```csharp
var optPositionsRequest = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionsModel()
    {
        SecType = SecType.OPT,
        Right = "CALL",       // Right 是 string
        Strike = 150.0        // Strike 是 Double
    }
};
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```csharp
var analyticsRequest = new TigerRequest<PrimeAnalyticsAssetResponse>()
{
    ApiMethodName = TradeApiService.ANALYTICS_ASSET,
    ModelValue = new PrimeAnalyticsAssetModel()
    {
        StartDate = "2026-01-01",   // yyyy-MM-dd
        EndDate = "2026-01-31",
        SegType = SegmentType.SEC,
        Currency = Currency.USD
    }
};
PrimeAnalyticsAssetResponse analytics = await tradeClient.ExecuteAsync(analyticsRequest);
```

> 常量是 `TradeApiService.ANALYTICS_ASSET`，**不是** `PRIME_ANALYTICS_ASSET`。
> `SegType` 是 `SegmentType` 枚举，`Currency` 是 `Currency` 枚举。

返回内容包含汇总（盈亏金额、收益率、年化收益率）与按日历史
（日期毫秒时间戳、总资产、当日盈亏、现金余额、持仓市值、入金、出金）。

---

## 最大可交易数量 / Estimate Tradable Quantity

```csharp
var qtyRequest = new TigerRequest<EstimateTradableQuantityResponse>()
{
    ApiMethodName = TradeApiService.ESTIMATE_TRADABLE_QUANTITY,
    ModelValue = new EstimateTradableQuantityModel()
    {
        Symbol = "AAPL",
        SecType = SecType.STK,        // 枚举
        Action = ActionType.BUY,      // 枚举
        OrderType = OrderType.LMT,    // 枚举
        LimitPrice = 150.0
    }
};
EstimateTradableQuantityResponse qty = await tradeClient.ExecuteAsync(qtyRequest);
TradableQuantityItem qtyItem = qty.Data;
```

返回现金可买/卖数量、融资融券可买/卖数量、持仓数量与持仓可交易数量。

---

## 资金转账（Segment 间）/ Segment Fund Transfer

三个接口共用同一个 `SegmentFundModel` / All three share one `SegmentFundModel`:

### 查询可转出金额 / Query Available Amount

```csharp
var availRequest = new TigerRequest<SegmentFundAvailableResponse>()
{
    ApiMethodName = TradeApiService.SEGMENT_FUND_AVAILABLE,
    ModelValue = new SegmentFundModel()
    {
        FromSegment = SegmentType.SEC,   // 枚举
        Currency = Currency.USD          // 枚举
    }
};
SegmentFundAvailableResponse avail = await tradeClient.ExecuteAsync(availRequest);
```

### 发起转账 / Transfer

```csharp
var transferRequest = new TigerRequest<SegmentFundResponse>()
{
    ApiMethodName = TradeApiService.TRANSFER_SEGMENT_FUND,
    ModelValue = new SegmentFundModel()
    {
        FromSegment = SegmentType.SEC,
        ToSegment = SegmentType.FUT,
        Currency = Currency.HKD,
        Amount = 1000D
    }
};
SegmentFundResponse transferred = await tradeClient.ExecuteAsync(transferRequest);
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

> 常量是 `TRANSFER_SEGMENT_FUND`（动词在前），**不是** `SEGMENT_FUND_TRANSFER`。
> 模型统一是 `SegmentFundModel`，没有 `SegmentFundTransferModel` / `SegmentFundAvailableModel`。

### 撤销转账与历史 / Cancel & History

```csharp
var cancelFundRequest = new TigerRequest<SegmentFundResponse>()
{
    ApiMethodName = TradeApiService.CANCEL_SEGMENT_FUND,
    ModelValue = new SegmentFundModel() { Id = 30359957871001600L }
};

var historyRequest = new TigerRequest<SegmentFundsResponse>()
{
    ApiMethodName = TradeApiService.SEGMENT_FUND_HISTORY,
    ModelValue = new SegmentFundModel() { Limit = 10 }
};
```

> 常量是 `CANCEL_SEGMENT_FUND`，**不是** `SEGMENT_FUND_CANCEL`。
> `SegmentFundModel.Id` 与 `Limit` 是**公开字段**（不是属性），类型分别为 `Int64` / `Int32`。

---

## 出入金记录 / Funding Records

SDK **没有** `DEPOSIT_WITHDRAW` 常量。出入金用 `TRANSFER_FUND`，资金明细用 `FUND_DETAILS`。
There is no `DEPOSIT_WITHDRAW`; use `TRANSFER_FUND` and `FUND_DETAILS`.

```csharp
var depositRequest = new TigerRequest<DepositWithdrawResponse>()
{
    ApiMethodName = TradeApiService.TRANSFER_FUND,
    ModelValue = new DepositWithdrawModel()
    {
        SegType = SegmentType.SEC,
        Limit = 20,
        Page = 1
    }
};

var fundDetailsRequest = new TigerRequest<FundDetailsResponse>()
{
    ApiMethodName = TradeApiService.FUND_DETAILS,
    ModelValue = new FundDetailsModel()
    {
        SegTypes = new List<string> { "SEC" },
        Currency = "USD",
        Limit = 20L
    }
};
```

> `DepositWithdrawModel.SegType` 是 `SegmentType?`；`FundDetailsModel.SegTypes`
> 是 `List<string>`、`Currency` 是 `string`、`Limit` 是 `long?`。
> 这两个接口在 SDK 示例中未被调用过，字段以源码为准。

---

## 聚合资产 / Aggregate Assets

```csharp
var aggregateRequest = new TigerRequest<AggregateAssetResponse>()
{
    ApiMethodName = TradeApiService.AGGREGATE_ASSETS,
    ModelValue = new AggregateAssetModel() { SegType = "SEC" }
};
AggregateAssetResponse aggregate = await tradeClient.ExecuteAsync(aggregateRequest);
```

> `AggregateAssetModel.SegType` / `BaseCurrency` 都是 `string`。

---

## 注意事项 / Notes

- 环球账户(Global)用 `ASSETS` + `TigerDictResponse`；综合/模拟账户用
  `PRIME_ASSETS` + `PrimeAssetResponse`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- **枚举参数必须传枚举值**（`SecType.STK`、`SegmentType.SEC`、`Currency.USD`、
  `ActionType.BUY`、`OrderType.LMT`、`Language.en_US`），不能传字符串
- `PositionsResponse.Data` 是 `PositionsItem`，列表在 `Data.Items`
- `PositionDetail` 用 `PositionQty` 与 `LatestPrice`
- 动词在前的常量：`TRANSFER_SEGMENT_FUND`、`CANCEL_SEGMENT_FUND`、`ANALYTICS_ASSET`
- `Execute`/`ExecuteAsync` 不抛异常，检查 `IsSuccess()` / `Code`
- 机构用户设置 `TigerConfig.SecretKey`，SDK 会自动注入到 `TradeModel` 子类请求
