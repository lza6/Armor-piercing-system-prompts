# Tiger OpenAPI C# SDK — Real-time Push / 实时推送

> C# SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Push;
using TigerOpenAPI.Push.Model;
using TigerOpenAPI.Quote.Pb;
using TigerOpenAPI.Common.Enum;

TigerConfig pushConfig = new TigerConfig()
{
    ConfigFilePath = "path/to/config/",
    IsSslSocket = true  // 默认已是 true
};

// PushClient 构造函数是 private，必须用 GetInstance()
// The PushClient ctor is private — use GetInstance()
PushClient pushClient = PushClient.GetInstance()
    .Config(pushConfig)
    .ApiComposeCallback(new MyApiComposeCallback());

await pushClient.ConnectAsync();   // 或同步 pushClient.Connect()
```

> **关键约定 / Key conventions**
> - 回调数据类型在两个命名空间：`TigerOpenAPI.Quote.Pb`（`QuoteBasicData`、`AssetData`、
>   `PositionData`、`OrderStatusData` 等 protobuf 类型）与 `TigerOpenAPI.Push.Model`
>   （`TradeTick`、`SubscribedSymbol`）
> - **不存在 `AckModel` 类型**；错误/踢线回调用 `string` / `int` 参数
> - 订阅方法返回 `uint`（请求 id）；未连接时返回 `0`
> - There is no `AckModel`; error callbacks take `string`/`int`.

---

## 实现回调接口 / Implement Callback Interface

`IApiComposeCallback` 继承 `ISubscribeApiCallback`，**共 27 个成员**，全部必须实现。
SDK 自带参考实现 `Sample/DefaultApiComposeCallback.cs`，采用**显式接口实现**写法。
`IApiComposeCallback` has **27 members**; the SDK's reference impl uses explicit
interface implementation.

```csharp
public class MyApiComposeCallback : IApiComposeCallback
{
    // ===== 连接生命周期（IApiComposeCallback 独有的 8 个成员）=====
    // 这 8 个用显式接口实现写法 / declared explicitly on the interface

    void IApiComposeCallback.ConnectionAck()
    {
        Console.WriteLine("Connected!");
        // 首次订阅可放在此处 / place the initial subscribe here
    }

    void IApiComposeCallback.ConnectionAck(int serverSendInterval, int serverReceiveInterval)
    {
        Console.WriteLine($"Connected. send={serverSendInterval} recv={serverReceiveInterval}");
    }

    void IApiComposeCallback.ConnectionClosed()
    {
        Console.WriteLine("Disconnected.");
    }

    void IApiComposeCallback.ConnectionKickout(int errorCode, string errorMsg)
    {
        Console.WriteLine($"Kicked out [{errorCode}]: {errorMsg}");
    }

    void IApiComposeCallback.HearBeat(string heartBeatContent)
    {
        // 注意方法名拼写是 HearBeat（不是 HeartBeat）
    }

    void IApiComposeCallback.ServerHeartBeatTimeOut(string channelId)
    {
        Console.WriteLine($"Heartbeat timeout, channel: {channelId}");
    }

    void IApiComposeCallback.Error(string errorMsg)
    {
        Console.WriteLine($"Error: {errorMsg}");
    }

    void IApiComposeCallback.Error(int id, int errorCode, string errorMsg)
    {
        Console.WriteLine($"Error id={id} code={errorCode}: {errorMsg}");
    }

    // ===== 行情数据回调（ISubscribeApiCallback，public 实现）=====

    public void QuoteChange(QuoteBasicData data)
    {
        Console.WriteLine($"[行情] {data.Symbol} {data.LatestPrice}");
    }

    public void QuoteAskBidChange(QuoteBBOData data) { }
    public void OptionChange(QuoteBasicData data) { }
    public void OptionAskBidChange(QuoteBBOData data) { }
    public void FutureChange(QuoteBasicData data) { }
    public void FutureAskBidChange(QuoteBBOData data) { }
    public void DepthQuoteChange(QuoteDepthData data) { }
    public void KlineChange(KlineData data) { }
    public void TradeTickChange(TradeTick data) { }        // Push.Model 类型
    public void FullTickChange(TickData data) { }
    public void StockTopPush(StockTopData data) { }
    public void OptionTopPush(OptionTopData data) { }

    // ===== 账户数据回调 =====

    public void AssetChange(AssetData data)
    {
        Console.WriteLine($"[资产] {data.Account} 净资产={data.NetLiquidation}");
    }

    public void PositionChange(PositionData data)
    {
        Console.WriteLine($"[持仓] {data.Symbol} {data.Position}");
    }

    public void OrderStatusChange(OrderStatusData data)
    {
        Console.WriteLine($"[订单] {data.Symbol} {data.Status}");
    }

    public void OrderTransactionChange(OrderTransactionData data) { }

    // ===== 订阅结果回调 =====

    public void SubscribeEnd(int id, string subject, string result) { }
    public void CancelSubscribeEnd(int id, string subject, string result) { }
    public void GetSubscribedSymbolEnd(SubscribedSymbol subscribedSymbol) { }
}
```

### 27 个成员一览 / All 27 Members

**`IApiComposeCallback` 独有（8）**：`Error(string)`、`Error(int, int, string)`、
`ConnectionClosed()`、`ConnectionKickout(int, string)`、`ConnectionAck()`、
`ConnectionAck(int, int)`、`HearBeat(string)`、`ServerHeartBeatTimeOut(string)`

**继承自 `ISubscribeApiCallback`（19）**：`OrderStatusChange`、`OrderTransactionChange`、
`PositionChange`、`AssetChange`、`TradeTickChange`、`FullTickChange`、`QuoteChange`、
`QuoteAskBidChange`、`OptionChange`、`OptionAskBidChange`、`FutureChange`、
`FutureAskBidChange`、`DepthQuoteChange`、`KlineChange`、`StockTopPush`、
`OptionTopPush`、`SubscribeEnd`、`CancelSubscribeEnd`、`GetSubscribedSymbolEnd`

---

## 订阅方法 / Subscribe Methods

所有订阅/退订方法返回 `uint`（请求 id）；未连接时返回 `0`。
All subscribe/unsubscribe methods return a `uint` request id, or `0` if not connected.

```csharp
// 账户类：用 Subject 枚举 / Account-family: use the Subject enum
pushClient.Subscribe(Subject.Asset);
pushClient.Subscribe(Subject.Position);
pushClient.Subscribe(Subject.OrderStatus);
pushClient.Subscribe(Subject.OrderTransaction);
pushClient.Subscribe(Subject.Asset, "your_account");   // 指定账户
pushClient.CancelSubscribe(Subject.Asset);

// 行情类：接收 ISet<string> / Quote-family: take an ISet<string>
pushClient.SubscribeQuote(new HashSet<string> { "AAPL", "TSLA" });
pushClient.CancelSubscribeQuote(new HashSet<string> { "TSLA" });
pushClient.CancelSubscribeQuote();   // 省略参数 = 全部退订

pushClient.SubscribeTradeTick(new HashSet<string> { "AAPL" });
pushClient.SubscribeDepthQuote(new HashSet<string> { "AAPL" });
pushClient.SubscribeKline(new HashSet<string> { "AAPL" });
pushClient.SubscribeOption(new HashSet<string> { "AAPL  260821C00150000" });
pushClient.SubscribeFuture(new HashSet<string> { "CLmain" });

// 榜单与全市场 / Ranking lists and whole-market
pushClient.SubscribeStockTop(Market.US);
pushClient.SubscribeOptionTop(Market.US);
pushClient.SubscribeMarketQuote(Market.US, QuoteSubject.Quote);

// 查询已订阅标的 / Query current subscriptions
pushClient.GetSubscribedSymbols();

// 连接状态与断开 / State and disconnect
bool connected = pushClient.IsConnected();
string url = pushClient.GetUrl();
pushClient.Disconnect();
```

> **没有** `SubscribeOrder()` 方法，也**没有** C# 事件（`OrderAssetChange += ...`）
> 或 `PushClientFactory`。账户推送用 `Subscribe(Subject.OrderStatus)`。
> There is no `SubscribeOrder()`, no events, and no `PushClientFactory`.

`Subject` 枚举：`None`、`Asset`、`Position`、`OrderStatus`、`OrderTransaction`。
`QuoteSubject` 枚举：`None`、`Quote`、`Option`、`Future`、`QuoteDepth`、`TradeTick`、
`StockTop`、`OptionTop`、`Kline`。

---

## 数据字段 / Data Fields

`AssetData`（`TigerOpenAPI.Quote.Pb`）：`Account`、`Currency`、`SegType`、
`NetLiquidation`、`CashBalance`、`BuyingPower`、`AvailableFunds`、`ExcessLiquidity`、
`EquityWithLoan`、`GrossPositionValue`、`InitMarginReq`、`MaintMarginReq`、`Timestamp`

`PositionData`：`Account`、`Symbol`、`Position`、`PositionScale`、`AverageCost`、
`LatestPrice`、`MarketValue`、`Expiry`、`Strike`、`Right`、`Identifier`、`Multiplier`

`OrderStatusData`：`Id`、`Account`、`Symbol`、`Action`、`OrderType`、`Status`、
`TotalQuantity`、`FilledQuantity`、`AvgFillPrice`、`LimitPrice`、`StopPrice`、`RealizedPnl`

---

## 注意事项 / Notes

- `PushClient` 是单例，构造函数私有，必须 `PushClient.GetInstance()`
- `IApiComposeCallback` 共 27 个成员，全部必须实现（可空实现）
- **不存在 `AckModel`**；`Error` / `ConnectionKickout` 用 `string` / `int` 参数
- 方法名拼写是 `HearBeat`（不是 `HeartBeat`）
- 回调数据类型分布在 `TigerOpenAPI.Quote.Pb` 与 `TigerOpenAPI.Push.Model` 两个命名空间
- 行情订阅接收 `ISet<string>`（如 `HashSet<string>`），不是 `List<string>`
- 账户订阅用 `Subscribe(Subject.Xxx)`，没有 `SubscribeOrder()` 之类的专用方法
- 订阅方法在未连接时返回 `0` 且只记录日志，不抛异常
- 同一进程只有一个 `PushClient` 实例（单例）
