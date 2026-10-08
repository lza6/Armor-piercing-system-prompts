
# Tiger OpenAPI TypeScript SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

1. **查到期日 Get expirations**: `qc.getOptionExpiration(['AAPL'], 'US')`
2. **查期权链 Get chain**: `qc.getOptionChain([['AAPL', '2026-08-21']])`
3. **查行情 Get quotes**: `qc.getOptionBrief([...])` / `qc.getOptionQuote([...])`

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股：`00700` → `TCH`（腾讯）
- 用 `qc.getOptionSymbols()` 查询港股期权代码映射

---

## 初始化 / Initialize

```typescript
import {
  createClientConfig,
  HttpClient,
  QuoteClient,
  TradeClient,
  limitOrder,
} from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const qc = new QuoteClient(httpClient);
const tc = new TradeClient(httpClient, config.account);
const account = config.account;
```

> **关键约定 / Key conventions**
> - 期权方法以 `getOption` 开头
> - `getOptionExpiration` / `getOptionChain` / `getOptionBrief` / `getOptionQuote` /
>   `getOptionKline` 接收**位置参数**；`getOptionDepth` / `getOptionTradeTicks` /
>   `getOptionTimeline` / `getOptionSymbols` / `getOptionAnalysis` 接收**请求对象**
> - 请求对象里的 `expiry` 是 **`number` 毫秒时间戳**；
>   `getOptionChain` 的元组 `expiry` 是 **`YYYY-MM-DD` 字符串**（SDK 内部转换）

---

## 期权到期日 / Option Expirations

```typescript
// 位置参数：symbol 数组 + 可选 market
const expirations = await qc.getOptionExpiration(['AAPL'], 'US');
for (const e of expirations) {
  console.log(e.symbol, e.dates, e.timestamps, e.periods);
}
```

返回 `OptionExpiration[]`：`symbol`、`optionSymbols`、`dates`（日期字符串）、
`timestamps`（毫秒）、`periods`（`m`=月 / `w`=周 / `q`=季）、`counts`。

---

## 期权链 / Option Chain

```typescript
// items 是 [symbol, expiry] 元组数组；expiry 必须是 YYYY-MM-DD，否则抛错
const chains = await qc.getOptionChain([['AAPL', '2026-08-21']]);

// 带 Greeks 与筛选 / with Greeks and filters
const chainsFiltered = await qc.getOptionChain(
  [['AAPL', '2026-08-21']],
  undefined,   // timezone
  true,        // returnGreekValue
  {
    inTheMoney: true,
    impliedVolatility: { min: 0.15, max: 0.8 },
    openInterest: { min: 100 },
    greeks: { delta: { min: 0.2, max: 0.8 } },
  },
);
```

返回 `OptionChain[]`，含 call/put 合约的 identifier、行权价、买卖价、成交量、
持仓量、隐含波动率与希腊字母（需 `returnGreekValue = true`）。

---

## 港股期权代码映射 / HK Option Symbol Mapping

```typescript
const optSymbols = await qc.getOptionSymbols({
  market: 'HK',
  lang: 'en_US',
});
// 返回 OptionSymbol[]：期权 symbol（如 'TCH.HK'）、标的名称、正股代码
```

---

## 期权实时行情 / Option Brief

```typescript
// 位置参数：identifier 数组
const optBriefs = await qc.getOptionBrief(['AAPL  260821C00150000']);

// getOptionQuote 支持可选 timezone
const optQuotes = await qc.getOptionQuote(['AAPL  260821C00150000'], 'US/Eastern');
```

期权代码格式：标的（6 位，右侧空格填充）+ YYMMDD + C/P + 行权价×1000（8 位）。
两者都返回 `Brief[]`；期权相关字段：`identifier`、`strike`、`right`、`expiry`、
`multiplier`、`openInterest`。

---

## 期权深度行情 / Option Depth

```typescript
// 请求对象；字段名是 optionBasic，expiry 是毫秒时间戳
const optDepth = await qc.getOptionDepth({
  optionBasic: [
    { symbol: 'AAPL', expiry: 1787356800000, right: 'PUT', strike: '210.0' },
  ],
  market: 'US',
});
```

---

## 期权逐笔成交 / Option Trade Ticks

```typescript
// 仅美股期权；注意字段名是 contracts
const optTicks = await qc.getOptionTradeTicks({
  contracts: [
    { symbol: 'AAPL', expiry: 1787356800000, right: 'PUT', strike: '185.0' },
  ],
});
```

---

## 期权 K 线 / Option K-line

```typescript
// 位置参数：identifiers, period, beginTime, endTime, timezone?, limit?, sortDir?
const optKlines = await qc.getOptionKline(
  ['AAPL  260821C00150000'],
  '1min',            // day/1min/5min/30min/60min
  1787270400000,     // beginTime 毫秒（默认 -1）
  1787356800000,     // endTime 毫秒（默认 -1）
  undefined,         // timezone
  10,                // limit
  'DESC',            // sortDir
);
```

---

## 期权分时 / Option Timeline

```typescript
// 目前仅支持港股期权；字段名是 optionQuery
const optTimeline = await qc.getOptionTimeline({
  optionQuery: [
    { symbol: 'ALB.HK', expiry: 1753878054000, right: 'CALL', strike: '117.50' },
  ],
  market: 'HK',
});
```

---

## 期权分析 / Option Analysis

```typescript
const analysis = await qc.getOptionAnalysis({
  symbols: ['AAPL'],
  period: '52week',            // 3year/52week/26week/13week
  requireVolatilityList: true,
  market: 'US',
});
```

返回 `OptionAnalysis[]`，含 30 日隐含波动率、历史波动率、IV/HV 比率、
Call/Put 比率、IV 百分位与排名。

> `OptionAnalysisRequest.symbols` 是 `string[]`；`period` 与
> `requireVolatilityList` 是请求级字段（不是每个 symbol 单独设置）。

---

## 单腿期权下单 / Single-leg Option Order

```typescript
const optOrder = limitOrder(account, 'AAPL', 'OPT', 'BUY', 1, 5.0);
optOrder.expiry = '20260821';   // 下单用 YYYYMMDD 字符串
optOrder.strike = '150.0';
optOrder.right = 'CALL';
optOrder.currency = 'USD';

const optPreview = await tc.previewOrder(optOrder);
const optPlaced = await tc.placeOrder(optOrder);
```

也可直接用 identifier / Or set the standard identifier:

```typescript
const byIdentifier = limitOrder(account, 'AAPL', 'OPT', 'BUY', 1, 5.0);
byIdentifier.identifier = 'AAPL  260821C00150000';
```

> **行情接口**的 `expiry` 是毫秒时间戳（`number`），**下单对象**的 `expiry`
> 是 `YYYYMMDD` 字符串。
> Quote APIs use ms timestamps; the order object uses a `YYYYMMDD` string.

---

## 多腿组合策略 / Multi-leg Combo Strategies

组合单需手工构造 `OrderRequest`（`comboOrder` / `contractLeg` helper 未在包根路径导出），
再交给 `placeOrder`。**没有** `placeComboOrder` 方法。
Build the combo `OrderRequest` by hand and submit via `placeOrder`; there is no
`placeComboOrder`, and the `comboOrder` / `contractLeg` helpers are not root-exported.

```typescript
import type { OrderRequest } from '@tigeropenapi/tigeropen';

// 牛市看涨价差 / Bull Call Spread (VERTICAL)
const spread: OrderRequest = {
  account,
  symbol: '',           // 组合单不使用 symbol
  secType: 'MLEG',
  action: 'BUY',
  orderType: 'LMT',
  totalQuantity: 1,
  limitPrice: 3.0,
  comboType: 'VERTICAL',
  contractLegs: [
    { symbol: 'AAPL', secType: 'OPT', action: 'BUY', ratio: 1,
      expiry: '2026-08-21', strike: '145.0', right: 'CALL' },
    { symbol: 'AAPL', secType: 'OPT', action: 'SELL', ratio: 1,
      expiry: '2026-08-21', strike: '155.0', right: 'CALL' },
  ],
  timeInForce: 'DAY',
};

const spreadResult = await tc.placeOrder(spread);
```

### 组合策略类型 / Combo Strategy Types

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

```typescript
// type: 'Exercise'（提前行权）| 'Expire'（提前放弃行权）
const exSubmitted = await tc.submitOptionExercise({
  account,
  contractId: 123456789,
  type: 'Exercise',
  quantity: 1,
  executingDate: '2026-08-21',
  isForce: false,
});

const exCancelled = await tc.cancelOptionExercise({ account, id: 987654321 });
```

两者返回 `boolean`。`itmRate`（0-10）为 `Expire` 专用。

---

## 查询期权持仓 / Query Option Positions

```typescript
const optPositions = await tc.getPositions({ account, secType: 'OPT' });
```

---

## 注意事项 / Notes

- 所有方法都是 `async`；返回强类型对象，不需要 `JSON.parse`
- **行情请求对象的 `expiry` 是毫秒时间戳（`number`）；下单对象的 `expiry` 是
  `YYYYMMDD` 字符串；`getOptionChain` 元组里的 `expiry` 是 `YYYY-MM-DD` 字符串**
- 各接口的合约字段名不同：`optionBasic`（depth）、`optionQuery`（timeline）、
  `contracts`（tradeTicks）
- **SDK 没有 `getOptionIndicator` 方法**；单合约 Greeks 通过
  `getOptionChain(items, tz, true)` 或 `getOptionBrief` 获取
- **SDK 没有 `placeComboOrder` 方法**；组合单手工构造 `OrderRequest` 后用 `placeOrder`
- 港股期权需先用 `getOptionSymbols()` 获取代码映射
- 期权每张合约通常代表 100 股标的；行权价小数位须和期权链一致
- 期权行情需要期权行情权限
- 从包根路径 `@tigeropenapi/tigeropen` 导入所有 API，不要用子路径
