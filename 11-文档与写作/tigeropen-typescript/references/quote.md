# Tiger OpenAPI TypeScript SDK — Market Data / 行情查询

> TypeScript SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```typescript
import {
  createClientConfig,
  HttpClient,
  QuoteClient,
} from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
const httpClient = new HttpClient(config);
const qc = new QuoteClient(httpClient);
```

> **入参约定 / Request convention**
> - 多数方法接收**请求对象**（`{ symbols: [...] }`），不是裸数组或字符串
> - 少数方法接收位置参数：`getMarketState(market)`、`getTimeline(symbols)`、
>   `getOptionExpiration(symbols, market?)`、`getOptionChain(items, ...)`、`getOptionKline(...)`
> - 返回**强类型对象**，无需手动 `JSON.parse`
> - Most methods take a **request object**; a few take positional args. Returns are typed.

---

## 市场状态 / Market State

```typescript
// 位置参数 / positional arg
const states = await qc.getMarketState('US');   // US / HK / CN / SG
for (const s of states) {
  console.log(s.market, s.marketStatus, s.status, s.openTime);
}
```

返回 `MarketState[]`：`market`、`marketStatus`、`status`、`openTime`。

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```typescript
// 请求对象，不是数组 / request object, NOT an array
const briefs = await qc.getRealTimeQuote({ symbols: ['AAPL', 'TSLA'] });
for (const b of briefs) {
  console.log(b.symbol, b.latestPrice, b.bidPrice, b.askPrice, b.volume, b.changeRate);
}
```

返回 `Brief[]`。常用字段：`symbol`、`latestPrice`、`latestTime`、
`open`/`high`/`low`/`close`、`preClose`、`askPrice`/`askSize`、`bidPrice`/`bidSize`、
`volume`、`change`、`changeRate`、`status`。除 `symbol` 外均为可选字段。

`getBrief` 与 `getRealTimeQuote` 入参、返回一致 / `getBrief` is an equivalent alias.

`BriefRequest` 可选字段：`includeHourTrading`、`secType`、`lang`。

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```typescript
// period: day/week/month/year/1min/5min/15min/30min/60min
const klines = await qc.getKline({
  symbols: ['AAPL'],
  period: 'day',
  limit: 30,
});
for (const k of klines) {
  for (const it of k.items) {
    console.log(k.symbol, it.time, it.open, it.high, it.low, it.close, it.volume);
  }
}
```

返回 `Kline[]`（`symbol`、`period`、`nextPageToken`、`items`）；
`KlineItem`：`time`、`open`、`high`、`low`、`close`、`volume`。

`KlineRequest` 支持时间范围（`beginTime`/`endTime`，毫秒）或分页
（`beginIndex`/`endIndex`、`pageToken`），另有 `right`、`tradeSession`、`date`、
`withFundamental`、`secType`、`lang`。分页另有 `getKlineByPage`。

---

## 分时 / Timeline

```typescript
// 位置参数：字符串数组 / positional: array of symbols
const timelines = await qc.getTimeline(['AAPL', 'TSLA']);
for (const t of timelines) {
  console.log(t.symbol, t.period, t.preClose);
  if (t.intraday) {
    console.log('intraday points:', t.intraday.items.length);
  }
}
```

返回 `Timeline[]`。分时按时段分桶：`intraday`、`preHours`、`afterHours`，均为可选，
取值前需判空。历史分时用 `getTimelineHistory`。

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```typescript
const depths = await qc.getQuoteDepth({ symbols: ['AAPL'] });
for (const d of depths) {
  for (const a of d.asks) console.log('ASK', a.price, a.volume);
  for (const b of d.bids) console.log('BID', b.price, b.volume);
}
```

返回 `Depth[]`（`symbol`、`asks`、`bids`）。

---

## 逐笔成交 / Trade Ticks

```typescript
const ticks = await qc.getTradeTick({ symbols: ['AAPL'], limit: 50 });
for (const t of ticks) {
  for (const it of t.items) console.log(it.time, it.price, it.volume);
}
```

---

## 期权 / Options

```typescript
// 到期日：位置参数（symbols 数组 + 可选 market）
const expirations = await qc.getOptionExpiration(['AAPL'], 'US');

// 期权链：位置参数，items 是 [symbol, expiry] 元组数组，expiry 必须是 YYYY-MM-DD
const chains = await qc.getOptionChain([['AAPL', '2026-08-21']]);

// 带 Greeks / with Greeks
const chainsWithGreeks = await qc.getOptionChain(
  [['AAPL', '2026-08-21']],
  undefined,  // timezone
  true,       // returnGreekValue
);

// 期权行情：位置参数，identifier 数组
const optBriefs = await qc.getOptionBrief(['AAPL  260821C00150000']);
```

详见 [option.md](option.md) / See option.md for the full option API.

---

## 期货 / Futures

```typescript
const exchanges = await qc.getFutureExchange();
const futureContracts = await qc.getFutureContracts('CME');   // 位置参数
const futureQuotes = await qc.getFutureRealTimeQuote({ contractCodes: ['CLmain'] });
const futureKlines = await qc.getFutureKline({
  contractCodes: ['CLmain'],
  period: 'day',
});
```

另有 `getFutureDepth`、`getFutureTradeTicks`、`getFutureTradingTimes`、
`getFutureContinuousContracts`、`getCurrentFutureContract`、`getAllFutureContracts`、
`getFutureKlineByPage`、`getFutureHistoryMainContract`。

---

## 资金流 / Capital Flow

```typescript
// 位置参数；返回值可能是 undefined
const flow = await qc.getCapitalFlow('AAPL', 'US', 'day');
const dist = await qc.getCapitalDistribution('AAPL', 'US');
if (flow) console.log(flow);
if (dist) console.log(dist);
```

---

## 公司行为 / Corporate Actions

```typescript
// actionType 由方法内部设置，请求里不要传
const symbolChanges = await qc.getCorporateSymbolChange({
  symbols: ['AAPL'],
  market: 'US',
});
const delistings = await qc.getCorporateDelisting({ symbols: [], market: 'US' });
const ipos = await qc.getCorporateIPO({ symbols: [], market: 'US' });
```

拆股/派息/财报日历分别用 `getCorporateSplit`、`getCorporateDividend`、
`getCorporateEarningsCalendar`（这些需要传 `actionType`）。

---

## 其他行情能力 / Other Quote APIs

```typescript
const symbols = await qc.getSymbols({ market: 'US' });
const perms = await qc.getQuotePermission({});
const grabbed = await qc.grabQuotePermission();
const scan = await qc.marketScanner({ market: 'US' });
```

还封装了基金（`getFundSymbols` / `getFundContracts` / `getFundQuote` /
`getFundHistoryQuote`）、窝轮（`getWarrantBriefs` / `getWarrantFilter`）、
行业（`getIndustryList` / `getIndustryStocks`）、财务（`getFinancialDaily` /
`getFinancialReport` / `getFinancialCurrency` / `getFinancialExchangeRate`）、
股票细节（`getStockDetails` / `getStockBroker` / `getStockFundamental` /
`getShortInterest` / `getTradeRank`）、夜盘（`getQuoteOvernight`）、
交易日历（`getTradingCalendar`）与 K 线额度（`getKlineQuota`）。

---

## Token 管理 / Token Management

```typescript
const token = await qc.queryToken();
await qc.refreshToken();
```

---

## 直接调用 API / Raw API Call

```typescript
const raw = await httpClient.execute(
  'quote_real_time',
  JSON.stringify({ symbols: ['AAPL', 'TSLA'] }),
);
console.log(raw);
```

第二个参数是 **JSON 字符串**，返回 **string**。
Second arg is a **JSON string**; returns a **string**.

---

## 注意事项 / Notes

- 大多数方法接收**请求对象**；`getMarketState`、`getTimeline`、`getOptionExpiration`、
  `getOptionChain`、`getOptionKline`、`getFutureContracts`、`getCapitalFlow` 接收位置参数
- 返回强类型对象，不需要 `JSON.parse`；只有 `httpClient.execute()` 返回字符串
- `Brief` 上除 `symbol` 外的字段都是可选的，读取前注意判空
- `Brief.expiry` 在 TypeScript SDK 中是 `string`
- 分时数据分 `intraday`/`preHours`/`afterHours` 三桶，均为可选
- `getCapitalFlow` / `getCapitalDistribution` / `getStockBroker` / `marketScanner`
  可能返回 `undefined`
- 从包根路径 `@tigeropenapi/tigeropen` 导入所有 API，不要用子路径
- 行情数据需要对应市场的行情权限；期权行情需要单独开通期权行情权限
