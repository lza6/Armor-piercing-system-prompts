# Tiger OpenAPI TypeScript SDK — Trading / 交易

> TypeScript SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**每步均为必须，缺少任何步骤不得下单**：
1. 调用 `previewOrder()` 查看预估佣金和保证金，展示给用户
2. 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止等待用户明确确认**；未收到确认前**禁止调用 `placeOrder()`**
3. 用户确认后调用 `placeOrder()`
4. 下单后通过 `getOrders()` 确认订单状态

---

## 初始化 / Initialize

```typescript
import {
  createClientConfig,
  HttpClient,
  TradeClient,
  limitOrder,
  marketOrder,
  stopOrder,
  stopLimitOrder,
} from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const tc = new TradeClient(httpClient, config.account);
const account = config.account;
```

> **约定 / Conventions**
> - 查询方法以 `get` 开头（`getOrders`、`getPositions`、`getAssets` …）
> - 查询方法接收**请求对象**（多为可选参数）；返回**强类型对象**
> - 单值接口返回 `T | undefined`，需判空
> - Query methods are `getX`, take an optional request object, and return typed values;
>   single-value APIs return `T | undefined`.

---

## 下单 / Place Orders
<!-- 当用户提到"下单"、"买入"、"卖出"、"buy"、"sell"、"order"时 -->

### 创建订单 / Create Order

所有 helper 的前 4 个参数固定为 `account, symbol, secType, action`。
Every helper starts with `account, symbol, secType, action`.

```typescript
// 限价单 / Limit order
const limit = limitOrder(account, 'AAPL', 'STK', 'BUY', 100, 150.0);

// 市价单 / Market order
const market = marketOrder(account, 'AAPL', 'STK', 'BUY', 100);

// 止损单 / Stop order (auxPrice = 触发价)
const stop = stopOrder(account, 'AAPL', 'STK', 'SELL', 100, 145.0);

// 止损限价单 / Stop-limit order (limitPrice, auxPrice)
const stopLimit = stopLimitOrder(account, 'AAPL', 'STK', 'SELL', 100, 145.0, 148.0);
```

> 根路径只导出 `marketOrder`、`limitOrder`、`stopOrder`、`stopLimitOrder` 四个 helper。
> 其余 helper（`trailOrder`、`icebergOrder`、`comboOrder`、`ocaOrder`、`algoOrder`、
> `auctionLimitOrder`、`auctionMarketOrder`、`marketOrderByAmount`、`limitOrderByAmount`、
> `trailOrderByPrice`、`limitOrderWithLegs`、`orderLeg`、`contractLeg`）未在根路径导出，
> 需要时可直接手工构造 `OrderRequest` 对象。
> Only those four helpers are re-exported from the package root; for the others,
> construct the `OrderRequest` object directly.

### 手工构造订单 / Build an OrderRequest directly

```typescript
import type { OrderRequest } from '@tigeropenapi/tigeropen';

const icebergReq: OrderRequest = {
  account,
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'ICEBERG',
  totalQuantity: 1000,
  limitPrice: 150.0,
  displaySize: 100,
  timeInForce: 'DAY',
};
```

### 预览订单 / Preview Order

```typescript
const preview = await tc.previewOrder(limit);
if (preview) {
  console.log(preview);
}
```

### 提交下单 / Submit Order

```typescript
const placed = await tc.placeOrder(limit);
const orderId = placed?.id;
console.log('order id:', orderId);
```

### 修改订单 / Modify Order

```typescript
// id 类型是 number | string —— 大整数订单 ID 请传【字符串】以避免精度丢失
// id is number | string — pass a STRING for large int64 ids to preserve precision
const modified = await tc.modifyOrder('12345678901234567', {
  ...limit,
  limitPrice: 155.0,
});
```

### 取消订单 / Cancel Order

```typescript
const cancelled = await tc.cancelOrder('12345678901234567');
```

> ⚠️ `modifyOrder` / `cancelOrder` 的 `id` 是 `number | string`。订单 ID 是 int64，
> 超出 JS 安全整数范围时**必须传字符串**，否则精度丢失。
> Order ids are int64 — pass them as strings when they exceed `Number.MAX_SAFE_INTEGER`.

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```typescript
// 请求对象可省略 / the request object is optional
const allOrders = await tc.getOrders({ limit: 50 });
const active = await tc.getActiveOrders();
const filled = await tc.getFilledOrders({ limit: 20 });
const inactive = await tc.getInactiveOrders();

// 单个订单 / single order
const one = await tc.getOrder({ id: 12345678 });

// 订单成交明细 / order transactions
const txns = await tc.getOrderTransactions({ orderId: 12345678 });
```

`getOrders` 等返回 `Order[]`；`getOrder` 返回 `Order | undefined`。

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```typescript
const positions = await tc.getPositions({ secType: 'STK' });
for (const p of positions) {
  console.log(p.symbol, p.positionQty, p.averageCost, p.marketValue, p.unrealizedPnl);
}
```

返回 `Position[]`。可按 `symbol`、`secType`、`market`、`currency` 等过滤。

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```typescript
// 普通/环球账户 / Standard & Global — 返回数组
const assets = await tc.getAssets({ segment: true });

// 综合账户（Prime）—— 返回 PrimeAsset | undefined
const prime = await tc.getPrimeAssets();
if (prime) {
  console.log(prime);
}
```

> `getAssets` 返回 `Asset[]`；`getPrimeAssets` 返回 `PrimeAsset | undefined`。
> 两者都是**强类型对象**，不需要 `JSON.parse`。

---

## 合约查询 / Contract Query

```typescript
const contracts = await tc.getContract('AAPL', 'STK');
const batch = await tc.getContracts(['AAPL', 'TSLA'], 'STK');
const optContract = await tc.getQuoteContract('AAPL', 'OPT', '2026-08-21');

// secType 值: 'STK'(股票)/'OPT'(期权)/'FUT'(期货)/'CASH'(外汇)
```

> 方法名是 `getContract` / `getContracts`（带 `get` 前缀），接收**位置参数**。

---

## 账户与资金 / Account & Funds

```typescript
const accounts = await tc.getManagedAccounts();
const analytics = await tc.getAnalyticsAsset({
  startDate: '2026-01-01',
  endDate: '2026-01-31',
  segType: 'SEC',
});
const est = await tc.getEstimateTradableQuantity({
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'MKT',
});
const aggregate = await tc.getAggregateAssets();
```

资金划转与出入金见 [account.md](account.md) / See account.md for fund transfers.

---

## 期权行权 / Option Exercise

```typescript
// type: 'Exercise'（提前行权）| 'Expire'（提前放弃行权）
// Exercise 时 executingDate（yyyy-MM-dd）与 isForce 必填；itmRate（0-10）为 Expire 专用
const submitted = await tc.submitOptionExercise({
  account,
  contractId: 123456789,
  type: 'Exercise',
  quantity: 1,
  executingDate: '2026-08-21',
  isForce: false,
});

const exCancelled = await tc.cancelOptionExercise({ account, id: 987654321 });
```

两者返回 `boolean` / Both return `boolean`.

---

## OrderRequest 字段说明 / Order Fields

| 字段 | 类型 | 说明 |
|-----|------|------|
| `account` | `string` | 账户 |
| `symbol` | `string` | 标的代码，如 `'AAPL'` |
| `secType` | `string` | `'STK'` / `'OPT'` / `'FUT'` / `'CASH'` |
| `action` | `string` | `'BUY'` / `'SELL'` |
| `orderType` | `string` | `'LMT'` / `'MKT'` / `'STP'` / `'STP_LMT'` / `'TRAIL'` / `'ICEBERG'` |
| `totalQuantity` | `number` | 数量 |
| `limitPrice` | `number` | 限价（LMT/STP_LMT 必填） |
| `auxPrice` | `number` | 止损触发价（STP/STP_LMT 必填） |
| `trailingPercent` | `number` | 跟踪止损百分比 |
| `timeInForce` | `string` | `'DAY'` / `'GTC'` / `'GTD'` |
| `outsideRth` | `boolean` | 是否允许盘前盘后交易 |
| `expiry` / `strike` / `right` | `string` | 期权合约要素 |
| `identifier` | `string` | 期权/期货标准代码 |
| `displaySize` | `number` | 冰山单展示数量 |
| `contractLegs` / `comboType` | — | 组合单腿与类型 |
| `id` / `orderId` | `number` | 订单 ID |

---

## 注意事项 / Notes

- 所有方法都是 `async`，需 `await`
- `modifyOrder` / `cancelOrder` 的 `id` 是 `number | string`；int64 大 ID 必须传字符串
- 查询方法接收请求对象（多为可选）；`getContract`/`getContracts` 接收位置参数
- 单值接口（`placeOrder`、`previewOrder`、`getOrder`、`getPrimeAssets`、
  `getAggregateAssets`）返回 `T | undefined`，需判空
- `placeOrder()` 成功仅表示订单**已提交**，需通过 `getOrders()` 或推送确认成交
- 根路径只导出 4 个 order helper，其余请手工构造 `OrderRequest`
- 机构用户在 `new TradeClient(httpClient, account, secretKey)` 传入 secret key
- 从包根路径 `@tigeropenapi/tigeropen` 导入所有 API，不要用子路径
