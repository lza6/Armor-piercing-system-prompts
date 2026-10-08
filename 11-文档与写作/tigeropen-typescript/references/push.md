# Tiger OpenAPI TypeScript SDK — Real-time Push / 实时推送

> TypeScript SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```typescript
import { createClientConfig, PushClient } from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const pc = new PushClient(config);
```

> 回调数据类型是 **protobuf 生成的类型**（`QuoteData`、`OrderStatusData` …）。
> 从包根路径导入 `PushClient`，不要用 `dist/esm/...` 内部路径。
> Callback payloads are protobuf-generated types; import `PushClient` from the package root.

---

## 完整示例 / Full Example

```typescript
const pcFull = new PushClient(config);

pcFull.setCallbacks({
  onQuote: (data) => {
    console.log(`[行情] ${data.symbol} 最新价: ${data.latestPrice} 量: ${data.volume}`);
  },
  onOrder: (data) => {
    console.log(`[订单] ${data.symbol} 状态: ${data.status} 已成交: ${data.filledQuantity}`);
  },
  onAsset: (data) => {
    console.log(`[资产] ${data.account} 净资产: ${data.netLiquidation} 购买力: ${data.buyingPower}`);
  },
  onPosition: (data) => {
    console.log(`[持仓] ${data.symbol} 数量: ${data.position} 市值: ${data.marketValue}`);
  },
  onConnect: () => {
    console.log('推送连接成功');
    // 首次连接后订阅 / Subscribe after the first connect.
    // SDK 重连成功后会自动恢复订阅，无需在此重复订阅
    // SDK auto-restores subscriptions on reconnect — do not re-subscribe here.
    pcFull.subscribeQuote(['AAPL', 'TSLA']);
    pcFull.subscribeAsset();
    pcFull.subscribeOrder();
    pcFull.subscribePosition();
  },
  onDisconnect: () => console.log('推送连接断开'),
  onError: (err) => console.error('推送错误:', err),
  onKickout: (msg) => console.warn('被踢下线:', msg),
});

await pcFull.connect();

process.on('SIGINT', () => {
  pcFull.unsubscribeQuote(['AAPL', 'TSLA']);
  pcFull.disconnect();
  process.exit(0);
});
```

---

## 回调函数 / Callbacks

全部定义在 `Callbacks`（`src/push/callbacks.ts`），均为可选字段。
数据类型来自 protobuf / All payload types are protobuf-generated:

| 回调 Callback | 触发时机 | 数据类型 |
|--------------|---------|---------|
| `onQuote` | 基础行情更新 | `QuoteData` |
| `onQuoteBBO` | 最优买卖价更新 | `QuoteData` |
| `onTick` | 逐笔成交 | `TradeTickData` |
| `onFullTick` | 全量逐笔 | `TickData` |
| `onDepth` | 盘口深度 | `QuoteDepthData` |
| `onKline` | K 线更新 | `KlineData` |
| `onOption` | 期权行情 | `QuoteData` |
| `onFuture` | 期货行情 | `QuoteData` |
| `onStockTop` | 股票榜单 | `StockTopData` |
| `onOptionTop` | 期权榜单 | `OptionTopData` |
| `onAsset` | 账户资产变动 | `AssetData` |
| `onPosition` | 持仓变动 | `PositionData` |
| `onOrder` | 订单状态变化 | `OrderStatusData` |
| `onTransaction` | 成交明细 | `OrderTransactionData` |
| `onConnect` / `onDisconnect` | 连接建立 / 断开 | - |
| `onError` | 发生错误 | `Error` |
| `onKickout` | 被服务端踢下线 | `string` |

---

## 订阅方法 / Subscribe Methods

```typescript
// 行情类：接收 symbol 数组 / Quote-family: take a symbol array
pc.subscribeQuote(['AAPL', 'TSLA', '00700']);
pc.unsubscribeQuote(['TSLA']);      // 参数可省略，省略表示全部退订

pc.subscribeTick(['AAPL']);
pc.subscribeDepth(['AAPL']);
pc.subscribeKline(['AAPL']);
pc.subscribeOption(['AAPL  260821C00150000']);
pc.subscribeFuture(['CLmain']);
pc.subscribeCc(['BTC/USD']);

// 榜单 / Ranking lists
pc.subscribeStockTop('US', ['changeRate']);
pc.subscribeOptionTop('US', ['volume']);

// 全市场 / Whole market
pc.subscribeMarket('US');

// 账户类：订阅可传 account（省略用默认账户），退订【不带参数】
// Account-family: subscribe takes an optional account; UNSUBSCRIBE TAKES NO ARGUMENTS
pc.subscribeAsset();
pc.subscribeOrder();
pc.subscribePosition();
pc.subscribeTransaction();

pc.unsubscribeAsset();
pc.unsubscribeOrder();
pc.unsubscribePosition();
pc.unsubscribeTransaction();

// 查询当前订阅 / Inspect subscriptions
const subs = pc.getSubscriptions();
const acctSubs = pc.getAccountSubscriptions();

// 断开连接 / Disconnect
pc.disconnect();
```

> 账户类退订方法**无参数**：`unsubscribeAsset()`，不是 `unsubscribeAsset(account)`。
> Account unsubscribe methods take **no** arguments.

---

## QuoteData 常用字段 / Common Fields

| 字段 Field | 说明 |
|-----------|------|
| `symbol` | 标的代码 |
| `latestPrice` | 最新价 |
| `latestTime` / `latestPriceTimestamp` | 最新价时间 |
| `volume` / `amount` | 成交量 / 成交额 |
| `open` / `high` / `low` / `preClose` | 开 / 高 / 低 / 昨收 |
| `askPrice` / `askSize` | 卖一价 / 量 |
| `bidPrice` / `bidSize` | 买一价 / 量 |
| `avgPrice` | 均价 |
| `marketStatus` | 市场状态 |
| `identifier` | 标准代码 |
| `openInt` | 未平仓量（期权/期货） |
| `timestamp` | 时间戳（毫秒） |

## AssetData 字段 / AssetData Fields

| 字段 Field | 说明 |
|-----------|------|
| `account` | 账户号 |
| `currency` / `segType` | 币种 / 分段 |
| `netLiquidation` | 净资产 |
| `cashBalance` | 现金余额 |
| `buyingPower` | 购买力 |
| `availableFunds` | 可用资金 |
| `excessLiquidity` | 剩余流动性 |
| `equityWithLoan` | 含借贷权益 |
| `grossPositionValue` | 持仓总市值 |
| `initMarginReq` / `maintMarginReq` | 初始 / 维持保证金 |

## PositionData 字段 / PositionData Fields

| 字段 Field | 说明 |
|-----------|------|
| `account` / `symbol` | 账户 / 标的 |
| `position` | 持仓数量（**不是** `quantity`） |
| `positionScale` | 数量精度 |
| `averageCost` | 平均成本 |
| `latestPrice` | 最新价（**不是** `marketPrice`） |
| `marketValue` | 市值 |
| `expiry` / `strike` / `right` | 期权合约要素 |
| `multiplier` | 合约乘数 |

## OrderStatusData 字段 / OrderStatusData Fields

| 字段 Field | 说明 |
|-----------|------|
| `id` | 订单 ID |
| `symbol` / `identifier` | 标的 / 标准代码 |
| `action` / `orderType` | 方向 / 订单类型 |
| `status` | 订单状态 |
| `totalQuantity` / `filledQuantity` | 总量 / 已成交量 |
| `avgFillPrice` | 成交均价 |

---

## 注意事项 / Notes

- **首次订阅放在 `onConnect` 回调中**，连接成功后才能订阅
- SDK 自动处理断线重连与心跳保活，无需手动重连
- ✅ **断线重连后 SDK 自动恢复订阅**（`resubscribe()` 见 `src/push/push-client.ts:380`）。请勿在 `onConnect` 中重复订阅，否则会产生重复订阅 / SDK auto-restores subscriptions on reconnect; do NOT re-subscribe in `onConnect`
- 回调数据是 protobuf 类型；`PositionData` 用 `position` 与 `latestPrice`，没有 `quantity` / `marketPrice`
- 账户类退订方法不带参数
- 从包根路径 `@tigeropenapi/tigeropen` 导入 `PushClient`
- 同一个 `PushClient` 实例只能有一个活跃连接
