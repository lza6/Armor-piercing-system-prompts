
# Tiger OpenAPI TypeScript SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```typescript
import {
  createClientConfig,
  HttpClient,
  TradeClient,
} from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const tc = new TradeClient(httpClient, config.account);
```

> **约定 / Conventions**
> - 方法以 `get` 开头，接收**请求对象**（字段多为可选）；返回**强类型对象**
> - 单值接口返回 `T | undefined`，需判空
> - `account` 省略时使用客户端默认账户
> - Methods are `getX`, take an optional request object, and return typed values.

---

## 账户列表 / Account List

```typescript
const accounts = await tc.getManagedAccounts();
```

返回 `ManagedAccount[]`。字段说明 / Fields:
- 账户号（综合 5~10 位数字，模拟 17 位，环球以 U 开头）
- `capability` — `CASH`（现金）/ `RegTMargin`（保证金）/ `PMGRN`（组合保证金）
- `status` — `Funded` / `Open` / `Pending` / `Rejected` / `Closed`
- 账户类型 — `STANDARD` / `GLOBAL` / `PAPER`

> 方法名是 `getManagedAccounts`，**没有** `accounts()` 方法。
> The method is `getManagedAccounts`; there is no `accounts()`.

---

## 账户资产 / Account Assets

### 环球账户 / Global Account

```typescript
const assets = await tc.getAssets({
  segment: true,      // 按证券/期货分类
  marketValue: true,  // 按市场分市值（仅环球账户）
});
```

返回 `Asset[]`。主要字段：净清算值、可用资金、购买力、现金、
初始/维持保证金要求、浮动与已实现盈亏、按品种分类的 `segments`。

### 综合/模拟账号 / Standard/Paper Account

```typescript
// 返回 PrimeAsset | undefined，需判空
const prime = await tc.getPrimeAssets({ segment: true });
if (prime) {
  console.log(prime);
}
```

`segments` 主要字段：
- `category` — S（证券）/ C（期货）/ F（基金）/ D（数字货币）
- `capability` — RegTMargin / Cash
- 购买力、可用资金、现金余额、净清算值
- 初始保证金、维持保证金（低于此值会强平）
- 浮动盈亏、按币种（USD/HKD/SGD/CNH）细分

> `AssetsRequest` 只有 `account`、`secretKey`、`subAccounts`、`segment`、
> `marketValue`、`lang` 六个字段，**没有** `baseCurrency` / `consolidated`。

---

## 账户持仓 / Account Positions

```typescript
const positions = await tc.getPositions({
  secType: 'STK',   // STK/OPT/FUT，默认 STK
  currency: 'ALL',  // ALL/USD/HKD/CNH
  market: 'ALL',    // ALL/US/HK/CN
});

for (const p of positions) {
  console.log(p.symbol, p.positionQty, p.averageCost, p.marketValue, p.unrealizedPnl);
}
```

返回 `Position[]`。常用字段：`symbol`、`secType`、`market`、`currency`、
`positionQty`（持仓数量）、`averageCost`（平均成本）、`marketValue`、
`realizedPnl`、`unrealizedPnl`。

### 期权持仓 / Option Positions

```typescript
const optPositions = await tc.getPositions({ secType: 'OPT' });
// 期权持仓额外字段 / Option-specific fields: strike, expiry, right
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```typescript
const analytics = await tc.getAnalyticsAsset({
  startDate: '2026-01-01',   // YYYY-MM-DD
  endDate: '2026-01-31',
  segType: 'SEC',            // SEC / FUT
});
```

> 方法名是 `getAnalyticsAsset`，**不是** `primeAnalyticsAsset`。
> `AnalyticsAssetRequest` 没有 `currency` 字段。

返回内容包含汇总（盈亏金额、收益率、年化收益率）与按日历史
（日期毫秒时间戳、总资产、当日盈亏、现金余额、持仓市值、入金、出金）。

---

## 最大可交易数量 / Estimate Tradable Quantity

```typescript
const qty = await tc.getEstimateTradableQuantity({
  symbol: 'AAPL',
  secType: 'STK',
  action: 'BUY',
  orderType: 'LMT',
  limitPrice: 150.0,
});
```

返回 `EstimateTradableQuantity | undefined`，含现金可买/卖数量、
融资融券可买/卖数量、持仓数量与持仓可交易数量。

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```typescript
const avail = await tc.getSegmentFundAvailable({
  fromSegment: 'SEC',   // SEC / FUT
  currency: 'USD',
});
```

### 发起转账 / Transfer

```typescript
// 返回 SegmentFund | undefined
const transferred = await tc.transferSegmentFund({
  fromSegment: 'SEC',
  toSegment: 'FUT',
  currency: 'USD',
  amount: 1000,
});
// 转账状态 status: NEW / PROC / SUCC / FAIL / CANC
```

> 方法名是 `transferSegmentFund`（动词在前），**不是** `segmentFundTransfer`。

### 撤销转账与历史 / Cancel & History

```typescript
const fundCancelled = await tc.cancelSegmentFund({ id: 'transfer_id' });
const fundHistory = await tc.getSegmentFundHistory({ limit: 20 });
```

> `SegmentFundRequest.id` 是 `string`，不是数字。

---

## 出入金记录 / Funding Records

SDK **没有** `depositWithdraw` 方法，用以下两个接口 / There is no `depositWithdraw`; use:

```typescript
const funding = await tc.getFundingHistory({
  segType: 'SEC',
  currency: 'USD',
  limit: 20,
});

const fundDetails = await tc.getFundDetails({
  segTypes: ['SEC'],
  currency: 'USD',
  startDate: 1767225600000,   // 毫秒时间戳
  endDate: 1769904000000,
  limit: 50,
});
```

> `FundDetailsRequest` / `FundingHistoryRequest` 的 `startDate`/`endDate`
> 是**毫秒时间戳（number）**，不是日期字符串。

---

## 聚合资产与持仓转移 / Aggregate Assets & Position Transfer

```typescript
// 返回 AggregateAssets | undefined
const aggregate = await tc.getAggregateAssets();

// 持仓转移：toAccount 与 transfers 是必填字段
const transferred2 = await tc.transferPosition({
  fromAccount: 'A',
  toAccount: 'B',
  transfers: [{ symbol: 'AAPL', quantity: 100, secType: 'STK' }],
});
```

---

## 直接调用 API / Raw API Call

```typescript
const raw = await httpClient.execute('accounts', JSON.stringify({}));
console.log(raw);
```

---

## 注意事项 / Notes

- 环球账户(Global)用 `getAssets()`，综合/模拟账户(Standard/Paper)用 `getPrimeAssets()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 所有方法都是 `async`；返回强类型对象，不需要 `JSON.parse`
- 单值接口（`getPrimeAssets`、`transferSegmentFund`、`cancelSegmentFund`、
  `getAggregateAssets`、`getEstimateTradableQuantity`、`transferPosition`）
  返回 `T | undefined`，需判空
- 维持保证金低于 0 时会触发强制平仓
- 机构用户在 `new TradeClient(httpClient, account, secretKey)` 传入 secret key，
  或在单个请求上设置 `secretKey` 覆盖
- 从包根路径 `@tigeropenapi/tigeropen` 导入所有 API，不要用子路径
