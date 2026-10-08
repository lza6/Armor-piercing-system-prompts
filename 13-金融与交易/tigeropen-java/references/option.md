
# Tiger Open API Java SDK 期权 / Options Quote

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option-java

## 期权操作工作流 / Option Workflow
<!-- 当用户提到 "期权"、"期权链"、"到期日"、"行权价"、"Greeks"、"Call"、"Put"、"看涨"、"看跌"、"option"、"option chain"、"expiration"、"strike" 时 -->

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `OptionExpirationQueryRequest` → 获取可选到期日列表 / Get available expiration dates
2. **查期权链 Get chain**: `OptionChainQueryRequest` → 获取指定到期日的所有合约，可按 Greeks 筛选 / Get contracts for a given expiry, filter by Greeks
3. **查行情 Get quotes**: `OptionBriefQueryRequest` / `OptionBriefQueryV2Request` → 获取期权实时行情 / Get real-time option quotes

### 期权交易 / Option Trading

1. **构造合约 Build contract**: 使用 `ContractItem` 设置 `secType=OPT`, `symbol`, `expiry`, `strike`, `right` / Set option contract fields
2. **预览订单 Preview**: 预览接口确认保证金和佣金 / Preview for margin and commission
3. **下单 Place order**: 期权数量单位为"张" / Option quantity is in "contracts"

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `OptionSymbolRequest` 查询港股期权代码映射 / Use `OptionSymbolRequest` for HK option symbol mapping

---

## 初始化 / Initialize

```java
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;

ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
clientConfig.tigerId = "your_tiger_id";
clientConfig.defaultAccount = "your_account";
clientConfig.privateKey = "your_private_key";
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(clientConfig);
```

---

## 期权到期日 / Option Expirations

**请求类 Request Class：`OptionExpirationQueryRequest`**

获取指定股票的期权到期日信息，批量请求单次最多30条。
Get option expiration dates for specified stocks. Max 30 per batch request.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbols | array | Yes | 股票代码列表，上限30 / Stock symbol list, max 30 |
| market | Market | Yes | US / HK |

**返回 / Response**

`OptionExpirationResponse` -> `List<OptionExpirationItem>`

访问方式 Access: `response.getOptionExpirationItems()`

```java
public class OptionExpirationResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionExpirationItem> optionExpirationItems;
}
```

**OptionExpirationItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 股票代码 / Stock symbol |
| count | int | 过期日期个数 / Number of expiration dates |
| dates | array | 过期时间，日期格式 / Expiration dates, e.g. `2024-06-28` |
| timestamps | array | 过期日期，时间戳格式 / Expiration timestamps (ms, NewYork timezone) |
| periodTags | array | 期权周期标签 / Period tags: `m`=月期权(monthly), `w`=周期权(weekly), `q`=季度(quarterly) |
| optionSymbols | array | 对应期权四要素的symbol / Option symbols for the four elements |

**示例 / Example**

```java
List<String> symbols = new ArrayList<>();
symbols.add("VIX");
OptionExpirationResponse response = client.execute(
        new OptionExpirationQueryRequest(symbols, Market.US));
// HK market option: market parameter must be Market.HK
// symbols.add("PAI.HK");
// OptionExpirationResponse response = client.execute(
//        new OptionExpirationQueryRequest(symbols, Market.HK));
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
    "code": 0,
    "data": [
        {
            "count": 12,
            "dates": [
                "2024-12-24",
                "2024-12-31",
                "2025-01-08",
                "2025-01-15",
                "2025-01-22",
                "2025-02-19",
                "2025-03-18",
                "2025-04-16",
                "2025-05-21",
                "2025-06-18",
                "2025-07-16",
                "2025-08-20"
            ],
            "optionSymbols": ["VIXW","VIXW","VIXW","VIXW","VIX","VIX","VIX","VIX","VIX","VIX","VIX","VIX"],
            "periodTags": ["w","q","w","w","m","m","m","m","m","m","m","m"],
            "symbol": "VIX",
            "timestamps": [1735016400000,1735621200000,1736312400000,1736917200000,1737522000000,1739941200000,1742270400000,1744776000000,1747800000000,1750219200000,1752638400000,1755662400000]
        }
    ],
    "message": "success",
    "success": true
}
```

> 关于标普500 .SPX 的期权符号: 月度期权符号是 `SPX`, 周期权和季度期权的符号都是 `SPXW`
> About S&P 500 .SPX option symbols: Monthly option symbol is `SPX`, weekly and quarterly option symbols are `SPXW`

---

## 期权链 / Option Chain

**请求类 Request Class：`OptionChainQueryV3Request`**

获取期权链。注意：返回的希腊值并非实时数据，而是上一交易日收盘时的最后更新值。盘中可使用期权指标计算。
Get option chain. Note: Greeks returned are from the last trading day's close, not real-time. Use Option Indicator Calculation for intraday values.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 股票代码 / Stock symbol, symbol+expiry combo max 30 |
| expiry | String | Yes | 期权过期日 / Expiration date, e.g. `2024-07-26` |
| market | Market | Yes | US / HK |

**筛选参数 / Filter Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| implied_volatility | double | No | 隐含波动率 / Implied volatility |
| in_the_money | boolean | No | 是否价内 / In the money |
| open_interest | int | No | 未平仓量 / Open interest |
| delta | double | No | delta |
| gamma | double | No | gamma |
| theta | double | No | theta |
| vega | double | No | vega |
| rho | double | No | rho |

**返回希腊值 / Return Greeks (默认不返回 / default off):**

```java
OptionChainQueryV3Request request = ...;
request.setReturnGreekValue(true);
```

**返回 / Response**

`OptionChainResponse` -> `List<OptionChainItem>`

访问方式 Access: `response.getOptionChainItems()`

```java
public class OptionChainResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionChainItem> optionChainItems;
}
```

**OptionChainItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 标的股票代码 / Underlying symbol |
| expiry | long | 期权过期日 / Expiration date (ms) |
| items | `List<OptionRealTimeQuoteGroup>` | 期权链数据 / Option chain data |

**OptionRealTimeQuoteGroup 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| put | OptionRealTimeQuote | 看跌期权 / Put option |
| call | OptionRealTimeQuote | 看涨期权 / Call option |

**OptionRealTimeQuote 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| identifier | string | 期权标识 / Option identifier, e.g. `AAPL  210115C00095000` |
| strike | double | 行权价 / Strike price |
| right | string | 期权方向 / Direction: PUT/CALL |
| askPrice | double | 卖盘价格 / Ask price |
| askSize | int | 卖盘数量 / Ask size |
| bidPrice | double | 买盘价格 / Bid price |
| bidSize | int | 买盘数量 / Bid size |
| lastTimestamp | long | 最新成交时间 / Last trade timestamp |
| latestPrice | double | 最新价 / Latest price |
| multiplier | double | 乘数 / Multiplier, US options default 100 |
| openInterest | int | 未平仓量 / Open interest |
| preClose | double | 前一交易日收盘价 / Previous close |
| volume | long | 成交量 / Volume |
| impliedVol | double | 隐含波动率 / Implied volatility |
| delta | double | delta |
| gamma | double | gamma |
| theta | double | theta |
| vega | double | vega |
| rho | double | rho |

**示例 / Example**

```java
OptionChainModel basicModel = new OptionChainModel("AAPL", "2024-07-26", TimeZoneId.NewYork);
OptionChainFilterModel filterModel = new OptionChainFilterModel()
  .inTheMoney(true)
  .impliedVolatility(0.1537, 0.8282)
  .openInterest(10, 50000)
  .greeks(new OptionChainFilterModel.Greeks()
    .delta(-0.8, 0.6)
    .gamma(0.024, 0.30)
    .vega(0.019, 0.343)
    .theta(-0.1, 0.1)
    .rho(-0.096, 0.101)
);
OptionChainQueryV3Request request = OptionChainQueryV3Request.of(basicModel, filterModel, Market.US);

OptionChainResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "data": [
    {
      "expiry": 1721966400000,
      "items": [
        {
          "put": {
            "askPrice": 4.65,
            "askSize": 2,
            "bidPrice": 4.5,
            "bidSize": 66,
            "delta": -0.503388,
            "gamma": 0.037062,
            "identifier": "AAPL  240726P00210000",
            "impliedVol": 0.183129,
            "lastTimestamp": 1719345582586,
            "latestPrice": 4.6,
            "multiplier": 100,
            "openInterest": 1858,
            "preClose": 5.26,
            "rho": -0.072819,
            "right": "put",
            "strike": "210.0",
            "theta": -0.060326,
            "vega": 0.24135,
            "volume": 404
          }
        }
      ],
      "symbol": "AAPL"
    }
  ],
  "message": "success",
  "success": true
}
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

**请求类 Request Class：`OptionBriefQueryV2Request`**

获取期权实时行情，批量请求单次最多30条。
Get real-time option quotes. Max 30 per batch request.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| market | Market | Yes | 市场 / Market: US, HK |
| option_basic | `List<OptionCommonModel>` | Yes | 期权四要素列表，最大30 / Option four-element list, max 30 |

**OptionCommonModel 参数 / Fields**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 股票代码 / Stock symbol (HK: use symbol from get_option_symbols) |
| right | string | Yes | 看多或看空 / Direction: CALL/PUT |
| expiry | long | Yes | 到期时间 / Expiry time |
| strike | string | Yes | 行权价 / Strike price (decimals must match option chain: US min 1 decimal, HK stock options 2 decimals, HK index options no decimals) |

**返回 / Response**

`OptionBriefResponse` -> `List<OptionBriefItem>`

访问方式 Access: `response.getOptionBriefItems()`

```java
public class OptionBriefResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionBriefItem> optionBriefItems;
}
```

**OptionBriefItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 股票代码 / Stock symbol |
| strike | string | 行权价 / Strike price |
| bidPrice | double | 买盘价格 / Bid price |
| bidSize | int | 买盘数量 / Bid size |
| askPrice | double | 卖盘价格 / Ask price |
| askSize | int | 卖盘数量 / Ask size |
| latestPrice | double | 最新价格 / Latest price |
| timestamp | long | 最新成交时间 / Last trade timestamp |
| volume | int | 成交量 / Volume |
| high | double | 最高价 / High |
| low | double | 最低价 / Low |
| open | double | 开盘价 / Open |
| preClose | double | 前一交易日收盘价 / Previous close |
| openInterest | int | 未平仓量 / Open interest |
| change | double | 涨跌额 / Change |
| multiplier | int | 乘数 / Multiplier, US options default 100 |
| ratesBonds | double | 一年期美国国债利率 / 1-year US treasury rate, e.g. 0.0078 = 0.78% |
| right | string | 方向 / Direction: PUT/CALL |
| volatility | string | 历史波动率 / Historical volatility |
| expiry | long | 到期时间（毫秒） / Expiry (ms, midnight) |
| midPrice | double | 中间价 / Mid price |
| midTimestamp | long | 中间价时间戳 / Mid price timestamp |
| markPrice | double | 标记价 / Mark price |
| markTimestamp | long | 标记价时间戳 / Mark price timestamp |
| preMarkPrice | double | 昨标记价 / Previous mark price |
| sellingReturn | double | 卖出年化收益 / Annualized selling return |

**示例 / Example**

```java
OptionCommonModel model = new OptionCommonModel();
model.setSymbol("TSLA");
model.setStrike("437.5");
model.setRight("PUT");
model.setExpiry("2026-01-30", TimeZoneId.NewYork);
List<OptionCommonModel> models = new ArrayList<>();
models.add(model);

OptionBriefQueryV2Request request = new OptionBriefQueryV2Request(models, Market.US);
OptionBriefResponse response = client.execute(request);
if (response.isSuccess()) {
   System.out.println(JSONObject.toJSONString(response));
} else {
   System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "message": "success",
  "optionBriefItems": [{
    "identifier": "TSLA  260130P00437500",
    "symbol": "TSLA",
    "strike": "437.5",
    "bidPrice": 17.45,
    "bidSize": 10,
    "askPrice": 17.65,
    "askSize": 10,
    "latestPrice": 17.6,
    "volume": 967,
    "high": 25.35,
    "low": 14.5,
    "open": 25.35,
    "preClose": 25.44,
    "openInterest": 679,
    "change": -7.84,
    "multiplier": 100,
    "right": "put",
    "volatility": "31.10%",
    "expiry": 1769749200000,
    "ratesBonds": 0.035227,
    "midPrice": 17.55,
    "markPrice": 17.6,
    "preMarkPrice": 26.125,
    "sellingReturn": 1.065105,
    "timestamp": 1769028900019
  }],
  "success": true
}
```

---

## 期权深度行情 / Option Depth Quotes

**请求类 Request Class：`OptionDepthQueryRequest`**

获取期权深度行情数据，支持美国和香港市场期权，批量请求单次最多30条。
Get option depth quote data. Supports US and HK markets. Max 30 per batch request.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| market | Market | Yes | 市场 / Market: US, HK |
| option_basic | `List<OptionCommonModel>` | Yes | 期权四要素列表，最大30 / Option four-element list, max 30 |

**OptionCommonModel 参数 / Fields** (同上 / Same as above)

**返回 / Response**

`OptionDepthResponse` -> `List<OptionDepthItem>`

访问方式 Access: `response.getOptionDepthItems()`

```java
public class OptionDepthResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionDepthItem> optionDepthItems;
}
```

返回数据为盘中17个交易所的实时报价。如果报价为0表示该交易所没有报价。
Returns real-time quotes from 17 exchanges during market hours. A price of 0 means no quote from that exchange.

**OptionDepthItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 标的股票代码 / Underlying symbol |
| expiry | long | 到期时间 / Expiry (ms) |
| strike | string | 行权价 / Strike price |
| right | string | PUT / CALL |
| timestamp | long | 数据时间戳 / Data timestamp |
| ask | `List<OptionDepthOrderBook>` | 卖盘挂单数据 / Ask order book |
| bid | `List<OptionDepthOrderBook>` | 买盘挂单数据 / Bid order book |

**OptionDepthOrderBook 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| price | double | 委托价 / Order price |
| code | string | 期权交易所 Code / Exchange code |
| timestamp | long | 交易所时间 / Exchange timestamp |
| volume | int | 委托量 / Order volume |

**示例 / Example**

```java
OptionCommonModel model = new OptionCommonModel();
model.setSymbol("AAPL");
model.setRight("PUT");
model.setStrike("210.0");
model.setExpiry("2024-06-28", TimeZoneId.NewYork);

OptionDepthQueryRequest request = OptionDepthQueryRequest.of(model).market(Market.US);
OptionDepthResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "data": [{
    "ask": [
      {"code": "CBOE", "price": 1.19, "volume": 10, "timestamp": 1718654399000},
      {"code": "BZX", "price": 1.19, "volume": 10, "timestamp": 1718654399000},
      {"code": "AMEX", "price": 1.19, "volume": 2, "timestamp": 1718654400000},
      {"code": "NSDQ", "price": 1.19, "volume": 2, "timestamp": 1718654399000},
      {"code": "PHLX", "price": 1.2, "volume": 54, "timestamp": 1718654399000}
    ],
    "bid": [
      {"code": "PHLX", "price": 1.12, "volume": 48, "timestamp": 1718654399000},
      {"code": "MIAX", "price": 1.12, "volume": 37, "timestamp": 1718654399000},
      {"code": "CBOE", "price": 1.12, "volume": 29, "timestamp": 1718654399000}
    ],
    "expiry": 1719547200000,
    "right": "PUT",
    "strike": "210.0",
    "timestamp": 1718654402000
  }],
  "message": "success",
  "success": true
}
```

---

## 期权逐笔成交 / Option Trade Ticks

**请求类 Request Class：`OptionTradeTickQueryRequest`**

获取期权逐笔成交数据，只支持美国市场期权，批量请求单次最多30条。
Get option tick-by-tick trade data. US market only. Max 30 per batch request.

> 开盘前半小时可以取到前一个交易日的全部，开盘后是新一天的数据。
> In the half hour before market open, you can get the previous trading day's full data. After open, it returns the current day's data.

**参数 / Parameters (OptionCommonModel)**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 股票代码 / Stock symbol |
| right | string | Yes | 看多或看空 / Direction: CALL/PUT |
| expiry | long | Yes | 到期时间（NewYork时间当天0点毫秒值） / Expiry (ms, NY midnight) |
| strike | string | Yes | 行权价 / Strike price (decimals must match option chain) |

**返回 / Response**

`OptionTradeTickResponse` -> `List<OptionTradeTickItem>`

访问方式 Access: `response.getOptionTradeTickItems()`

```java
public class OptionTradeTickResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionTradeTickItem> optionTradeTickItems;
}
```

**OptionTradeTickItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 标的股票代码 / Underlying symbol |
| expiry | long | 到期时间 / Expiry (ms) |
| strike | string | 行权价 / Strike price |
| right | string | PUT / CALL |
| items | `List<TradeTickPoint>` | 逐笔成交数据 / Trade tick list |

**TradeTickPoint 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| price | double | 成交价格 / Trade price |
| time | long | 成交时间 / Trade time (ms) |
| volume | long | 成交量 / Volume |

**示例 / Example**

```java
List<OptionCommonModel> modelList = new ArrayList<>();
OptionCommonModel model1 = new OptionCommonModel();
model1.setSymbol("AAPL");
model1.setRight("PUT");
model1.setStrike("185.0");
model1.setExpiry("2024-03-08", TimeZoneId.NewYork);
modelList.add(model1);

OptionCommonModel model2 = new OptionCommonModel();
model2.setSymbol("AAPL");
model2.setRight("CALL");
model2.setStrike("185.0");
model2.setExpiry("2024-03-08", TimeZoneId.NewYork);
modelList.add(model2);

OptionTradeTickResponse response = client.execute(OptionTradeTickQueryRequest.of(modelList));
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "data": [{
    "expiry": 1709874000000,
    "items": [
      {"price": 2.63, "time": 1708698601086, "volume": 4},
      {"price": 2.62, "time": 1708698602594, "volume": 6},
      {"price": 2.73, "time": 1708698606317, "volume": 4},
      {"price": 2.72, "time": 1708698607576, "volume": 38}
    ],
    "right": "put",
    "strike": "185.0",
    "symbol": "AAPL"
  }, {
    "expiry": 1709874000000,
    "items": [
      {"price": 2.98, "time": 1708698600473, "volume": 1},
      {"price": 2.98, "time": 1708698601051, "volume": 5},
      {"price": 2.99, "time": 1708698601051, "volume": 11}
    ],
    "right": "call",
    "strike": "185.0",
    "symbol": "AAPL"
  }],
  "message": "success",
  "success": true
}
```

---

## 期权K线 / Option K-line (Candlestick)

**请求类 Request Class：`OptionKlineQueryV2Request`**

获取期权K线，批量请求单次最多30条。
Get option K-line (candlestick) data. Max 30 per batch request.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| market | Market | Yes | 市场 / Market: US, HK |
| option_query | `List<OptionKlineModel>` | Yes | 期权K线查询条件列表，最大30 / Query list, max 30 |

**OptionKlineModel 参数 / Fields**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 股票代码 / Stock symbol |
| right | string | Yes | 看多或看空 / Direction: CALL/PUT |
| expiry | long | Yes | 到期时间 / Expiry time |
| strike | string | Yes | 行权价 / Strike price (decimals must match option chain) |
| begin_time | long | Yes | 开始时间 / Begin time (ms) |
| end_time | long | Yes | 结束时间 / End time (ms) |
| period | string | No | K线类型 / Period: `day`, `1min`, `5min`, `30min`, `60min` |
| limit | int | No | 分钟线返回记录数，默认300，最大1200 / Minute bar limit, default 300, max 1200. Day bars not supported |
| sort_dir | string | No | 排序方向 / Sort direction: ASC/DESC |

**返回 / Response**

`OptionKlineResponse` -> `List<OptionKlineItem>`

访问方式 Access: `response.getKlineItems()`

```java
public class OptionKlineResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionKlineItem> klineItems;
}
```

**OptionKlineItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 股票代码 / Stock symbol |
| period | string | 周期类型 / Period type |
| right | string | 看多或看空 / Direction: CALL/PUT |
| strike | string | 行权价 / Strike price |
| expiry | long | 到期时间（毫秒） / Expiry (ms) |
| items | `List<OptionKlinePoint>` | K线数据 / K-line data points |

**OptionKlinePoint 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| high | double | 最高价 / High |
| low | double | 最低价 / Low |
| open | double | 开盘价 / Open |
| close | double | 收盘价 / Close |
| time | long | K线时间 / K-line time (ms) |
| volume | int | 成交量 / Volume |
| openInterest | int | 未平仓量（只有日K线有值） / Open interest (day bars only) |

**示例 / Example**

```java
OptionKlineModel model = new OptionKlineModel();
model.setSymbol("AAPL");
model.setRight("CALL");
model.setStrike("170.0");
model.setExpiry("2024-06-28", TimeZoneId.NewYork);
model.setBeginTime("2024-06-26", TimeZoneId.NewYork);
model.setEndTime("2024-06-26 12:59:59", TimeZoneId.NewYork);
model.setPeriod(OptionKType.min1.getValue());
model.setLimit(10);
model.setSortDir(SortDir.SortDir_Descend);
OptionKlineQueryV2Request request = OptionKlineQueryV2Request.of(model).market(Market.US);

OptionKlineResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
    "code": 0,
    "data": [
        {
            "expiry": 1719547200000,
            "items": [
                {"close": 43.13, "high": 43.13, "low": 43.13, "open": 43.13, "time": 1719419340000, "volume": 0},
                {"close": 43.13, "high": 43.13, "low": 43.13, "open": 43.13, "time": 1719419280000, "volume": 0}
            ],
            "period": "1min",
            "right": "CALL",
            "strike": "170.0",
            "symbol": "AAPL"
        }
    ],
    "message": "success",
    "success": true
}
```

---

## 期权分时数据 / Option Timeline

**请求类 Request Class：`OptionTimelineRequest`**

获取期权的分时数据。
Get option timeline (intraday) data.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| optionTimelineModels | `List<OptionTimelineModel>` | Yes | 期权列表 / Option list |
| market | Market | No | 市场，默认HK，仅支持HK / Market, default HK, HK only |

**OptionTimelineModel 参数 / Fields**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 股票代码 / Stock symbol |
| right | string | Yes | 看多或看空 / Direction: CALL/PUT |
| expiry | long | Yes | 到期时间 / Expiry time |
| strike | string | Yes | 行权价 / Strike price (decimals must match option chain) |

**返回 / Response**

**OptionTimelineItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 股票代码 / Stock symbol |
| right | string | 看多或看空 / Direction: CALL/PUT |
| expiry | long | 到期时间 / Expiry (ms) |
| strike | string | 行权价 / Strike price |
| preClose | double | 昨日收盘价 / Previous close |
| openAndCloseTimeList | `List<List<Long>>` | 交易时间段列表 / Trading session time list |
| minutes | `List<OptionTimelinePoint>` | 分时数组 / Timeline data points |

**OptionTimelinePoint 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| volume | long | 成交量 / Volume |
| avgPrice | double | 平均成交价格 / Average trade price |
| price | double | 最新价格 / Latest price |
| time | long | 当前分时时间 / Current minute timestamp |

**示例 / Example**

```java
OptionTimelineModel model1 = new OptionTimelineModel();
model1.setSymbol("ALB.HK");
model1.setExpiry(1753878054000L);
model1.setStrike("117.50");
model1.setRight("CALL");

OptionTimelineModel model2 = new OptionTimelineModel();
model2.setSymbol("LNI.HK");
model2.setExpiry(1753878054000L);
model2.setStrike("17.00");
model2.setRight("PUT");

OptionTimelineRequest request = OptionTimelineRequest.of(model1, model2);
OptionTimelineResponse response = client.execute(request);
if (response.isSuccess()) {
    System.out.println(JSONObject.toJSONString(response));
} else {
    System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "message": "success",
  "timelineItems": [{
    "symbol": "ALB.HK",
    "expiry": 1753878054000,
    "right": "CALL",
    "strike": "117.50",
    "preClose": 2.72,
    "minutes": [
      {"price": 3.4, "avgPrice": 3.5235946, "time": 1750822140000, "volume": 29},
      {"price": 3.4, "avgPrice": 3.5235946, "time": 1750822200000, "volume": 0}
    ]
  }, {
    "symbol": "LNI.HK",
    "expiry": 1753878054000,
    "right": "PUT",
    "strike": "17.00",
    "preClose": 1.28,
    "minutes": [
      {"price": 1.21, "avgPrice": 1.21, "time": 1750822140000, "volume": 0}
    ]
  }],
  "success": true
}
```

---

## 港股期权代码 / HK Option Symbols

**请求类 Request Class：`OptionSymbolRequest`**

获取港股期权的代码映射，例如 00700 -> TCH.HK。
Get HK option symbol mapping, e.g. 00700 -> TCH.HK.

> 港股期权标的代码与股票代码不同，需先映射。
> HK option underlying symbols differ from stock codes; map first.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| market | Market | Yes | 市场，只支持HK / Market, HK only |
| lang | string | No | 语言 / Language: en_US, zh_CN, zh_TW (default: en_US) |

**返回 / Response**

`OptionSymbolResponse` -> `List<OptionSymbolItem>`

访问方式 Access: `response.getSymbolItems()`

```java
public class OptionSymbolResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionSymbolItem> symbolItems;
}
```

**OptionSymbolItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 期权四要素的symbol / Option symbol |
| name | string | 标的名称 / Underlying name |
| underlyingSymbol | string | 底层资产标的代码 / Underlying asset code |

**示例 / Example**

```java
OptionSymbolRequest request = OptionSymbolRequest.newRequest(Market.HK, Language.en_US);
OptionSymbolResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("response error:" + response.getMessage());
}
```

**返回示例 / Response Example**

```json
{
  "code": 0,
  "data": [
    {"name": "ALC", "symbol": "ALC.HK", "underlyingSymbol": "02600"},
    {"name": "CRG", "symbol": "CRG.HK", "underlyingSymbol": "00390"}
  ],
  "message": "success",
  "success": true
}
```

---

## 期权指标计算 / Option Indicator Calculation

计算所选期权的各类指标（盘中实时计算）。
Calculate various indicators for selected options (real-time intraday calculation).

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| client | object | Yes | SDK HTTP client |
| symbol | string | Yes | 股票代码 / Stock symbol |
| right | string | Yes | 看多或看空 / Direction: CALL/PUT |
| strike | string | Yes | 行权价 / Strike price |
| expiry | string | Yes | 到期时间 / Expiry: `yyyy-MM-dd` format |
| underlyingSymbol | string | No | 底层资产标的，默认为symbol / Underlying symbol, defaults to symbol |

**返回 / Response**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| delta | double | 希腊字母 delta / Greek delta |
| gamma | double | 希腊字母 gamma / Greek gamma |
| theta | double | 希腊字母 theta / Greek theta |
| vega | double | 希腊字母 vega / Greek vega |
| rho | double | 希腊字母 rho / Greek rho |
| insideValue | double | 内在价值 / Intrinsic value |
| timeValue | double | 时间价值 / Time value |
| leverage | double | 杠杆率 / Leverage ratio |
| openInterest | int | 未平仓量 / Open interest |
| historyVolatility | double | 历史波动率（百分比数值） / Historical volatility (percentage) |
| premiumRate | double | 溢价率（百分比数值） / Premium rate (percentage) |
| profitRate | double | 买入盈利率（百分比数值） / Buy profit rate (percentage) |
| volatility | double | 隐含波动率（百分比数值） / Implied volatility (percentage) |

**示例 / Example**

```java
OptionFundamentals optionFundamentals = OptionCalcUtils.getOptionFundamentals(
    client, "BABA", "CALL", "205.0", "2019-11-01");
System.out.println(JSONObject.toJSONString(optionFundamentals));
```

**返回示例 / Response Example**

```json
{
  "delta": 0.8573062699731591,
  "gamma": 0.05151538284065261,
  "historyVolatility": 24.38,
  "insideValue": 4.550000000000011,
  "leverage": 30.695960907449216,
  "openInterest": 35417.0,
  "premiumRate": 0.18619306788885054,
  "profitRate": 47.138051059662665,
  "rho": 1.1107261502375654,
  "theta": -0.17927469728943862,
  "timeValue": 0.32499999999998863,
  "vega": 0.034473845504081974,
  "volatility": 28.62548828125
}
```

> 注意: historyVolatility/premiumRate/profitRate/volatility 为百分比形式。例如 24.38 表示 24.38%。
> Note: historyVolatility/premiumRate/profitRate/volatility are in percentage form. e.g. 24.38 means 24.38%.

---

## 期权分析数据 / Option Analysis

**请求类 Request Class：`OptionAnalysisRequest`**

查询期权分析数据。
Query option analysis data.

**参数 / Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbols | `List<OptionAnalysisModel>` | Yes | 期权分析查询项列表 / Analysis query list |
| market | Market | No | 市场，默认US / Market, default US |

**OptionAnalysisModel 参数 / Fields**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|---------|----------------|-----------------|
| symbol | string | Yes | 标的代码 / Symbol, e.g. `AAPL` |
| period | string | Yes | 周期 / Period, e.g. `52week` (see OptionAnalysisPeriod) |
| requireVolatilityList | Boolean | No | 是否返回波动率列表数据 / Whether to return volatility list (IV/HV time series) |

### OptionAnalysisPeriod 分析周期 / Analysis Periods

| 周期 Period | 值 Value | 说明 Description |
|------------|---------|-----------------|
| `THREE_YEAR` | `3year` | 3年 / 3 years |
| `FIFTY_TWO_WEEK` | `52week` | 52周（1年，默认） / 52 weeks (default) |
| `TWENTY_SIX_WEEK` | `26week` | 26周（6个月） / 26 weeks |
| `THIRTEEN_WEEK` | `13week` | 13周（3个月） / 13 weeks |

**返回 / Response**

`OptionAnalysisResponse` -> `List<OptionAnalysisItem>`

```java
public class OptionAnalysisResponse extends TigerResponse {
  @JSONField(name = "data")
  private List<OptionAnalysisItem> optionAnalysisItems;
}

public class OptionAnalysisItem extends ApiModel {
  private String symbol;
  private Double impliedVol30Days;
  private Double hisVolatility;
  private Double ivHisVRatio;
  private Double callPutRatio;
  private ImpliedVolMetric impliedVolMetric;
  private List<VolatilityItem> volatilityList;
}

public class ImpliedVolMetric implements Serializable {
  private String period;
  private Double percentile;
  private Double rank;
}
```

**OptionAnalysisItem 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| symbol | string | 标的代码 / Symbol |
| impliedVol30Days | double | 标的资产隐含波动率(30天加权) / 30-day implied volatility of underlying. Weighted calculation from option chain IV, reflecting overall 30-day expected volatility. |
| hisVolatility | double | 标的资产历史波动率(30天) / 30-day historical volatility of underlying. Measures actual price deviation over past 30 days. |
| ivHisVRatio | double | 隐含波动率/历史波动率 比值 / IV/HV ratio |
| callPutRatio | double | Call/Put 比值 / Call/Put ratio |
| impliedVolMetric | ImpliedVolMetric | IV指标 / IV metric (see below) |
| volatilityList | `List<VolatilityItem>` | 波动率列表（需 requireVolatilityList=true） / Volatility list (requires requireVolatilityList=true) |

**ImpliedVolMetric 属性 / Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| period | string | 分析周期 / Analysis period |
| percentile | double | IV百分位 / IV Percentile: percentage of trading days in the period with IV lower than current. Range 0%-100%. |
| rank | double | IV排名 / IV Rank: (current IV - min IV) / (max IV - min IV). Range 0-1. |

**VolatilityItem 属性 / Fields** (requireVolatilityList=true 时返回 / returned when requireVolatilityList=true)

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| impliedVol | Double | 隐含波动率 / Implied volatility |
| percentile | Double | IV百分位 / IV percentile |
| rank | Double | IV排名 / IV rank |
| hisVolatility | Double | 历史波动率 / Historical volatility |
| timestamp | Long | 时间戳（毫秒） / Timestamp (ms) |

**示例 / Example**

```java
// 基础查询 / Basic query
List<OptionAnalysisModel> items = new ArrayList<>();
items.add(new OptionAnalysisModel("AAPL", OptionAnalysisPeriod.FIFTY_TWO_WEEK));
items.add(new OptionAnalysisModel("TSLA", OptionAnalysisPeriod.FIFTY_TWO_WEEK));
OptionAnalysisRequest request = OptionAnalysisRequest.newRequest(items, Market.US);
OptionAnalysisResponse response = client.execute(request);

// 返回波动率列表数据 / With volatility list
OptionAnalysisRequest requestWithVolList =
      OptionAnalysisRequest.of("AAPL", OptionAnalysisPeriod.FIFTY_TWO_WEEK, true, Market.US);
OptionAnalysisResponse responseWithVolList = client.execute(requestWithVolList);
```

**返回示例 / Response Example**

```json
[
  {
    "callPutRatio": 0.6,
    "hisVolatility": 0.1967,
    "impliedVol30Days": 0.3071,
    "impliedVolMetric": {
      "percentile": 0.527363184079602,
      "period": "52week",
      "rank": 0.18213875790384876
    },
    "ivHisVRatio": 1.5617,
    "symbol": "AAPL"
  },
  {
    "callPutRatio": 0,
    "hisVolatility": 0.3603,
    "impliedVol30Days": 0.5162,
    "impliedVolMetric": {
      "percentile": 0.08,
      "period": "52week",
      "rank": 0.04194153521422974
    },
    "ivHisVRatio": 1.4328,
    "symbol": "TSLA"
  }
]
```

**requireVolatilityList=true 返回示例:**

```json
[
  {
    "callPutRatio": 0.6,
    "hisVolatility": 0.1967,
    "impliedVol30Days": 0.3071,
    "impliedVolMetric": {"percentile": 0.527, "period": "52week", "rank": 0.182},
    "ivHisVRatio": 1.5617,
    "symbol": "AAPL",
    "volatilityList": [
      {"impliedVol": 0.3012, "percentile": 0.512, "rank": 0.175, "hisVolatility": 0.1923, "timestamp": 1709856000000},
      {"impliedVol": 0.2985, "percentile": 0.498, "rank": 0.168, "hisVolatility": 0.1901, "timestamp": 1709769600000}
    ]
  }
]
```

---

## 期权代码格式 / Option Symbol Format

- 美股 US: `'AAPL  250829C00150000'`
  - 格式: 标的(padding至6位) + `YYMMDD` + `C/P` + 行权价*1000(8位)
  - Format: symbol padded to 6 chars + YYMMDD + C/P + strike*1000 (8 digits)
- 港股 HK: `'TCH.HK 230616C00550000'`
  - 注意使用映射后的代码 / Use mapped symbol from OptionSymbolRequest

---

## 注意事项 / Notes

- 港股期权需先获取代码映射 / HK options require symbol mapping via `OptionSymbolRequest`
- 期权每张合约通常代表100股标的 / Each contract typically = 100 shares
- 期权链返回的希腊值为上一交易日收盘值，盘中请用 `OptionCalcUtils` / Chain Greeks are from previous close; use `OptionCalcUtils` for intraday
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- `OptionCommonModel` 是多个接口共用的参数结构 / `OptionCommonModel` is shared across multiple APIs
- 具体字段可通过对象的 get 方法访问，如 `getSymbol()` / Access fields via getter methods, e.g. `getSymbol()`
- 更多期权知识见 / More: https://docs.itigerup.com/docs/quote-option-java
