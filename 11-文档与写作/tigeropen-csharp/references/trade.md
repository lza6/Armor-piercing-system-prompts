# Tiger OpenAPI C# SDK — Trading / 交易

> C# SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**每步均为必须，缺少任何步骤不得下单**：
1. 调用 `PreviewOrder` 查看预估佣金和保证金，展示给用户
2. 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止等待用户明确确认**；未收到确认前**禁止调用 `PlaceOrder`**
3. 用户确认后调用 `PlaceOrder`
4. 下单后通过查询订单 API 确认订单状态

---

## 初始化 / Initialize

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Trade;
using TigerOpenAPI.Common.Enum;

TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/",
};
TradeClient tradeClient = new TradeClient(config);
string account = config.DefaultAccount;
```

---

## 查询账户 / Query Accounts

```csharp
using TigerOpenAPI.Trade.Model;

var request = new TigerRequest<AccountsResponse>()
{
    ApiMethodName = TradeApiService.ACCOUNTS,
    ModelValue = new ApiModel()
};
var response = await tradeClient.ExecuteAsync(request);
// 返回账户列表，含 accountId, accountType 等
```

---

## 下单 / Place Orders
<!-- 当用户提到"买入"、"卖出"、"下单"、"buy"、"sell"、"order"时 -->

C# SDK 使用 `PlaceOrderModel` 的静态工厂方法构造订单，再通过 `TigerRequest` 提交：

### 限价单 / Limit Order

```csharp
// 1. 构造合约 / Build contract
var contract = new ContractItem()
{
    Symbol = "AAPL",
    SecType = "STK",
    Market = "US",
    Currency = "USD"
};

// 2. 构造订单 / Build order
var orderModel = PlaceOrderModel.BuildLimitOrder(
    account,          // 账户号
    contract,         // 合约
    ActionType.BUY,   // 买卖方向: ActionType.BUY / ActionType.SELL
    100L,             // 数量
    150.0             // 限价
);

// 3. 提交下单 / Submit order
var request = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PLACE_ORDER,
    ModelValue = orderModel
};
var response = await tradeClient.ExecuteAsync(request);
Console.WriteLine($"Order ID: {response?.Data?.Id}");
// 返回成功仅表示订单已提交，不代表已成交
```

### 市价单 / Market Order

```csharp
var orderModel = PlaceOrderModel.BuildMarketOrder(
    account, contract, ActionType.BUY, 100L
);
```

### 止损限价单 / Stop-Limit Order

```csharp
var orderModel = PlaceOrderModel.BuildStopLimitOrder(
    account,
    contract,
    ActionType.SELL,
    100L,
    145.0,  // limitPrice（限价）
    148.0   // auxPrice（止损触发价）
);
```

### 追踪止损单 / Trail Order

```csharp
var orderModel = PlaceOrderModel.BuildTrailOrder(
    account,
    contract,
    ActionType.SELL,
    100L,
    10.0,   // trailingPercent（追踪百分比）
    0.0     // auxPrice（触发价，0 = 市价触发）
);
```

### 盘前/盘后交易 / Extended Hours

```csharp
var orderModel = PlaceOrderModel.BuildLimitOrder(account, contract, ActionType.BUY, 100L, 150.0);
orderModel.OutsideRth = true;   // 允许盘前盘后
orderModel.TimeInForce = TimeInForce.GTC; // 枚举: GTC / DAY / GTD
```

### 期权下单 / Option Order

```csharp
var optContract = new ContractItem()
{
    Symbol = "AAPL",
    SecType = "OPT",
    Expiry = "20250829",  // YYYYMMDD
    Strike = 150.0,
    Right = "CALL",       // CALL / PUT
    Currency = "USD",
    Market = "US"
};
var optOrder = PlaceOrderModel.BuildLimitOrder(account, optContract, ActionType.BUY, 1L, 5.0);
// 期权数量单位为"张"（1张 = 100股）
```

---

## 预览订单 / Preview Order

```csharp
var orderModel = PlaceOrderModel.BuildLimitOrder(account, contract, ActionType.BUY, 100L, 150.0);

var request = new TigerRequest<PlaceOrderResponse>()
{
    ApiMethodName = TradeApiService.PREVIEW_ORDER,
    ModelValue = orderModel
};
var response = await tradeClient.ExecuteAsync(request);
// 返回预估保证金、佣金等信息
```

---

## 修改订单 / Modify Order

```csharp
// orderId 来自下单响应：placeResponse.Data.Id
long orderId = 12345678901234567L;

var modifyModel = new ModifyOrderModel()
{
    Account = account,
    Id = orderId,            // 内部订单 ID（long）
    LimitPrice = 155.0,      // 新限价
    TotalQuantity = 100L     // 新数量
};

var request = new TigerRequest<TigerDictResponse>()
{
    ApiMethodName = TradeApiService.MODIFY_ORDER,
    ModelValue = modifyModel
};
```

---

## 撤单 / Cancel Order

```csharp
var cancelModel = new CancelOrderModel()
{
    Account = account,
    Id = orderId    // 内部订单 ID（long）
};

var request = new TigerRequest<TigerDictResponse>()
{
    ApiMethodName = TradeApiService.CANCEL_ORDER,
    ModelValue = cancelModel
};
var response = await tradeClient.ExecuteAsync(request);
```

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"、"order status"时 -->

```csharp
// 全部订单 / All orders
var request = new TigerRequest<OrderBatchResponse>()
{
    ApiMethodName = TradeApiService.ORDERS,
    ModelValue = new QueryOrderModel() { Account = account }
};

// 待成交订单 / Active (pending) orders
var activeRequest = new TigerRequest<OrderBatchResponse>()
{
    ApiMethodName = TradeApiService.ACTIVE_ORDERS,
    ModelValue = new QueryOrderModel() { Account = account }
};

// 已成交订单 / Filled orders
var now = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();
var ninetyDaysAgo = now - 90L * 24 * 3600 * 1000;
var filledRequest = new TigerRequest<OrderBatchResponse>()
{
    ApiMethodName = TradeApiService.FILLED_ORDERS,
    ModelValue = new QueryOrderModel()
    {
        Account = account,
        StartDate = ninetyDaysAgo,
        EndDate = now
    }
};

// 已撤销订单 / Cancelled orders
var inactiveRequest = new TigerRequest<OrderBatchResponse>()
{
    ApiMethodName = TradeApiService.INACTIVE_ORDERS,
    ModelValue = new QueryOrderModel() { Account = account }
};
```

---

## 成交明细 / Order Transactions

```csharp
var request = new TigerRequest<OrderTransactionsResponse>()
{
    ApiMethodName = TradeApiService.ORDER_TRANSACTIONS,
    ModelValue = new OrderTransactionsModel()
    {
        Account = account,
        OrderId = 31318009878020096L   // 可选
        // Symbol = "AAPL"              // 或按标的查询
    }
};
```

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"我的股票"、"positions"时 -->

```csharp
var request = new TigerRequest<PositionsResponse>()
{
    ApiMethodName = TradeApiService.POSITIONS,
    ModelValue = new PositionsModel()
    {
        Account = account,
        SecType = SecType.STK,   // 枚举，可选: STK/OPT/FUT
        Market = Market.US,      // 枚举，可选: US/HK
        // Symbol = "AAPL"       // 可选
    }
};
var response = await tradeClient.ExecuteAsync(request);

if (response?.Data?.Items != null)
{
    foreach (var pos in response.Data.Items)
    {
        Console.WriteLine($"{pos.Symbol}: qty={pos.PositionQty}, " +
                          $"cost={pos.AverageCost}, pnl={pos.UnrealizedPnl}");
    }
}
```

### Position 字段 / Position Fields

| 字段 Field | 说明 Description | APP 对应 |
|-----------|-----------------|---------|
| `Symbol` | 标的代码 | — |
| `PositionQty` | 持仓数量（**没有** `Quantity` 字段） | 持有数量 |
| `AverageCost` | 含佣金持仓均价 | 平均成本 |
| `LatestPrice` | 最新价格（**没有** `MarketPrice` 字段） | 现价 |
| `MarketValue` | 市值 | 市值 |
| `UnrealizedPnl` | 浮动盈亏 | 未实现盈亏 |
| `UnrealizedPnlPercent` | 浮动盈亏率 | 盈亏百分比 |
| `RealizedPnl` | 已实现盈亏 (FIFO) | 已实现盈亏 |
| `SalableQty` | 可卖数量 | 可卖数量 |
| `TodayPnl` | 今日盈亏额 | 今日盈亏 |

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"、"balance"时 -->

```csharp
// 普通账户资产 / Standard account assets
var request = new TigerRequest<TigerDictResponse>()
{
    ApiMethodName = TradeApiService.ASSETS,
    ModelValue = new PrimeAssetsModel()
    {
        Account = account,
        BaseCurrency = "USD"  // 统一换算为 USD，多币种时避免直接累加
    }
};

// 综合账户资产 (Prime) / Prime account assets
var primeRequest = new TigerRequest<PrimeAssetResponse>()
{
    ApiMethodName = TradeApiService.PRIME_ASSETS,
    ModelValue = new PrimeAssetsModel() { Account = account }
};
var primeResponse = await tradeClient.ExecuteAsync(primeRequest);
```

### 综合账户 Segment 关键字段 / Prime Segment Fields

| 字段 Field | 说明 Description | APP 对应 |
|-----------|-----------------|---------|
| `NetLiquidation` | 总资产（净清算价值） | 总资产 |
| `CashBalance` | 现金额 | 现金 |
| `CashAvailableForTrade` | 可用资金 | 可用资金 |
| `CashAvailableForWithdrawal` | 可提现金额 | 可提金额 |
| `GrossPositionValue` | 证券总价值 | 市值 |
| `BuyingPower` | 购买力 | 购买力 |
| `UnrealizedPl` | 浮动盈亏 | 未实现盈亏 |
| `RealizedPl` | 已实现盈亏 | 已实现盈亏 |
| `InitMargin` | 初始保证金 | 初始保证金 |
| `MaintainMargin` | 维持保证金 | 维持保证金 |
| `Leverage` | 杠杆倍数 | 杠杆 |

> ⚠️ **多币种注意**：同时持有 USD 和 HKD 标的时，指定 `BaseCurrency` 统一换算口径，避免直接累加不同币种数值。

---

## 可交易数量 / Estimate Tradable Quantity

```csharp
var orderModel = PlaceOrderModel.BuildLimitOrder(account, contract, ActionType.BUY, 1L, 150.0);

var request = new TigerRequest<EstimateTradableQuantityResponse>()
{
    ApiMethodName = TradeApiService.ESTIMATE_TRADABLE_QUANTITY,
    ModelValue = orderModel
};
var response = await tradeClient.ExecuteAsync(request);
```

---

## 合约查询 / Contract Query

```csharp
// 单个合约 / Single contract
var request = new TigerRequest<ContractResponse>()
{
    ApiMethodName = TradeApiService.CONTRACT,
    ModelValue = new ContractModel() { Symbol = "AAPL", SecType = "STK" }
};

// 批量合约 / Multiple contracts
var batchRequest = new TigerRequest<ContractsResponse>()
{
    ApiMethodName = TradeApiService.CONTRACTS,
    ModelValue = new ContractsModel()
    {
        Symbols = new List<string> { "AAPL", "TSLA" },
        SecType = "STK"
    }
};
```

---

## TradeApiService 常量速查 / Constants

| 常量 Constant | API 说明 |
|--------------|---------|
| `PLACE_ORDER` | 下单 |
| `CANCEL_ORDER` | 撤单 |
| `MODIFY_ORDER` | 改单 |
| `PREVIEW_ORDER` | 预览订单 |
| `ACCOUNTS` | 账户列表 |
| `ASSETS` | 普通账户资产 |
| `PRIME_ASSETS` | 综合账户资产 |
| `POSITIONS` | 持仓 |
| `ORDERS` | 全部订单 |
| `ACTIVE_ORDERS` | 待成交订单 |
| `INACTIVE_ORDERS` | 已撤销订单 |
| `FILLED_ORDERS` | 已成交订单 |
| `ORDER_TRANSACTIONS` | 成交明细 |
| `ESTIMATE_TRADABLE_QUANTITY` | 可交易数量 |
| `CONTRACT` | 单个合约信息 |
| `CONTRACTS` | 批量合约信息 |

---

## PlaceOrderModel 字段 / Order Fields

| 字段 Field | 说明 | 示例 |
|-----------|------|------|
| `Symbol` | 标的代码 | `"AAPL"` |
| `SecType` | 证券类型 | `"STK"/"OPT"/"FUT"` |
| `Action` | 买卖方向 | `ActionType.BUY/ActionType.SELL` |
| `OrderType` | 订单类型 | `"LMT"/"MKT"/"STP_LMT"` |
| `TotalQuantity` | 数量 | `100L` |
| `LimitPrice` | 限价 | `150.0` |
| `AuxPrice` | 止损触发价 | `148.0` |
| `TrailingPercent` | 追踪止损百分比 | `10.0` |
| `TimeInForce` | 有效期 | `"DAY"/"GTC"/"GTD"` |
| `OutsideRth` | 允许盘前盘后 | `true` |
| `UserMark` | 自定义标记 | — |

---

## 注意事项 / Notes

- `PlaceOrder` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `ORDERS` 查询或推送回调确认最终成交状态
- `FILLED_ORDERS` 需要提供 `StartDate`（建议不超过 90 天范围）
- 期权下单需先通过 `OPTION_CHAIN` 获取合约详情（identifier）
- `TradeClient` 自动根据账户号判断模拟/实盘并路由
