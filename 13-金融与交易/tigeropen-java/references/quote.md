
# Tiger Open API Java SDK 行情数据 / Market Data

> 中文 | English — 双语技能，代码示例通用。Bilingual skill with shared code examples.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-stock-java
> SDK GitHub: https://github.com/tigerfintech/openapi-java-sdk

## 初始化 / Initialize

```java
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;

// 方式1: 使用默认配置（读取 tiger_openapi_config.properties）
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(ClientConfig.DEFAULT_CONFIG);

// 方式2: 启动时不自动抢占行情权限 / Disable auto grab on startup
ClientConfig.DEFAULT_CONFIG.isAutoGrabPermission = false;
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(ClientConfig.DEFAULT_CONFIG);
```

> 建议模块级创建一次并复用。Create TigerHttpClient once and reuse.

---

## 行情权限管理 / Quote Permission Management

### 抢占行情权限 / Grab Quote Permission

当同一账号在多台设备同时使用时，行情数据仅在主设备上返回。执行"行情权限抢占"将当前设备设为主设备。
启动时默认执行一次抢占行情权限。

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.GRAB_QUOTE_PERMISSION);
String bizContent = AccountParamBuilder.instance().buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
```

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| name | string | 权限名称 |
| expireAt | long | 过期时间戳，-1 表示无限制 |

**权限名称枚举 / Permission Name Enum**

| name 取值 | 说明 |
|----------|------|
| usQuoteBasic | 美股L1行情权限 |
| usStockQuoteLv2Totalview | 美股L2行情权限 |
| hkStockQuoteLv2 | 大陆地区用户赠送的港股L2权限 |
| hkStockQuoteLv2Global | 非大陆地区用户购买的港股L2权限 |
| usOptionQuote | 美股期权L1行情权限 |
| CBOEFuturesQuoteLv2 | 芝加哥期权交易所L2权限 |
| HKEXFuturesQuoteLv2 | 香港期货交易所L2权限 |
| SGXFuturesQuoteLv2 | 新加坡交易所L2权限 |
| OSEFuturesQuoteLv2 | 大阪交易所L2权限 |

### 获取行情权限列表 / Get Quote Permission List

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.GET_QUOTE_PERMISSION);
String bizContent = AccountParamBuilder.instance().buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
```

### 获取历史行情额度 / Get K-line Quota

```java
KlineQuotaRequest request = KlineQuotaRequest.newRequest(Boolean.TRUE); // true=返回详情
KlineQuotaResponse response = client.execute(request);
// response.getQuotaItems() -> List<QuotaItem>
```

**QuotaItem 字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| used | int | 已使用数量 |
| remain | int | 剩余数量 |
| method | String | api接口名 (kline/future_kline/option_kline) |
| symbolDetails | `List<SymbolDetail>` | 已使用的标的列表 |

### 刷新 Token / Refresh Token

仅香港牌照 TBHK 需要使用 Token。Token 有效期为 15 天，SDK 默认不刷新。

```java
UserTokenRefreshRequest request = new UserTokenRefreshRequest();
UserTokenResponse response = TigerHttpClient.getInstance().execute(request);
```

---

## 市场状态 / Market Status

**请求类：QuoteMarketRequest**

```java
QuoteMarketResponse response = client.execute(QuoteMarketRequest.newRequest(Market.US));
// response.getMarketItems() -> List<MarketItem>
for (MarketItem item : response.getMarketItems()) {
    System.out.println(item.getMarket() + ": " + item.getStatus() + ", openTime=" + item.getOpenTime());
}
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| market | Market | Yes | US 美股，HK 港股，CN A股，ALL 所有 |
| lang | Language | No | zh_CN，zh_TW，en_US，默认：en_US |

**MarketItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| market | string | 市场代码（US/HK/CN） |
| marketStatus | string | 市场状态描述（含节假日信息），如 "Trading"、"Closed Independence Day" |
| status | string | 市场状态枚举：NOT_YET_OPEN/PRE_HOUR_TRADING/TRADING/MIDDLE_CLOSE/POST_HOUR_TRADING/CLOSING/OVERNIGHT_TRADING/EARLY_CLOSED/MARKET_CLOSED |
| openTime | string | 最近开盘时间，格式 MM-dd HH:mm:ss |

---

## 交易日历 / Trading Calendar

**请求类：QuoteTradeCalendarRequest**

```java
QuoteTradeCalendarRequest request = QuoteTradeCalendarRequest.newRequest(Market.US, "2022-06-01", "2022-06-30");
QuoteTradeCalendarResponse response = client.execute(request);
// response.getItems() -> List<TradeCalendar>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| market | string | Yes | US/HK/CN |
| begin_date | string | No | yyyy-MM-dd 格式 |
| end_date | string | No | yyyy-MM-dd 格式 |

**TradeCalendar 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| date | string | 交易日日期 |
| type | string | TRADING(正常交易日) / EARLY_CLOSE(提前休市) |

---

## 股票代码与信息 / Stock Symbols & Info

### 获取股票代码列表 / Get Symbol List

**请求类：QuoteSymbolRequest**

```java
QuoteSymbolResponse response = client.execute(QuoteSymbolRequest.newRequest(Market.US).includeOTC(false));
// response.getSymbols() -> List<String>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| market | string | Yes | US/HK/CN |
| include_otc | boolean | No | 是否包含OTC标的，默认false |
| lang | string | No | zh_CN/zh_TW/en_US，默认 en_US |

### 获取股票代码和名称 / Get Symbol Names

**请求类：QuoteSymbolNameRequest**

```java
QuoteSymbolNameResponse response = client.execute(QuoteSymbolNameRequest.newRequest(Market.US).includeOTC(false));
// response.getSymbolNameItems() -> List<SymbolNameItem>
// SymbolNameItem: symbol, name
```

### 获取股票交易信息 / Get Stock Trade Meta

**请求类：QuoteStockTradeRequest**

```java
List<String> symbols = Arrays.asList("00700", "00810");
QuoteStockTradeResponse response = client.execute(QuoteStockTradeRequest.newRequest(symbols));
// response.getStockTradeItems() -> List<QuoteStockTradeItem>
```

**QuoteStockTradeItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | String | 股票代码 |
| lotSize | Integer | 每手股数 |
| spreadScale | Integer | 报价精度 |
| minTick | Double | 股价最小变动单位 |

### 获取个股详情 / Get Stock Detail

**请求类：TigerHttpRequest(MethodName.STOCK_DETAIL)**

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.STOCK_DETAIL);
List<String> symbols = Arrays.asList("TSLA");
String bizContent = QuoteParamBuilder.instance().symbols(symbols).market(Market.US).buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | `List<String>` | Yes | 股票列表 |
| market | Market | Yes | US/HK |

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| name | string | 公司全称 |
| market | string | 市场 |
| exchange | string | 交易所 |
| secType | string | 证券类型 STK |
| latestPrice | double | 最新价 |
| open | double | 开盘价 |
| high | double | 最高价 |
| low | double | 最低价 |
| preClose | double | 前收盘价 |
| change | double | 涨跌额 |
| amplitude | string | 振幅 |
| volume | long | 成交量 |
| amount | double | 成交金额 |
| shares | long | 总股本 |
| floatShares | long | 流通股本 |
| eps | double | 每股收益 |
| marketStatus | string | 市场状态 |
| tradingStatus | int | 交易状态 0:非交易/1:盘前/2:交易中/3:盘后 |
| halted | double | 是否停牌 0.0=未停牌 |
| hourTrading.volume | long | 盘前/盘后成交量 |
| hourTrading.latestPrice | double | 盘前/盘后最新价 |
| hourTrading.tag | string | Pre-Mkt/Post-Mkt |
| listingDate | long | 上市时间戳(毫秒) |
| timestamp | long | 数据更新时间戳 |

---

## 股票基本面 / Stock Fundamentals

**请求类：QuoteStockFundamentalRequest**

需要财报权限，需联系管理员单独开通。

```java
List<String> symbols = Arrays.asList("TIGR");
QuoteStockFundamentalRequest request = QuoteStockFundamentalRequest.newRequest(symbols, Market.US.name());
QuoteStockFundamentalResponse response = client.execute(request);
// response.getStockFundamentalItems()
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | `List<String>` | Yes | 股票列表 |
| market | String | Yes | US/HK |

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| roe | double | 净资产收益率 |
| roa | double | 资产收益率 |
| pbRate | double | 市净率 |
| psRate | double | 市销率 |
| divideRate | double | 股息率 |
| week52High | double | 52周最高 |
| week52Low | double | 52周最低 |
| ttmEps | double | 每股收益(TTM) |
| lyrEps | double | 每股静态收益(LYR) |
| volumeRatio | double | 量比 |
| turnoverRate | double | 换手率 |
| ttmPeRate | double | 市盈率(TTM) |
| lyrPeRate | double | 市盈率(LYR) |
| marketCap | double | 总市值 |
| floatMarketCap | double | 流通市值 |

---

## 实时行情 / Real-time Quotes

### 股票实时行情 / Stock Real-time Quote

**请求类：QuoteRealTimeQuoteRequest**

每次请求最多支持 50 只股票。需购买相应行情权限。

```java
QuoteRealTimeQuoteResponse response = client.execute(
    QuoteRealTimeQuoteRequest.newRequest(Arrays.asList("AAPL"), true)); // true=含盘前盘后
// response.getRealTimeQuoteItems() -> List<RealTimeQuoteItem>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | array | Yes | 股票代码列表（单次上限50） |
| includeHourTrading | Boolean | No | 是否包含盘前盘后数据，默认false |
| lang | string | No | zh_CN/zh_TW/en_US，默认 en_US |

**RealTimeQuoteItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| open | double | 开盘价 |
| high | double | 最高价 |
| low | double | 最低价 |
| close | double | 收盘价 |
| preClose | double | 前一交易日收盘价 |
| latestPrice | double | 最新价 |
| latestTime | long | 最新成交时间 |
| askPrice | double | 卖盘价 |
| askSize | long | 卖盘数量 |
| bidPrice | double | 买盘价 |
| bidSize | long | 买盘数量 |
| volume | long | 成交量 |
| status | string | 交易状态 (NORMAL/HALTED/DELIST/NEW/ALTER/CIRCUIT_BREAKER/ST) |
| change | double | 涨跌额 |
| changeRate | double | 涨跌幅 |
| amplitude | double | 振幅 |
| hourTrading | object | 盘前盘后数据 |

**hourTrading 对象字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| tag | string | "Pre-Mkt" / "Post-Mkt" |
| latestPrice | double | 最新价 |
| preClose | double | 昨收价 |
| latestTime | string | 最新成交时间(美东时间) |
| volume | long | 成交量 |
| change | double | 涨跌额 |
| changeRate | double | 涨跌幅 |
| timestamp | long | 最新成交时间戳 |

### 股票延迟行情 / Delayed Quote

免费延迟行情接口，无需购买权限，仅支持美股，延迟约15分钟。

**请求类：QuoteDelayRequest**

```java
List<String> symbols = Arrays.asList("AAPL", "TSLA");
QuoteDelayRequest request = QuoteDelayRequest.newRequest(symbols);
QuoteDelayResponse response = client.execute(request);
// response.getQuoteDelayItems() -> List<QuoteDelayItem>
```

**QuoteDelayItem 返回字段**: symbol, close, high, low, open, preClose, time, volume

---

## 深度行情 / Depth Quote (Level 2)

**请求类：QuoteDepthRequest**

获取买卖 N 档挂单数据，每次最多支持 50 只证券。需要 L2 权限。
美股最多40档，港股最多10档。

```java
List<String> symbols = Arrays.asList("DD");
QuoteDepthResponse response = client.execute(QuoteDepthRequest.newRequest(symbols, Market.US.name()));
for (QuoteDepthItem item : response.getQuoteDepthItems()) {
    System.out.println(item.getSymbol());
    System.out.println(item.getAsks()); // 卖盘
    System.out.println(item.getBids()); // 买盘
}
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | array | Yes | 股票代码列表（上限50） |
| market | string | Yes | US/HK/CN/ALL |

**买卖盘字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| price | double | 委托价 |
| volume | long | 委托量 |
| count | int | 委托订单数（仅港股） |

---

## 逐笔成交 / Trade Ticks

**请求类：QuoteTradeTickRequest**

支持收盘后查询当日全量逐笔记录，也支持盘中获取最新实时逐笔数据。

```java
List<String> symbols = Arrays.asList("AAPL");
QuoteTradeTickRequest request = QuoteTradeTickRequest.newRequest(symbols, 0L, 30L, 10);
request.setTradeSession(TradeSession.Regular);
QuoteTradeTickResponse response = client.execute(request);
// response.getTradeTickItems() -> List<TradeTickItem>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | array | Yes | 股票代码列表 |
| trade_session | String | No | PreMarket/Regular/AfterHours，默认 Regular |
| beginIndex | long | Yes | 起始索引，设为 -1 返回最新数据 |
| endIndex | long | Yes | 结束索引，与起始差值不能大于 2000 |
| limit | integer | No | 单次请求返回数量，默认 200 |

**TradeTickItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| beginIndex | long | 实际开始索引 |
| endIndex | long | 实际结束索引 |
| symbol | string | 股票代码 |
| items | List\<TickPoint> | 逐笔数据列表 |

**TickPoint 字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| time | long | 交易时间戳 |
| price | double | 成交价 |
| volume | long | 成交量 |
| type | string | "+"主动买入 "-"主动卖出 "*"中性 |

---

## K线数据 / K-line (Candlestick) Data

**请求类：QuoteKlineRequest**

支持港股、美股K线。每次最多返回 1200 条记录。

```java
List<String> symbols = Arrays.asList("AAPL");
// 日K / Daily
QuoteKlineResponse response = client.execute(
    QuoteKlineRequest.newRequest(symbols, KType.day, "2023-05-16", "2023-05-19")
        .withLimit(1000)
        .withRight(RightOption.br));
// response.getKlineItems() -> List<KlineItem>

// 分钟K / Minute K-line (指定日期)
QuoteKlineResponse response = client.execute(
    QuoteKlineRequest.newRequest(symbols, KType.min1).withDate("20220420"));

// 含基本面数据 / With fundamental data
QuoteKlineResponse response = client.execute(
    QuoteKlineRequest.newRequest(symbols, KType.day, beginTime, endTime)
        .withFundamental(true));
```

**参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| symbols | array | 股票代码列表，上限50（A股上限30） |
| period | string | K线周期：day/week/month/year/1min/3min/5min/15min/30min/60min/120min/240min |
| trade_session | string | PreMarket/Regular/AfterHours/OverNight，默认 Regular |
| right | string | 复权：br(前复权，默认) / nr(不复权) |
| begin_time | long | 开始时间戳(ms) |
| end_time | long | 结束时间戳(ms) |
| date | string | 指定日期(yyyyMMdd)，获取该日分钟K线 |
| limit | integer | 返回数量，默认300，最大1200 |
| withFundamental | boolean | 是否返回市盈率换手率 |
| pageToken | string | 分页token（仅单个symbol + begin_time时有效） |

**KlineItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| period | string | K线周期 |
| nextPageToken | string | 下一页token |
| items | array | KlinePoint 数组 |

**KlinePoint 字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| open | double | 开盘价 |
| close | double | 收盘价 |
| high | double | 最高价 |
| low | double | 最低价 |
| time | long | 时间 |
| volume | long | 成交量 |
| amount | double | 成交额 |
| turnoverRate | double | 换手率（需 withFundamental=true） |
| ttmPe | double | TTM市盈率（需 withFundamental=true） |
| lyrPe | double | 静态市盈率（需 withFundamental=true） |

### PageToken 分页获取大量K线

```java
List<String> symbols = Arrays.asList("AAPL");
QuoteKlineRequest request = QuoteKlineRequest.newRequest(symbols, KType.min1,
    "2022-04-25 00:00:00", "2022-04-28 00:00:00", TimeZoneId.NewYork);
request.withLimit(200);
request.withRight(RightOption.br);

while (true) {
    QuoteKlineResponse response = client.execute(request);
    if (!response.isSuccess() || response.getKlineItems().isEmpty()) break;
    KlineItem klineItem = response.getKlineItems().get(0);
    if (klineItem.getNextPageToken() == null) break;
    TimeUnit.SECONDS.sleep(6); // 频率限制
    request.withPageToken(klineItem.getNextPageToken());
}
```

### PageToken 工具类

```java
List<KlinePoint> list = PageTokenUtil.getKlineByPage("AAPL", KType.min1,
    "2022-04-25 00:00:00", "2022-04-28 00:00:00", TimeZoneId.NewYork,
    RightOption.br, 10, 30, PageTokenUtil.DEFAULT_TIME_INTERVAL);
```

---

## 分时数据 / Timeline (Intraday)

### 当日分时 / Current Day Timeline

**请求类：QuoteTimelineRequest**

```java
QuoteTimelineResponse response = client.execute(
    QuoteTimelineRequest.newRequest(Arrays.asList("AAPL"), 1544129760000L));
// response.getTimelineItems() -> List<TimelineItem>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | array | Yes | 股票代码列表 |
| period | string | Yes | day / day5 |
| trade_session | String | No | PreMarket/Regular/AfterHours |
| begin_time | long | No | 开始时间(ms)，默认返回当天 |

**TimelineItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| period | string | 周期 day/5day |
| preClose | double | 昨日收盘价 |
| intraday | object | 盘中分时数组 |
| preMarket | object | (仅美股) 盘前分时 |
| afterHours | object | (仅美股) 盘后分时 |

**分时数据字段**

| 字段 | 说明 |
|------|------|
| volume | 成交量 |
| avgPrice | 平均成交价格 |
| price | 最新价格 |
| time | 当前分时时间 |

### 历史分时 / History Timeline

**请求类：QuoteHistoryTimelineRequest**

```java
List<String> symbols = Arrays.asList("AAPL");
QuoteHistoryTimelineRequest request = QuoteHistoryTimelineRequest.newRequest(symbols, "20220420");
request.withRight(RightOption.br);
QuoteHistoryTimelineResponse response = client.execute(request);
// response.getTimelineItems() -> List<HistoryTimelineItem>
```

---

## 资金流向 / Capital Flow

### 股票资金流数据 / Capital Flow

**请求类：QuoteCapitalFlowRequest**

```java
// 实时数据
QuoteCapitalFlowRequest request = QuoteCapitalFlowRequest.newRequest("AAPL", Market.US, CapitalPeriod.intraday);
QuoteCapitalFlowResponse response = client.execute(request);
// response.getCapitalFlowItem() -> CapitalFlowItem

// 最近10天数据
QuoteCapitalFlowRequest request = QuoteCapitalFlowRequest.newRequest("00700", Market.HK, CapitalPeriod.day);
request.setLimit(10);
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbol | string | Yes | 股票代码 |
| period | string | Yes | intraday/day/week/month/year/quarter/6month |
| market | string | Yes | US/HK/CN（实时不支持A股） |
| begin_time | long | No | 开始时间(ms) |
| end_time | long | No | 结束时间(ms) |
| limit | integer | No | 默认200，最大1200 |

**CapitalFlowItem 返回字段**: symbol, period, items(数组)

**items 字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| netInflow | double | 净流入金额，负数表示流出 |
| time | string | 时间字符串 |
| timestamp | long | 时间戳 |

### 股票资金分布 / Capital Distribution

**请求类：QuoteCapitalDistributionRequest**

```java
QuoteCapitalDistributionRequest request = QuoteCapitalDistributionRequest.newRequest("00700", Market.HK);
QuoteCapitalDistributionResponse response = client.execute(request);
// response.getCapitalDistributionItem() -> CapitalDistributionItem
```

**CapitalDistributionItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| netInflow | double | 净流入金额 |
| inAll | double | 资金流入总额 |
| inBig | double | 大单流入 |
| inMid | double | 中单流入 |
| inSmall | double | 小单流入 |
| outAll | double | 资金流出总额 |
| outBig | double | 大单流出 |
| outMid | double | 中单流出 |
| outSmall | double | 小单流出 |

---

## 港股经纪商 / HK Broker Data

### 港股经纪商买卖席位 / Broker Seats

**请求类：QuoteStockBrokerRequest**

```java
QuoteStockBrokerRequest request = QuoteStockBrokerRequest.newRequest("00700", 10, Language.zh_CN);
QuoteStockBrokerResponse response = client.execute(request);
// response.getStockBrokerItem() -> StockBrokerItem
```

**StockBrokerItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| bidBroker | array | 买方价格档位数组 (LevelBroker) |
| askBroker | array | 卖方价格档位数组 (LevelBroker) |

**LevelBroker 字段**: level, price, brokerCount, broker(数组: id, name)

### 港股经纪商持仓市值 / Broker Holdings (CCASS)

**请求类：QuoteBrokerHoldRequest**

```java
QuoteBrokerHoldResponse response = client.execute(
    QuoteBrokerHoldRequest.newRequest(Market.HK, 50, 0, "buyAmount", "DESC"));
// response.getBrokerHoldPageItem()
```

**BrokerHoldItem 返回字段**: orgId, orgName, date, sharesHold, marketValue, buyAmount, buyAmount5, buyAmount20, buyAmount60

---

## 期货行情 / Futures Quotes

### 获取期货交易所列表 / Get Future Exchanges

**请求类：FutureExchangeRequest**

```java
FutureExchangeResponse response = client.execute(FutureExchangeRequest.newRequest(SecType.FUT.name()));
// response.getFutureExchangeItems() -> List<FutureExchangeItem>
// FutureExchangeItem: code, name, zoneId
```

### 获取交易所下的可交易合约 / Get Contracts by Exchange

**请求类：FutureContractByExchCodeRequest**

```java
FutureBatchContractResponse response = client.execute(
    FutureContractByExchCodeRequest.newRequest("CME"));
// response.getFutureContractItems() -> List<FutureContractItem>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| exchangeCode | string | Yes | 交易所代码 |
| lang | string | No | zh_CN/en_US |

### 获取指定品种的合约 / Get Contracts by Type

**请求类：FutureContractsRequest**（`FutureContractByTypeRequest` 不存在 / does not exist）

```java
FutureContractsResponse response = client.execute(FutureContractsRequest.newRequest("CL"));
// response.getFutureContractItems()
```

### 获取主力合约 / Get Current/Main Contract

**请求类：FutureContinuousContractRequest**

```java
FutureContractResponse response = client.execute(FutureContinuousContractRequest.newRequest("CL"));
// response.getFutureContractItem() -> FutureContractItem
```

**FutureContractItem 主要字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| type | string | 交易品种 |
| trade | boolean | 是否可交易 |
| continuous | boolean | 是否连续合约 |
| name | string | 合约名称 |
| currency | string | 交易货币 |
| ibCode | string | 交易合约代码(下单用) |
| contractCode | string | 合约代码 (CL1901) |
| contractMonth | string | 交割月份 |
| lastTradingDate | string | 最后交易日 |
| multiplier | double | 合约乘数 |
| exchange | string | 交易所代码 |
| minTick | double | 最小报价单位 |
| firstNoticeDate | string | 第一通知日 |

### 查询主连的历史合约 / History Main Contract

**请求类：FutureHistoryMainContractRequest**

```java
List<String> contractCodes = Arrays.asList("ESmain");
FutureHistoryMainContractRequest request = FutureHistoryMainContractRequest.newRequest(
    contractCodes, "2023-06-01", "2023-10-05", TimeZoneId.NewYork);
FutureHistoryMainContractResponse response = client.execute(request);
```

### 查询期货交易时间 / Get Trading Times

**请求类：FutureTradingDateRequest**

```java
FutureTradingDateResponse response = client.execute(
    FutureTradingDateRequest.newRequest("ES2203", System.currentTimeMillis()));
// response.getFutureTradingDateItem() -> tradingTimes, biddingTimes, timeSection
```

### 期货实时行情 / Futures Real-time Quote

**请求类：FutureRealTimeQuoteRequest**

```java
List<String> contractCodes = Arrays.asList("CL1902");
FutureRealTimeQuoteResponse response = client.execute(FutureRealTimeQuoteRequest.newRequest(contractCodes));
// response.getFutureRealTimeItems() -> List<FutureRealTimeItem>
```

**FutureRealTimeItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| contractCode | string | 合约代码 |
| latestPrice | double | 最新价 |
| latestSize | double | 最新价成交量 |
| latestTime | long | 最新价成交时间 |
| bidPrice | double | 买盘价(一档) |
| bidSize | long | 买盘数量(一档) |
| askPrice | double | 卖盘价(一档) |
| askSize | long | 卖盘数量(一档) |
| volume | long | 当日累计成交手数 |
| openInterest | double | 未平仓合约数量 |
| open | double | 开盘价 |
| high | double | 最高价 |
| low | double | 最低价 |
| settlement | double | 结算价 |
| limitUp | double | 涨停价 |
| limitDown | double | 跌停价 |

### 期货深度行情 / Futures Depth

**请求类：FutureDepthRequest**

```java
FutureDepthRequest request = FutureDepthRequest.newRequest(Collections.singletonList("XWmain"));
FutureDepthResponse response = client.execute(request);
// response.getFutureDepthItems()
// FutureDepthItem: contractCode, contractId, ask(List), bid(List)
// FutureDepthAskBidItem: price(BigDecimal), volume(Long)
```

### 期货逐笔数据 / Futures Ticks

**请求类：FutureTickRequest**

索引每天北京时间凌晨6:00重置为0。

```java
FutureTickResponse response = client.execute(FutureTickRequest.newRequest("CL2209", 10L, 100L, 20));
// response.getFutureTickItems()
// FutureTickBatchItem: contractCode, items(index, price, volume, time)
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| contractCode | string | Yes | 期货合约代码 |
| beginIndex | long | Yes | 起始索引，-1=最新 |
| endIndex | long | Yes | 结束索引，差值大于1000按最大1000条返回 |
| limit | int | No | 默认200，最大1000 |

### 期货K线数据 / Futures K-line

**请求类：FutureKlineRequest**

热门合约近10年日K线，全部合约2017年8月至今分钟数据。

```java
List<String> contractCodes = Arrays.asList("CL1901");
FutureKlineResponse response = client.execute(
    FutureKlineRequest.newRequest(contractCodes, FutureKType.min15,
        1535634249489L, 1538807049489L, 200));
// response.getFutureKlineItems() -> List<FutureKlineBatchItem>
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| contractCodes | array | Yes | 合约代码列表，支持主连(CLmain) |
| period | string | Yes | min/3min/5min/10min/15min/30min/45min/60min/2hour/3hour/4hour/6hour/day/week/month |
| beginTime | long | Yes | 开始时间(包含) |
| endTime | long | Yes | 结束时间(不包含) |
| limit | int | No | 默认200，最大1000 |
| pageToken | string | No | 分页token |

**K线字段**: time, open, close, high, low, volume, openInterest, settlement, lastTime

### 期货K线分页 / Futures K-line PageToken

```java
List<String> contractCodes = Arrays.asList("NGmain");
FutureKlineRequest request = FutureKlineRequest.newRequest(contractCodes, FutureKType.day,
    1650920400000L, 1651870900000L, 3);
while (true) {
    FutureKlineResponse response = client.execute(request);
    if (!response.isSuccess() || response.getFutureKlineItems().isEmpty()) break;
    String nextPageToken = response.getFutureKlineItems().get(0).getNextPageToken();
    if (nextPageToken == null) break;
    TimeUnit.SECONDS.sleep(6);
    request.withPageToken(nextPageToken);
}
```

---

## 基金行情 / Fund Quotes

### 获取基金代码列表 / Get Fund Symbols

**请求类：FundSymbolRequest**

```java
FundSymbolResponse response = client.execute(FundSymbolRequest.newRequest());
// response.getSymbols() -> List<String>
```

### 获取基金合约信息 / Get Fund Contracts

**请求类：FundContractsRequest**

```java
List<String> symbols = Arrays.asList("IE00B11XZ988.USD", "LU0476943708.HKD");
FundContractsRequest request = FundContractsRequest.newRequest(symbols, Language.zh_CN);
FundContractsResponse response = client.execute(request);
// response.getFundContractItems() -> List<FundContractItem>
```

**FundContractItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 基金代码(后缀为货币) |
| name | string | 基金名称 |
| companyName | string | 公司名称 |
| market | string | 市场 |
| secType | string | FUND |
| currency | string | USD/HKD/CNH |
| tradeable | boolean | 是否可交易 |
| subType | string | 子类别 (Fixed Income 等) |
| dividendType | string | 分红类型 (INC/ACC) |
| tigerVault | boolean | 是否为老虎钱袋子 |

### 获取基金最新行情 / Get Fund Quote

**请求类：FundQuoteRequest**

```java
List<String> symbols = Arrays.asList("IE00B11XZ988.USD", "LU0476943708.HKD");
FundQuoteRequest request = FundQuoteRequest.newRequest(symbols);
FundQuoteResponse response = client.execute(request);
// response.getQuoteItems() -> List<FundQuoteItem>
// FundQuoteItem: symbol, close, timestamp
```

### 获取基金历史行情 / Get Fund History Quote

**请求类：FundHistoryQuoteRequest**

```java
List<String> symbols = Arrays.asList("IE00B11XZ988.USD", "LU0476943708.HKD");
FundHistoryQuoteRequest request = FundHistoryQuoteRequest.newRequest(symbols);
request.beginTime(DateUtils.getTimestamp("2023-07-01", TimeZoneId.Shanghai));
request.endTime(DateUtils.getTimestamp("2023-07-26", TimeZoneId.Shanghai));
request.limit(5);
FundHistoryQuoteResponse response = client.execute(request);
// response.getQuoteItems() -> List<FundHistoryQuoteItem>
// FundHistoryQuoteItem: symbol, items(List<FundQuotePoint>)
// FundQuotePoint: nav(净值), time(时间戳)
```

---

## 数字货币行情 / Cryptocurrency Quotes

### 获取代码列表 / Get CC Symbols

**请求类：QuoteSymbolRequest**

```java
QuoteSymbolRequest request = QuoteSymbolRequest.newCcRequest();
QuoteSymbolResponse response = client.execute(request);
// response.getSymbols() -> List<String> e.g. "BTC.USD", "ETH.USD"
```

### 获取实时行情 / CC Real-time Quote

**请求类：QuoteRealTimeQuoteRequest**

```java
List<String> symbols = Arrays.asList("BTC.USD", "ETH.USD");
QuoteRealTimeQuoteRequest request = QuoteRealTimeQuoteRequest.newCcRequest(symbols);
QuoteRealTimeQuoteResponse response = client.execute(request);
// response.getRealTimeQuoteItems()
```

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 数字货币代码 |
| open | double | 开盘价 |
| high | double | 最高价 |
| low | double | 最低价 |
| close | double | 收盘价 |
| preClose | double | 前一交易日收盘价 |
| latestPrice | double | 最新价 |
| latestTime | long | 最新成交时间 |
| volumeDecimal | double | 成交量(完整小数精度) |
| change | double | 涨跌额 |
| changeRate | double | 涨跌幅 |

### 获取K线数据 / CC K-line

**请求类：QuoteKlineRequest**

每次最多返回 1200 条。分钟级: BTC 支持2024年3月27日起。日K及以上: BTC 支持2010年7月13日起。

```java
List<String> symbols = Arrays.asList("BTC.USD", "ETH.USD");
QuoteKlineRequest request = QuoteKlineRequest.newRequest(symbols, KType.min1,
    "2026-01-29", "2026-01-30", TimeZoneId.NewYork)
    .withLimit(200)
    .withSecType(SecType.CC);
QuoteKlineResponse response = client.execute(request);
// response.getKlineItems() -> List<KlineItem>
```

**参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| symbols | list | 代码列表，上限10 |
| period | string | 1min/60min/day/week/month |
| begin_time | long | 区间开始时间 |
| end_time | long | 区间截止时间 |
| limit | int | 默认300，最大1200 |

**CC K线字段**: open, close, high, low, time, volumeDecimal

### 获取分时数据 / CC Timeline

**请求类：QuoteTimelineRequest**

```java
List<String> symbols = Arrays.asList("BTC.USD", "ETH.USD");
QuoteTimelineRequest request = QuoteTimelineRequest.newCcRequest(symbols,
    System.currentTimeMillis() - 60 * 60 * 1000);
QuoteTimelineResponse response = client.execute(request);
```

**分时字段**: avgPrice, price, time, volumeDecimal

---

## 窝轮牛熊证行情 / Warrant & CBBC Quotes

### 轮证筛选器 / Warrant Filter

**请求类：WarrantFilterRequest**

```java
WarrantFilterRequest request = WarrantFilterRequest.newRequest("00700");
request.lang(Language.zh_CN);
request.sortFieldName("expireDate");
request.sortDir(SortDir.SortDir_Descend);
request.warrantType(WarrantType.Bull);
request.issuerName("高盛");
request.strike(300.0, 320.0);
request.pageSize(10);
WarrantFilterResponse response = client.execute(request);
// response.getItem() -> WarrantFilterItem
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbol | string | Yes | 正股代码 |
| page | int | No | 页码，从0开始 |
| page_size | int | No | 每页数量，默认50 |
| sort_field_name | string | No | 排序字段，默认 expireDate |
| sort_dir | SortDir | No | SortDir_Ascend/SortDir_Descend |
| warrant_type | `Set<Integer>` | No | 1:Call 2:Put 3:Bull 4:Bear 0:All |
| issuer_name | string | No | 发行商名称 |
| expire_ym | string | No | 到期日 yyyy-MM |
| state | int | No | 0全部 1正常 2终止交易 3等待上市 |
| strike | `Range<Double>` | No | 行权价区间 |
| effective_leverage | `Range<Double>` | No | 有效杠杆区间 |
| volume | `Range<Long>` | No | 成交量区间 |
| implied_volatility | `Range<Double>` | No | 隐含波动率区间 |

**WarrantItem 返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 标的代码 |
| name | string | 标的名称 |
| type | WarrantType | 1认购 2认沽 3牛证 4熊证 |
| secType | string | WAR(窝轮)/IOPT(牛熊证) |
| entitlementRatio | double | 换股比率 |
| premium | double | 溢价 |
| callPrice | double | 回收价(仅牛熊证) |
| expireDate | string | 到期日 |
| latestPrice | double | 最新价 |
| volume | long | 成交量 |
| strike | double | 行权价 |
| leverageRatio | double | 杠杆比率 |
| effectiveLeverage | double | 有效杠杆 |
| impliedVolatility | double | 隐含波动率 |
| inOutPrice | double | 价内(>0)/价外(<0) |
| delta | double | 对冲值 |

### 获取轮证行情 / Warrant Quote

**请求类：WarrantQuoteRequest**

```java
List<String> symbols = Arrays.asList("68723", "68722");
WarrantQuoteRequest request = WarrantQuoteRequest.newRequest(symbols);
request.lang(Language.zh_CN);
WarrantQuoteResponse response = client.execute(request);
// response.getItem() -> WarrantQuoteItem (contains items list of WarrantQuote)
```

**WarrantQuote 返回字段**: symbol, name, exchange, market, secType, currency, expiry, strike, right, multiplier, lastTradingDate, entitlementRatio, entitlementPrice, minTick, listingDate, callPrice, halted, underlyingSymbol, timestamp, latestPrice, preClose, open, high, low, volume, amount, premium, outstandingRatio, impliedVolatility, inOutPrice, delta, leverageRatio, breakevenPoint

---

## 选股器 / Market Scanner

**请求类：MarketScannerRequest**

通过不同技术指标条件扫描全市场行情，筛选满足投资需求的标的。

```java
// 构建基础指标筛选
List<BaseFilter> baseFilterList = new ArrayList<>();
baseFilterList.add(BaseFilter.builder()
    .fieldName(StockField.StockField_MarketValue)
    .filterMin(1000000000D)
    .filterMax(2000000000D)
    .build());

// 多标签指标
List<String> conceptTagList = Arrays.asList("BK1549", "BK1541", "BK1545", "BK1512");
List<MultiTagsRelationFilter> multiTagsRelationFilter = new ArrayList<>();
multiTagsRelationFilter.add(MultiTagsRelationFilter.builder()
    .fieldName(MultiTagField.MultiTagField_Concept)
    .tagList(conceptTagList)
    .build());

// 执行请求
MarketScannerRequest request = MarketScannerRequest.newRequest(
    Market.HK, baseFilterList, null, null, multiTagsRelationFilter, null, 1, 200, null);
MarketScannerResponse response = client.execute(request);
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| market | enum | Yes | US/SG/HK |
| baseFilterList | `list<BaseFilter>` | No | 基础指标(价格,成交量,市值,涨跌幅,PE,换手率等) |
| accumulateFilterList | `list<AccumulateFilter>` | No | 累积指标(累积涨跌幅,EPS,净利润,ROA等) |
| financialFilterList | `list<FinancialFilter>` | No | 财务指标(毛利,净利率,负债率等,仅支持LTM) |
| multiTagsRelationFilterList | `list<MultiTagsRelationFilter>` | No | 多标签(行业,概念,52周新高,是否ETF等) |
| sortFieldData | object | No | 排序字段(fieldName, fieldType, sortDir) |
| page | int | Yes | 页码(从0开始) |
| pageSize | int | Yes | 每页数量，最大200 |
| cursorId | String | Yes | 游标ID(首页传null) |

### 筛选字段类 / Filter Classes

```java
/** 基础指标过滤器 **/
BaseFilter {
    private StockField fieldName;     // 简单属性枚举
    private Double filterMin;          // 区间下限
    private Double filterMax;          // 区间上限
    private boolean isNoFilter = false; // 是否不筛选
}

/** 累积指标过滤器 **/
AccumulateFilter {
    private AccumulateField fieldName;
    private Double filterMin;
    private Double filterMax;
    private boolean isNoFilter = false;
    private AccumulatePeriod period;   // 时间周期
}

/** 财务指标过滤器 **/
FinancialFilter {
    private FinancialField fieldName;
    private Double filterMin;
    private Double filterMax;
    private boolean isNoFilter = false;
    private FinancialPeriod quarter;   // 财报周期
}

/** 多标签指标过滤器 **/
MultiTagsRelationFilter {
    private MultiTagField fieldName;
    private List<String> tagList;
    private boolean isNoFilter = false;
}
```

### 字段类别 / Field Belong Type

```java
public enum FieldBelongType {
    StockField_Type(1),        // 基础指标
    AccumulateField_Type(2),   // 累积指标
    FinancialField_Type(3),    // 财务指标
    MultiTagField_Type(5);     // 多标签
}
```

### 排序方向 / Sort Direction

```java
public enum SortDir {
    SortDir_No(0),       // 不排序
    SortDir_Ascend(1),   // 升序
    SortDir_Descend(2);  // 降序
}
```

### 按 ETF 类型筛选 / Filter by ETF Type

```java
// ETF 类型标签值:
// package_us_v1_etf_hot (热门关注), package_us_v1_etf_bank (银行ETF),
// package_us_v1_etf_bond (债券ETF), package_us_v1_etf_buffer (缓冲型),
// package_us_v1_etf_index (宽基指数型), package_us_v1_etf_leverage (杠杆&反向型),
// package_us_v1_etf_sector (行业型), package_us_v1_etf_single_stock (单只股票杠杆型),
// package_us_v1_etf_market_cap (Cap型), package_us_v1_etf_thematic (主题类),
// package_us_v1_etf_international (国际市场), package_us_v1_etf_growth (成长&价值型),
// package_us_v1_etf_commodity (大宗商品型), package_us_v1_etf_ark (ARK木头姐),
// package_us_v1_etf_volatility (波动率), package_us_v1_etf_currency (汇率型),
// package_us_v1_etf_alternative (另类投资)

List<String> etfTagList = Arrays.asList("package_us_v1_etf_hot");
List<MultiTagsRelationFilter> multiTagsRelationFilter = new ArrayList<>();
multiTagsRelationFilter.add(MultiTagsRelationFilter.builder()
    .fieldName(MultiTagField.MultiTagField_ETF_TYPE)
    .tagList(etfTagList)
    .build());
```

**返回结构**

```java
MarketScannerResponse -> MarketScannerBatchItem {
    int page;
    int totalPage;
    int totalCount;
    int pageSize;
    String cursorId;           // 用于下一页请求
    List<MarketScannerItem> data;
}

MarketScannerItem {
    Market market;
    String symbol;
    List<MarketIndicatorValue> baseDataList;
    List<MarketIndicatorValue> accumulateDataList;
    List<MarketIndicatorValue> financialDataList;
    List<MarketIndicatorValue> multiTagDataList;
}

MarketIndicatorValue {
    Integer index;
    String name;
    Object value;
}
```

### 获取多标签筛选字段的标签值 / Get Scanner Tag Values

**请求类：MarketScannerTagsRequest**

```java
List<MultiTagField> multiTagFieldList = Arrays.asList(MultiTagField.MultiTagField_Industry);
MarketScannerTagsRequest request = MarketScannerTagsRequest.newRequest(Market.HK, multiTagFieldList);
MarketScannerTagsResponse response = client.execute(request);
// response.getItems() -> List<MarketScannerTagItem>
// MarketScannerTagItem: market, multiTagField, tagList(List<TagValue>)
// TagValue: tag, value
```

---

## 公司行动 / Corporate Actions

> 公司行动接口(拆合股,分红,财报日历)共同占用频率限制: 60 次/min

### 获取拆合股 / Get Stock Splits

**请求类：CorporateSplitRequest**

```java
Date beginDate = Date.from(LocalDate.parse("2023-01-01")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
Date endDate = Date.from(LocalDate.parse("2025-10-02")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
List<String> symbols = Arrays.asList("NVDA");
CorporateSplitRequest request = CorporateSplitRequest.newRequest(symbols, Market.US, beginDate, endDate);
CorporateSplitResponse response = client.execute(request);
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | `List<String>` | Yes | 股票列表 |
| market | Market | Yes | US/HK |
| beginDate | Date | Yes | 开始时间 |
| endDate | Date | Yes | 结束时间 |

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| market | string | 市场 |
| exchange | string | 交易所 |
| actionType | string | SPLIT |
| fromFactor | double | 拆细前因子(1股) |
| toFactor | double | 拆细后因子(10股) |
| ratio | double | 拆细比例 |
| executeDate | LocalDate | 执行日期 |

### 获取分红 / Get Dividends

**请求类：CorporateDividendRequest**

```java
Date beginDate = Date.from(LocalDate.parse("2025-01-01")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
Date endDate = Date.from(LocalDate.parse("2025-10-27")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
List<String> symbols = Arrays.asList("AAPL");
CorporateDividendRequest request = CorporateDividendRequest.newRequest(symbols, Market.US, beginDate, endDate);
CorporateDividendResponse response = client.execute(request);
```

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| actionType | string | DIVIDEND |
| amount | double | 每股分红金额 |
| currency | string | 币种 |
| recordDate | LocalDate | 记录日 |
| announcedDate | LocalDate | 公告日 |
| executeDate | LocalDate | 执行日 |
| payDate | LocalDate | 付款日 |

### 获取财报日历 / Get Earnings Calendar

**请求类：CorporateEarningRequest**

```java
Date beginDate = Date.from(LocalDate.parse("2025-08-01")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
Date endDate = Date.from(LocalDate.parse("2025-08-10")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
CorporateEarningRequest request = CorporateEarningRequest.newRequest(Market.US, beginDate, endDate);
CorporateEarningResponse response = client.execute(request);
```

**参数**: market(Market, Yes), beginDate(Date, Yes), endDate(Date, Yes)

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| actionType | string | EARNING |
| expectedEps | double | 预期EPS |
| actualEps | double | 实际EPS |
| reportTime | string | 盘前/盘后 |
| fiscalQuarterEnding | string | 财季结束时间 |
| executeDate | LocalDate | 执行日 |
| reportDate | LocalDate | 公告日 |

---

### 代码变更 / Symbol Change（SDK 2.6.0+）

**请求类：CorporateSymbolChangeRequest**

```java
List<String> scSymbols = Arrays.asList("META");
Date scBegin = Date.from(LocalDate.parse("2022-01-01")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());
Date scEnd = Date.from(LocalDate.parse("2023-01-01")
    .atStartOfDay(ZoneId.of("America/New_York")).toInstant());

CorporateSymbolChangeResponse scResp = client.execute(
    CorporateSymbolChangeRequest.newRequest(scSymbols, Market.US, scBegin, scEnd));

// 返回按 symbol 分组的 Map，不是扁平列表
Map<String, List<CorporateSymbolChangeItem>> scItems = scResp.getItems();
for (List<CorporateSymbolChangeItem> group : scItems.values()) {
  for (CorporateSymbolChangeItem item : group) {
    System.out.println(item.getOldSymbol() + " -> " + item.getNewSymbol());
  }
}
```

### 退市事件 / Delisting（SDK 2.6.0+）

**请求类：CorporateDelistingRequest**

```java
CorporateDelistingResponse dlResp = client.execute(
    CorporateDelistingRequest.newRequest(Arrays.asList("AAPL"), Market.US, null, null));
Map<String, List<CorporateDelistingItem>> dlItems = dlResp.getItems();
```

### 新股上市 / IPO（SDK 2.6.0+）

**请求类：CorporateIpoRequest**

```java
CorporateIpoResponse ipoResp = client.execute(
    CorporateIpoRequest.newRequest(Arrays.asList("AAPL"), Market.US, null, null));
Map<String, List<CorporateIpoItem>> ipoItems = ipoResp.getItems();
```

三个请求类的签名完全一致：`newRequest(List<String> symbols, Market market, Date beginDate, Date endDate)`，
内部共用 `MethodName.CORPORATE_ACTION`，仅 `actionType` 不同。
All three share one signature and the `CORPORATE_ACTION` method, differing only in action type.

`CorporateActionType` 共 6 个值 / 6 values：`SPLIT`、`DIVIDEND`、`EARNING`、
`SYMBOL_CHANGE`、`DELISTING`、`IPO`（后三个为 2.6.0 新增）。

> 这三个响应的 `getItems()` 返回 **`Map<String, List<XxxItem>>`**（按 symbol 分组），
> 不是 `List`。
> These three return a symbol-keyed `Map`, not a flat `List`.

---

## 财务报告 / Financial Reports

**请求类：FinancialReportRequest**

需要财报权限，需联系管理员单独开通。

```java
List<String> symbols = Arrays.asList("TIGR");
List<String> fields = Arrays.asList(
    FinancialReportField.totalAssets.getField(),
    FinancialReportField.totalAssetTurnover.getField());
FinancialReportRequest request = FinancialReportRequest.newRequest(symbols, fields);
FinancialReportResponse response = client.execute(request);
// response.getFinancialReportItems()
```

**参数**

| 参数 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| symbols | `List<String>` | Yes | 股票列表 |
| fields | `List<String>` | Yes | 财务指标列表，枚举: `FinancialReportField` |

**返回字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| symbol | string | 股票代码 |
| currency | string | 货币单位 |
| field | string | 财务指标名 |
| value | string | 数据值 |
| filingDate | string | 财报发布日 |
| periodEndDate | string | 会计期间结束日 |

---

## 注意事项 / Notes

- 行情权限需单独购买，API 与 App 独立 / Quote permissions require separate purchase
- 多设备同账号：最后抢占的设备获取权限 / Last device to grab gets permissions
- K线有配额限制，大量数据使用 pageToken 分页 / Use pageToken for large datasets
- 港股代码5位数 `00700`(腾讯) / HK codes are 5-digit like `00700`
- 深度行情需 L2 权限 / Depth quotes require Level 2 permission
- 公司行动接口共享频率限制 60 次/min
- SDK 主要包: `com.tigerbrokers.stock.openapi.client`
- 更多详情见 / More details: https://docs.itigerup.com/docs/quote-stock-java
