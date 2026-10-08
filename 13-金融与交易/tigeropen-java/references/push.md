
# Tiger Open API Java SDK 实时推送 / Real-time Push

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/push-quote-java

## 创建推送客户端 / Create Push Client

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.socket.WebSocketClient;

ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
// 从开发者信息页面导出的配置文件存放路径
clientConfig.configFilePath = "/data/tiger_config";
// clientConfig.secretKey = "xxxxxx"; // institutional trader private key

// 也可手动配置（不使用 properties 文件时，必须配置以下三项）
// clientConfig.tigerId = "your tiger id";
// clientConfig.defaultAccount = "your account";
// clientConfig.privateKey = ConfigUtil.readPrivateKey("/path/to/rsa_private_key_pkcs8.pem");

WebSocketClient client = WebSocketClient.getInstance()
    .clientConfig(clientConfig)
    .apiComposeCallback(new DefaultApiComposeCallback());
```

> `ApiComposeCallback` 是所有推送回调的统一接口，需要实现该接口来接收各种推送数据。
> `ApiComposeCallback` is the unified callback interface for all push data. Implement it to receive push events.

---

## 回调接口 / Callback Interface (ApiComposeCallback)

所有推送回调通过实现 `ApiComposeCallback` 接口来接收。以下为完整回调类示例：
All push callbacks are received by implementing the `ApiComposeCallback` interface. Full example:

```java
package com.tigerbrokers.stock.openapi.demo;

import com.tigerbrokers.stock.openapi.client.socket.ApiComposeCallback;
import com.tigerbrokers.stock.openapi.client.socket.data.TradeTick;
import com.tigerbrokers.stock.openapi.client.socket.data.pb.*;
import com.tigerbrokers.stock.openapi.client.struct.SubscribedSymbol;
import com.tigerbrokers.stock.openapi.client.util.ApiLogger;
import com.tigerbrokers.stock.openapi.client.util.ProtoMessageUtil;

public class DefaultApiComposeCallback implements ApiComposeCallback {

  /* 股票基本行情回调 Stock quote change */
  @Override
  public void quoteChange(QuoteBasicData data) {
    ApiLogger.info("quoteChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 股票最优买卖价行情回调 Stock BBO change */
  @Override
  public void quoteAskBidChange(QuoteBBOData data) {
    ApiLogger.info("quoteAskBidChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 期权行情回调 Option quote change */
  @Override
  public void optionChange(QuoteBasicData data) {
    ApiLogger.info("optionChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 期权最优买卖价行情回调 Option BBO change */
  @Override
  public void optionAskBidChange(QuoteBBOData data) {
    ApiLogger.info("optionAskBidChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 期货行情回调 Future quote change */
  @Override
  public void futureChange(QuoteBasicData data) {
    ApiLogger.info("futureChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 期货最优买卖价行情回调 Future BBO change */
  @Override
  public void futureAskBidChange(QuoteBBOData data) {
    ApiLogger.info("futureAskBidChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 数字货币行情回调 Crypto quote change */
  @Override
  public void ccChange(QuoteBasicData data) {
    ApiLogger.info("ccChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 数字货币最优买卖价行情回调 Crypto BBO change */
  @Override
  public void ccAskBidChange(QuoteBBOData data) {
    ApiLogger.info("ccAskBidChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 深度行情回调 Depth quote change */
  @Override
  public void depthQuoteChange(QuoteDepthData data) {
    ApiLogger.info("depthQuoteChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 逐笔成交行情回调 Trade tick change (snapshot) */
  @Override
  public void tradeTickChange(TradeTick data) {
    // TradeTick 是普通 SDK 模型（socket.data 包），不是 protobuf Message，
    // 不能传给 ProtoMessageUtil.toJson()
    // TradeTick is a plain SDK model, not a protobuf Message
    ApiLogger.info("tradeTickChange:" + JSONObject.toJSONString(data));
  }

  /* 全量逐笔成交行情回调 Full tick change */
  @Override
  public void fullTickChange(TickData data) {
    ApiLogger.info("fullTickChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 分钟K线数据回调 Minute K-line change */
  @Override
  public void klineChange(KlineData data) {
    ApiLogger.info("klineChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 股票行情榜单数据推送 Stock top ranking push */
  @Override
  public void stockTopPush(StockTopData data) {
    ApiLogger.info("stockTopPush, market:" + data.getMarket());
  }

  /* 期权行情榜单数据推送 Option top ranking push */
  @Override
  public void optionTopPush(OptionTopData data) {
    ApiLogger.info("optionTopPush, market:" + data.getMarket());
  }

  /* 订单变动回调 Order status change */
  @Override
  public void orderStatusChange(OrderStatusData data) {
    ApiLogger.info("orderStatusChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 订单执行明细回调 Order transaction change */
  @Override
  public void orderTransactionChange(OrderTransactionData data) {
    ApiLogger.info("orderTransactionChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 持仓变动回调 Position change */
  @Override
  public void positionChange(PositionData data) {
    ApiLogger.info("positionChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 资产变动回调 Asset change */
  @Override
  public void assetChange(AssetData data) {
    ApiLogger.info("assetChange:" + ProtoMessageUtil.toJson(data));
  }

  /* 订阅成功回调 Subscribe end */
  @Override
  public void subscribeEnd(int id, String subject, String result) {
    ApiLogger.info("subscribe " + subject + " end. id:" + id + ", " + result);
  }

  /* 取消订阅回调 Cancel subscribe end */
  @Override
  public void cancelSubscribeEnd(int id, String subject, String result) {
    ApiLogger.info("cancel subscribe " + subject + " end. id:" + id + ", " + result);
  }

  /* 查询已订阅 symbol 回调 Get subscribed symbols end */
  @Override
  public void getSubscribedSymbolEnd(SubscribedSymbol subscribedSymbol) {
    ApiLogger.info("getSubscribedSymbolEnd:" + subscribedSymbol);
  }

  /* 连接成功回调 Connection ack（接口有两个重载，都必须实现）*/
  @Override
  public void connectionAck() {
    System.out.println("connect ack.");
  }

  /* 带心跳间隔的连接成功回调 / Connection ack with heartbeat intervals */
  @Override
  public void connectionAck(int serverSendInterval, int serverReceiveInterval) {
    System.out.println("connect ack. send=" + serverSendInterval
        + ", receive=" + serverReceiveInterval);
  }

  /* 连接已关闭回调 Connection closed */
  @Override
  public void connectionClosed() {
    System.out.println("connection closed.");
  }

  /* 连接被踢出回调 Connection kicked out */
  @Override
  public void connectionKickout(int errorCode, String errorMsg) {
    System.out.println(errorMsg + " and the connection is closed.");
  }

  /* 心跳回调 Heartbeat */
  @Override
  public void hearBeat(String s) {}

  /* 服务器心跳超时 Server heartbeat timeout */
  @Override
  public void serverHeartBeatTimeOut(String s) {}

  /* 异常回调 Error */
  @Override
  public void error(String errorMsg) {
    System.out.println("receive error:" + errorMsg);
  }

  /* 异常回调(带ID) Error with id */
  @Override
  public void error(int id, int errorCode, String errorMsg) {
    System.out.println("receive error id:" + id + ",errorCode:" + errorCode + ",errorMsg:" + errorMsg);
  }
}
```

---

## 连接与断开 / Connect & Disconnect

```java
// 建立连接 / Connect
client.connect();

// 断开连接（会自动注销所有订阅） / Disconnect (auto-unsubscribes all)
client.disconnect();
```

---

## 连接事件回调 / Connection Event Callbacks

长连接建立或断开时的回调。
Callbacks when WebSocket connection is established or disconnected.

**回调接口 / Callback methods:**

```java
void connectionAck()                                    // 连接成功 Connected
void connectionAck(int sendInterval, int receiveInterval) // 连接成功（含心跳间隔）Connected with heartbeat intervals
void connectionClosed()                                 // 连接已关闭 Connection closed
void connectionKickout(int errorCode, String errorMsg)  // 被另一个连接踢掉 Kicked out by another connection
void hearBeat(String s)                                 // 心跳回调 Heartbeat
void serverHeartBeatTimeOut(String s)                   // 服务器心跳超时 Server heartbeat timeout
```

**异常回调 / Error callbacks:**

```java
void error(String errorMsg)                         // 订阅异常 Subscription error
void error(int id, int errorCode, String errorMsg)  // 订阅异常(带ID) Subscription error with id
```

---

## 订阅股票行情 / Subscribe Stock Quotes

`subscribeQuote(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeQuote(Set<String> symbols)`

**说明 / Description:**
订阅股票行情数据，实时获取行情变化信息。回调返回基本行情 `QuoteBasicData` 和最优报价 `QuoteBBOData` 两种类型。
Subscribe to stock quote data for real-time market changes. Callbacks return `QuoteBasicData` (basic quote) and `QuoteBBOData` (best bid/offer) separately.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票代码列表，如 `AAPL`, `00700` |

**返回值 / Return:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | string | SDK 订阅请求时本地生成的 ID，在 `subscribeEnd(int id, String subject, String result)` 回调方法中返回 |

**回调接口 / Callbacks:**
- `void quoteChange(QuoteBasicData data)` — 基本行情回调 Basic quote
- `void quoteAskBidChange(QuoteBBOData data)` — 最优买卖价回调 BBO quote

**示例 / Example:**

```java
client.connect();
Set<String> symbols = new HashSet<>();
symbols.add("AAPL");
symbols.add("SPY");

// 订阅行情 / Subscribe quotes
client.subscribeQuote(symbols);

// 订阅深度 / Subscribe depth
client.subscribeDepthQuote(symbols);

// 查询订阅详情 / Query subscriptions
client.getSubscribedSymbols();

// 等待接收数据 / Wait to receive data
TimeUnit.SECONDS.sleep(60000);

// 取消订阅 / Cancel subscription
client.cancelSubscribeQuote(symbols);
client.cancelSubscribeDepthQuote(symbols);

// 取消全部标的（股票、期权、期货）的行情订阅 / Cancel all quote subscriptions
client.cancelSubscribeQuote(new HashSet<>());
// 取消全部深度行情订阅 / Cancel all depth subscriptions
client.cancelSubscribeDepthQuote(new HashSet<>());
// 取消全部逐笔行情订阅 / Cancel all trade tick subscriptions
client.cancelSubscribeTradeTick(new HashSet<>());
```

**返回数据示例 / Response examples:**

QuoteBasicData (港股/HK):
```json
{
    "symbol": "00700",
    "type": "BASIC",
    "timestamp": "1684721758123",
    "serverTimestamp": "1684721758228",
    "avgPrice": 330.493,
    "latestPrice": 332.2,
    "latestPriceTimestamp": "1684721758103",
    "latestTime": "05-22 10:15:58",
    "preClose": 333.2,
    "volume": "4400026",
    "amount": 1454057948,
    "open": 331.8,
    "high": 334.2,
    "low": 328.2,
    "marketStatus": "Trading",
    "mi": {"p": 332.2, "a": 330.493, "t": "1684721700000", "v": "75400", "o": 331.6, "h": 332.2, "l": 331.4}
}
```

QuoteBBOData (港股/HK):
```json
{
    "symbol": "00700",
    "type": "BBO",
    "timestamp": "1684721757927",
    "askPrice": 332.2,
    "askSize": "32100",
    "askTimestamp": "1684721757344",
    "bidPrice": 332,
    "bidSize": "3500",
    "bidTimestamp": "1684721757773"
}
```

QuoteBasicData (美股/US):
```json
{
    "symbol": "AAPL",
    "type": "BASIC",
    "timestamp": "1684766012120",
    "serverTimestamp": "1684766012129",
    "avgPrice": 174.1721,
    "latestPrice": 174.175,
    "latestPriceTimestamp": "1684766011918",
    "latestTime": "05-22 10:33:31 EDT",
    "preClose": 175.16,
    "volume": "12314802",
    "amount": 2144365591.41,
    "open": 173.98,
    "high": 174.71,
    "low": 173.45,
    "marketStatus": "Trading",
    "mi": {"p": 174.175, "a": 174.1721, "t": "1684765980000", "v": "57641", "o": 174.21, "h": 174.22, "l": 174.14}
}
```

QuoteBasicData (美股盘前/US PreMarket):
> 美股盘前盘后的字段和盘中不一样。PreMarket/AfterHours fields differ from regular hours.

```json
{
    "symbol": "AAPL",
    "type": "BASIC",
    "timestamp": "1684753559744",
    "serverTimestamp": "1684753559752",
    "latestPrice": 173.66,
    "latestPriceTimestamp": "1684753559744",
    "latestTime": "07:05 EDT",
    "preClose": 175.16,
    "volume": "366849",
    "amount": 63731858.18,
    "hourTradingTag": "PreMarket",
    "mi": {"p": 173.66, "a": 173.72891, "t": "1684753500000", "v": "1270", "o": 173.63, "h": 173.67, "l": 173.63}
}
```

---

## 订阅期权行情 / Subscribe Option Quotes

`subscribeOption(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeOption(Set<String> symbols)`

**说明 / Description:**
订阅期权行情（支持美国和香港市场期权）。
Subscribe option quotes (US and HK markets supported).

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 期权代码列表 |

期权 symbol 支持2种格式 / Two symbol formats supported:
- `AAPL 20190329 182.5 PUT` — symbol + 到期日 + 行权价 + 方向，空格分隔
- `SPY   190508C00290000` — identifier 格式 (OCC format)

**回调接口 / Callbacks:**
- `void optionChange(QuoteBasicData data)` — 期权行情回调 Option quote
- `void optionAskBidChange(QuoteBBOData data)` — 期权最优买卖价回调 Option BBO

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
// 两种订阅方式 / Two subscription formats
symbols.add("AAPL 20230317 150.0 CALL");
symbols.add("ALB.HK 20250730 117.50 CALL");
symbols.add("SPY   190508C00290000");

client.subscribeOption(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeOption(symbols);
```

**返回数据示例 / Response example:**
```json
{
    "symbol": "AAPL 20230317 150.0 CALL",
    "type": "BASIC",
    "timestamp": "1676994444927",
    "latestPrice": 4.83,
    "preClose": 6.21,
    "volume": "3181",
    "amount": 939117.006,
    "open": 4.85,
    "high": 5.6,
    "low": 4.64,
    "identifier": "AAPL  230317C00150000",
    "openInt": "82677"
}
```

---

## 订阅期货行情 / Subscribe Future Quotes

`subscribeFuture(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeFuture(Set<String> symbols)`

**说明 / Description:**
订阅期货行情数据。
Subscribe to future quote data.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 期货代码列表，如 `ESmain`, `ES2306` |

**回调接口 / Callbacks:**
- `void futureChange(QuoteBasicData data)` — 期货行情回调 Future quote
- `void futureAskBidChange(QuoteBBOData data)` — 期货最优买卖价回调 Future BBO

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("ESmain");
symbols.add("ES2306");

client.subscribeFuture(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeFuture(symbols);
```

**返回数据示例 / Response example:**
```json
{
    "symbol": "ESmain",
    "type": "BASIC",
    "timestamp": "1684766824130",
    "avgPrice": 4206.476,
    "latestPrice": 4202.5,
    "preClose": 4204.75,
    "volume": "557570",
    "open": 4189,
    "high": 4221.75,
    "low": 4186.5,
    "marketStatus": "Trading",
    "tradeTime": "1684766824000",
    "preSettlement": 4204.75,
    "minTick": 0.25,
    "mi": {"p": 4202.25, "a": 4206.476, "t": "1684766820000", "v": "96", "o": 4202.25, "h": 4202.5, "l": 4202.0}
}
```

---

## 订阅数字货币行情 / Subscribe Crypto Quotes

`subscribeCc(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeCc(Set<String> symbols)`

**说明 / Description:**
订阅数字货币行情数据。
Subscribe to crypto quote data.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 数字货币代码列表，如 `ETH.USD`, `BTC.USD` |

**回调接口 / Callbacks:**
- `void ccChange(QuoteBasicData data)` — 数字货币行情回调 Crypto quote
- `void ccAskBidChange(QuoteBBOData data)` — 数字货币最优买卖价回调 Crypto BBO

**示例 / Example:**

```java
Set<String> ccSymbols = new HashSet<>();
ccSymbols.add("BTC.USD");
ccSymbols.add("ETH.USD");

client.subscribeCc(ccSymbols);
client.getSubscribedSymbols();

TimeUnit.SECONDS.sleep(60000);
client.disconnect();
```

**返回数据示例 / Response example:**
```json
{
    "symbol": "ETH.USD",
    "type": "BASIC",
    "timestamp": "1770041127619",
    "serverTimestamp": "1770041127816",
    "latestPrice": 2322.97,
    "preClose": 2313.05,
    "volumeDecimal": 27711.5,
    "amount": 62465538.27,
    "open": 2314.18,
    "high": 2374.87,
    "low": 2156.8,
    "marketStatus": "TRADING"
}
```

---

## 订阅深度行情 / Subscribe Depth Quotes

`subscribeDepthQuote(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeDepthQuote(Set<String> symbols)`

**说明 / Description:**
订阅多档深度行情，支持股票（美股/港股）、期权（美股/港股）和期货。美股深度行情推送频率为 300ms，港股深度行情推送频率为 2s，返回最高 40 档买卖盘挂单数据（港股只有 10 档）。
Subscribe multi-level depth quotes. Supports stocks (US/HK), options (US/HK), and futures. US push interval: 300ms, HK push interval: 2s. Returns up to 40 levels (HK: 10 levels).

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票、期权、期货代码列表 |

**回调接口 / Callback:**
- `void depthQuoteChange(QuoteDepthData data)`

**QuoteDepthData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 股票标的 |
| timestamp | long | 深度数据时间 |
| ask | `List<OrderBook>` | 卖盘数据 |
| bid | `List<OrderBook>` | 买盘数据 |

**OrderBook 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| price | double | 每档价格 |
| volume | long | 挂单量 |
| orderCount | int | 订单数（只有港股股票有值） |
| exchange | string | 期权数据源（只有期权有值），price/volume 为 0 表示已失效 |
| time | long | 期权交易所挂单时间戳（只有期权有值） |

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("AAPL");
symbols.add("ESmain");
symbols.add("AAPL 20240209 180.0 CALL");

client.subscribeDepthQuote(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeDepthQuote(symbols);
```

**返回数据示例 / Response example (HK):**
```json
{
  "symbol": "00700",
  "timestamp": "1670465696884",
  "ask": {
    "price": [311.4, 311.6, 311.8, 312.0, 312.2, 312.4, 312.6, 312.8, 313.0, 313.2],
    "volume": ["15600", "5700", "16600", "33800", "61100", "14800", "28300", "28400", "61100", "39200"],
    "orderCount": [16, 13, 19, 79, 39, 29, 66, 56, 160, 27]
  },
  "bid": {
    "price": [311.2, 311.0, 310.8, 310.6, 310.4, 310.2, 310.0, 309.8, 309.6, 309.4],
    "volume": ["2300", "8300", "18000", "8800", "7700", "8500", "26700", "11700", "13700", "22600"],
    "orderCount": [10, 15, 18, 9, 6, 11, 17, 30, 10, 5]
  }
}
```

---

## 订阅逐笔成交数据 / Subscribe Trade Ticks

`subscribeTradeTick(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeTradeTick(Set<String> symbols)`

**说明 / Description:**
订阅股票的逐笔成交数据。逐笔推送间隔为 200ms，采用快照方式推送，每次推送最新的 50 条逐笔记录。支持美国和香港市场股票、期货标的。
Subscribe trade tick data (snapshot mode). Push interval: 200ms, each push contains latest 50 tick records. Supports US/HK stocks and futures.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票、期货代码列表 |

**回调接口 / Callback:**
- `void tradeTickChange(TradeTick data)`

**TradeTick 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 股票标的、期货标的 |
| secType | SecType | STK/FUT |
| quoteLevel | string | 数据来自的行情权限级别 |
| timestamp | long | 数据时间戳 |
| ticks | `List<Tick>` | 逐笔成交数据集合 |

**Tick 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| sn | long | 逐笔序号 |
| volume | long | 成交量 |
| tickType | string | `*`中性, `+`主动买入, `-`主动卖出（期货无此字段） |
| price | double | 成交价 |
| time | long | 交易时间戳 |
| cond | string | 成交条件，空表示自动对盘成交（期货无此字段） |
| partCode | string | 交易所 code（仅美股股票） |
| partName | string | 交易所名（仅美股股票） |

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("AAPL");
symbols.add("00700");
symbols.add("ESmain");

client.subscribeTradeTick(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeTradeTick(symbols);
```

**返回数据示例 / Response examples:**

美股/US:
```json
{
  "symbol": "AAPL",
  "secType": "STK",
  "quoteLevel": "usQuoteBasic",
  "timestamp": 1676993925700,
  "ticks": [
    {"sn": 116202, "volume": 50, "tickType": "*", "price": 149.665, "time": 1676993924289, "cond": "US_REGULAR_SALE"},
    {"sn": 116203, "volume": 1, "tickType": "*", "price": 149.68, "time": 1676993924459, "cond": "US_REGULAR_SALE"}
  ]
}
```

港股/HK:
```json
{
  "symbol": "00700",
  "secType": "STK",
  "quoteLevel": "hkStockQuoteLv2",
  "timestamp": 1669345639970,
  "ticks": [
    {"sn": 35115, "volume": 300, "tickType": "+", "price": 269.2, "time": 1669345639496, "cond": "HK_AUTOMATCH_NORMAL"}
  ]
}
```

期货/Futures:
```json
{
  "symbol": "HSImain",
  "secType": "FUT",
  "timestamp": 1669345640575,
  "ticks": [
    {"sn": 261560, "volume": 1, "price": 17465.0, "time": 1669345639000}
  ]
}
```

---

## 订阅全量逐笔成交数据 / Subscribe Full Tick Data

`subscribeTradeTick(Set<String> symbols)` (需配置 `useFullTick = true`)

**取消方法 / Cancel:** `cancelSubscribeTradeTick(Set<String> symbols)`

**说明 / Description:**
订阅股票的全量逐笔成交数据。需要找管理员申请开通权限。区分快照逐笔数据，需在开通权限后配置 `ClientConfig.DEFAULT_CONFIG.useFullTick = true;`。
Subscribe full tick-by-tick data. Requires admin permission. Set `useFullTick = true` after permission is granted.

支持美国和香港市场股票。Supports US and HK stocks.

**配置步骤 / Configuration:**

```java
ClientConfig.DEFAULT_CONFIG.useFullTick = true;
```

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票代码列表 |

**回调接口 / Callback:**
- `void fullTickChange(TickData data)`

**TickData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 股票标的 |
| timestamp | long | 数据时间戳 |
| ticks | `List<Tick>` | 逐笔成交数据集合 |

**Tick 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| sn | long | 逐笔序号 |
| time | long | 交易时间戳 |
| price | float | 成交价 |
| volume | long | 成交量 |
| type | string | `*`中性, `+`主动买入, `-`主动卖出 |
| cond | string | 成交条件，可能为 null |
| partCode | string | 交易所 code（仅美股股票），可能为 null |

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("AAPL");
symbols.add("00700");

client.subscribeTradeTick(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeTradeTick(symbols);
```

**返回数据示例 / Response example:**
```json
{
  "symbol": "AAPL",
  "ticks": [
    {"sn": "69745", "time": "1712585464248", "price": 168.96, "volume": 26, "type": "+", "partCode": "t"},
    {"sn": "69746", "time": "1712585464248", "price": 168.96, "volume": 22, "type": "+", "partCode": "t"}
  ],
  "timestamp": "1712585464415",
  "source": "NLS"
}
```

---

## 订阅夜盘全量逐笔成交数据 / Subscribe After-Hours Full Tick Data

`subscribeTradeTick(Set<String> symbols)`

**说明 / Description:**
订阅指定美股的夜盘全量逐笔成交数据。夜盘分钟K线和分钟K线使用同一订阅方法和回调方法，有夜盘权限才会推送夜盘数据。
Subscribe after-hours full tick data for US stocks. After-hours kline and regular kline share the same subscription/callback methods. Requires after-hours permission.

**权限说明 / Permissions:**
- 订阅全量逐笔权限：需联系管理员申请开通 / Full tick permission: contact admin
- 夜盘权限：需联系管理员单独开通 / After-hours permission: contact admin separately

**配置步骤 / Configuration:**

```java
ClientConfig.DEFAULT_CONFIG.useFullTick = true;
```

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票代码列表 |

**回调接口 / Callback:**
- `void fullTickChange(TickData data)`

TickData 数据结构同上 / Same structure as Full Tick Data above.

**Tick 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| sn | long | 逐笔序号 |
| time | long | 交易时间戳 |
| price | float | 成交价 |
| volume | long | 成交量 |
| type | string | `*`中性, `+`主动买入, `-`主动卖出 |
| cond | string | 成交条件，可能为 null |
| partCode | string | 交易所 code（仅美股股票），可能为 null |

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("NVDA");

client.subscribeTradeTick(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeTradeTick(symbols);
```

**返回数据示例 / Response example:**
```json
{
  "symbol": "NVDA",
  "ticks": [
    {"sn": "41492", "time": "1764054920794", "price": 178.73, "volume": 1, "type": "+"}
  ],
  "timestamp": "1764054920962",
  "source": "BOATS"
}
```

---

## 订阅分钟K线数据 / Subscribe Minute K-line Data

`subscribeKline(Set<String> symbols)`

**取消方法 / Cancel:** `cancelSubscribeKline(Set<String> symbols)`

**说明 / Description:**
订阅股票的分钟K线数据，需要找管理员申请开通权限。支持美国和香港市场股票。
Subscribe minute K-line data. Requires admin permission. Supports US and HK stocks.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| symbols | Set\<String> | Yes | 股票代码列表 |

**回调接口 / Callback:**
- `void klineChange(KlineData data)`

**KlineData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| time | long | 分钟时间戳 |
| open | float | 开始价 |
| high | float | 最高价 |
| low | float | 最低价 |
| close | float | 最终价 |
| avg | float | 平均价 |
| volume | long | 成交量 |
| count | int | 成交笔数 |
| symbol | string | 股票标的 |
| amount | double | 成交金额 |
| serverTimestamp | long | 服务器推送时间戳 |

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("AAPL");
symbols.add("00700");

client.subscribeKline(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeKline(symbols);
```

**返回数据示例 / Response example:**
```json
{
  "time": "1712584560000",
  "open": 168.9779,
  "high": 169.0015,
  "low": 168.9752,
  "close": 169.0,
  "avg": 168.778,
  "volume": "3664",
  "count": 114,
  "symbol": "AAPL",
  "amount": 617820.6508,
  "serverTimestamp": "1712584569746"
}
```

---

## 订阅夜盘分钟K线数据 / Subscribe After-Hours Minute K-line Data

`subscribeKline(Set<String> symbols)`

**说明 / Description:**
订阅指定美股的夜盘分钟K线数据。夜盘分钟K线和分钟K线使用同一订阅方法和回调方法，有夜盘权限才会推送夜盘数据。
Subscribe after-hours minute K-line data for US stocks. Shares the same method/callback as regular kline. Requires after-hours permission.

**权限说明 / Permissions:**
- 分钟K线订阅权限：需联系管理员申请开通 / Minute K-line permission: contact admin
- 夜盘权限：需联系管理员单独开通 / After-hours permission: contact admin separately

**回调接口 / Callback:**
- `void klineChange(KlineData data)`

KlineData 数据结构同上 / Same structure as Minute K-line above.

**示例 / Example:**

```java
Set<String> symbols = new HashSet<>();
symbols.add("NVDA");

client.subscribeKline(symbols);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeKline(symbols);
```

**返回数据示例 / Response example:**
```json
{
  "time": "1764054840000",
  "open": 178.73,
  "high": 178.77,
  "low": 178.69,
  "close": 178.72,
  "avg": 178.94081,
  "volume": "3470",
  "count": 80,
  "symbol": "NVDA",
  "amount": 620154.12,
  "serverTimestamp": "1764054901017"
}
```

---

## 订阅股票行情榜单数据 / Subscribe Stock Top Ranking

`subscribeStockTop(Market market, Set<Indicator> indicators)`

**取消方法 / Cancel:** `cancelSubscribeStockTop(Market market, Set<Indicator> indicators)`

**说明 / Description:**
订阅股票行情榜单数据，非交易时间不推送，推送间隔为 30s，每次推送订阅指标 Top30 的标的数据。支持美国和香港市场。盘中数据榜单有 StockRankingIndicator 所有指标，美股盘前盘后只有 changeRate 和 changeRate5Min 两个指标。
Subscribe stock top ranking data. Push interval: 30s, Top30 per indicator. US and HK markets supported.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| market | Market | Yes | 市场: US, HK |
| indicators | `Set<Indicator>` | No | 股票榜单指标，默认全部。StockRankingIndicator: changeRate(涨幅), changeRate5Min(5分钟涨幅), turnoverRate(换手率), amount(成交额), volume(成交量), amplitude(振幅) |

**回调接口 / Callback:**
- `void stockTopPush(StockTopData data)`

**StockTopData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| market | string | 市场: US/HK |
| timestamp | long | 时间戳 |
| topData | `List<TopData>` | 各指标榜单数据列表 |

**TopData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| targetName | string | 指标名 |
| item | `List<StockItem>` | 该指标维度下的榜单数据列表 |

**StockItem 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 标的 |
| latestPrice | double | 最新价 |
| targetValue | double | 对应指标值 |

**示例 / Example:**

```java
Market market = Market.US;
Set<Indicator> indicators = new HashSet<>();
indicators.add(StockRankingIndicator.Amplitude);
indicators.add(StockRankingIndicator.TurnoverRate);

// 订阅市场所有指标 / Subscribe all indicators
client.subscribeStockTop(market, null);

TimeUnit.SECONDS.sleep(120000);
// 取消部分指标 / Cancel specific indicators
client.cancelSubscribeStockTop(market, indicators);
// 取消所有指标 / Cancel all indicators
client.cancelSubscribeStockTop(market, null);
```

---

## 订阅期权行情榜单数据 / Subscribe Option Top Ranking

`subscribeOptionTop(Market market, Set<Indicator> indicators)`

**取消方法 / Cancel:** `cancelSubscribeOptionTop(Market market, Set<Indicator> indicators)`

**说明 / Description:**
订阅期权行情榜单数据，非交易时间不推送，推送间隔为 30s，每次推送订阅指标 Top50 的标的数据。支持美国市场。异动大单为单笔成交量大于 1000。
Subscribe option top ranking data. Push interval: 30s, Top50 per indicator. US market only. Big orders = single trade volume > 1000.

**请求参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| market | Market | Yes | 市场: US |
| indicators | `Set<Indicator>` | No | 期权榜单指标，默认全部。OptionRankingIndicator: bigOrder(异动大单), volume(成交量), amount(成交额), openInt(未平仓量) |

**回调接口 / Callback:**
- `void optionTopPush(OptionTopData data)`

**OptionTopData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| market | string | 市场: US |
| timestamp | long | 时间戳 |
| topData | `List<TopData>` | 各指标榜单数据列表 |

**TopData 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| targetName | string | 指标名 (bigOrder, volume, amount, openInt) |
| bigOrder | `List<BigOrder>` | 异动大单指标数据列表 |
| item | `List<OptionItem>` | 该指标维度下的榜单数据列表 |

**BigOrder 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 股票/ETF 标的 |
| expiry | string | 过期日，格式: yyyyMMdd |
| strike | string | 行权价 |
| right | string | CALL/PUT |
| dir | string | 买卖方向: BUY/SELL/NONE |
| volume | double | 成交量 |
| price | double | 成交价 |
| amount | double | 成交额 |
| tradeTime | long | 成交时间戳 |

**OptionItem 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| symbol | string | 股票/ETF 标的 |
| expiry | string | 过期日，格式: yyyyMMdd |
| strike | string | 行权价 |
| right | string | CALL/PUT |
| totalAmount | double | 成交额 |
| totalVolume | double | 成交量 |
| totalOpenInt | double | 未平仓量 |
| volumeToOpenInt | double | 成交量/未平仓量 |
| latestPrice | double | 最新价 |
| updateTime | long | 指标数据更新时间戳 |

**示例 / Example:**

```java
Market market = Market.US;
Set<Indicator> indicators = new HashSet<>();
indicators.add(OptionRankingIndicator.Amount);
indicators.add(OptionRankingIndicator.OpenInt);

client.subscribeOptionTop(market, null);

TimeUnit.SECONDS.sleep(120000);
client.cancelSubscribeOptionTop(market, indicators);
client.cancelSubscribeOptionTop(market, null);
```

---

## 查询已订阅的标的 / Query Subscribed Symbols

`getSubscribedSymbols()`

**说明 / Description:**
查询已经订阅过的标的信息。
Query currently subscribed symbols.

**回调接口 / Callback:**

```java
void getSubscribedSymbolEnd(SubscribedSymbol subscribedSymbol)
```

**SubscribedSymbol 数据结构:**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| limit | int | 订阅行情标的限制最大数 |
| used | int | 已订阅行情标的数 |
| subscribedSymbols | array | 已订阅行情标的 |
| askBidLimit | int | 订阅深度行情限制最大数 |
| askBidUsed | int | 已订阅深度行情数 |
| subscribedAskBidSymbols | array | 已订阅深度行情标的 |
| tradeTickLimit | int | 订阅逐笔成交限制最大数 |
| tradeTickUsed | int | 已订阅逐笔成交数 |
| subscribedTradeTickSymbols | array | 已订阅逐笔成交标的 |

**示例 / Example:**

```java
client.getSubscribedSymbols();
```

**返回数据示例 / Response example:**
```json
{
    "askBidLimit": 10,
    "askBidUsed": 0,
    "limit": 20,
    "subscribedAskBidSymbols": [],
    "subscribedSymbols": [],
    "subscribedTradeTickSymbols": ["PDD", "AMD", "SPY", "01810"],
    "tradeTickLimit": 20,
    "tradeTickUsed": 4,
    "used": 0
}
```

---

## 账户变动推送 / Account Change Push

### 订阅 / Subscribe

`subscribe(Subject subject)`
`subscribe(String account, Subject subject)`

**取消方法 / Cancel:** `cancelSubscribe(Subject subject)`

**说明 / Description:**
交易推送接口提供订单、持仓、资产、订单执行明细的实时变动推送。通过实现 `ApiComposeCallback` 接口接收数据。默认推送全部资金账号数据（包括模拟账号），可按返回数据的 account 区分，或指定 account 订阅。
Trade push provides real-time order, position, asset, and transaction change notifications. Implement `ApiComposeCallback` to receive data. Pushes all accounts (including paper) by default.

**输入参数 / Parameters:**

| 参数 | 类型 | 是否必填 | 说明 |
| --- | --- | --- | --- |
| account | string | No | 资金账号，不设置则订阅所有资金账号（含模拟账号） |
| subject | Subject | Yes | 订阅主题 |

**Subject 订阅主题 / Subscription subjects:**

- **OrderStatus** — 订单状态变化推送 (Submitted/Cancelled/Inactive/Filled)
- **Asset** — 账户资产变化推送
- **Position** — 账户持仓变化推送
- **OrderTransaction** — 订单执行明细报告推送

> **注意:** 订阅上述四种类型的任意一个主题，都会推送全部四种主题数据。取消四种中的任意一个主题，会取消全部四种主题数据的订阅。
> **Note:** Subscribing to any one of the four subjects will push all four types. Cancelling any one will cancel all four.

**回调接口 / Callbacks:**

```java
void assetChange(AssetData data)                          // 资产变动 Asset change (subject = Asset)
void positionChange(PositionData data)                    // 持仓变动 Position change (subject = Position)
void orderStatusChange(OrderStatusData data)              // 订单变动 Order change (subject = OrderStatus)
void orderTransactionChange(OrderTransactionData data)    // 订单执行明细 Transaction (subject = OrderTransaction)
```

**示例 / Example:**

```java
client.connect();

// 订阅 订单/资产/持仓/订单执行报告 / Subscribe order/asset/position/transaction
client.subscribe(Subject.OrderStatus);
client.subscribe(Subject.Asset);
client.subscribe(Subject.Position);
client.subscribe(Subject.OrderTransaction);

TimeUnit.SECONDS.sleep(60000);

// 取消订阅 / Cancel subscription
client.cancelSubscribe(Subject.Asset);
client.cancelSubscribe(Subject.Position);
client.cancelSubscribe(Subject.OrderStatus);
client.cancelSubscribe(Subject.OrderTransaction);
```

---

### 资产变动回调字段 / Asset Change Callback Fields

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| account | String | 资金账号 |
| currency | String | 币种。USD美元，HKD港币 |
| segType | String | 分类。S表示股票，C表示期货 |
| availableFunds | double | 可用资金 |
| excessLiquidity | double | 当前剩余流动性 |
| netLiquidation | double | 总资产(净清算值) |
| equityWithLoan | double | 含贷款价值总权益 |
| buyingPower | double | 购买力（仅股票品种） |
| cashBalance | double | 现金额 |
| grossPositionValue | double | 证券总价值 |
| initMarginReq | double | 初始保证金 |
| maintMarginReq | double | 维持保证金 |
| timestamp | long | 时间戳 |

**返回示例 / Response example:**
```json
{
  "account": "13810712",
  "currency": "USD",
  "segment": "S",
  "availableFunds": 2285040.55,
  "excessLiquidity": 2284942.05,
  "netLiquidation": 2285529.37,
  "equityWithLoan": 2285418.28,
  "buyingPower": 9140162.19,
  "cashBalance": 2284275.24,
  "grossPositionValue": 1143.04,
  "initMarginReq": 377.736,
  "maintMarginReq": 476.236,
  "timestamp": "1669888806020"
}
```

---

### 持仓变动回调字段 / Position Change Callback Fields

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| account | String | 资金账号 |
| symbol | String | 持仓标的代码，如 `AAPL`, `00700`, `ES`, `CN` |
| expiry | String | 仅支持期权、窝轮、牛熊证 |
| strike | String | 仅支持期权、窝轮、牛熊证 |
| right | String | 仅支持期权、窝轮、牛熊证 |
| identifier | String | 标的标识符，期货带合约月份如 `CN2201` |
| multiplier | int | 每手数量 (futures, options, warrants, CBBC) |
| market | String | 市场: US, HK |
| currency | String | 币种: USD, HKD |
| segType | String | 分类: S股票, C期货 |
| secType | String | STK/OPT/WAR/IOPT/CASH/FUT/FOP |
| positionQty | double | 持仓数量 |
| salableQty | double | 可卖数量 |
| averageCost | double | 持仓均价 |
| latestPrice | double | 标的当前价格 |
| marketValue | double | 持仓市值 |
| unrealizedPnl | double | 持仓盈亏 |
| timestamp | long | 时间戳 |

**返回示例 / Response example:**
```json
{
  "account": "13810712",
  "symbol": "AAPL",
  "identifier": "AAPL",
  "multiplier": 1,
  "market": "US",
  "currency": "USD",
  "segment": "S",
  "secType": "STK",
  "position": "4",
  "averageCost": 75.0,
  "latestPrice": 147.23,
  "marketValue": 588.92,
  "unrealizedPnl": 288.92,
  "timestamp": "1669888802018"
}
```

---

### 订单变动回调字段 / Order Status Change Callback Fields

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | long | 订单号 |
| account | String | 资金账号 |
| symbol | String | 标的代码 |
| expiry | String | 仅期权/窝轮/牛熊证 |
| strike | String | 仅期权/窝轮/牛熊证 |
| right | String | 仅期权/窝轮/牛熊证 |
| identifier | String | 标的标识符 |
| multiplier | int | 每手数量 |
| action | String | BUY/SELL |
| market | String | US/HK |
| currency | String | USD/HKD |
| segType | String | S股票/C期货 |
| secType | String | STK/OPT/WAR/IOPT/CASH/FUT/FOP |
| orderType | String | MKT/LMT/STP/STP_LMT/TRAIL |
| isLong | boolean | 是否多头持仓 |
| totalQuantity | long | 下单数量 |
| totalQuantityScale | int | 下单数量偏移量 |
| filledQuantity | long | 成交总数量（累计） |
| filledQuantityScale | int | 成交总数量偏移量 |
| avgFillPrice | double | 成交均价 |
| limitPrice | double | 限价单价格 |
| stopPrice | double | 止损价格 |
| realizedPnl | double | 已实现盈亏（仅综合账号） |
| status | String | 订单状态 |
| replaceStatus | String | 订单改单状态 |
| cancelStatus | String | 订单撤单状态 |
| outsideRth | boolean | 是否允许盘前盘后交易（仅美股） |
| canModify | boolean | 是否能修改 |
| canCancel | boolean | 是否能取消 |
| liquidation | boolean | 是否为平仓订单 |
| name | String | 标的名称 |
| source | String | 订单来源 |
| errorMsg | String | 错误信息 |
| attrDesc | String | 订单描述信息 |
| commissionAndFee | float | 佣金费用总计 |
| openTime | long | 下单时间 |
| timestamp | long | 订单状态最后更新时间 |
| userMark | String | 自定义标注信息 |
| totalCashAmount | double | 下单总金额（仅限金额订单） |
| filledCashAmount | double | 成交金额（仅限金额订单） |
| attrList | `List<String>` | 订单属性列表 (LIQUIDATION, FRACTIONAL_SHARE, EXERCISE, EXPIRE, ASSIGNMENT, etc.) |
| timeInForce | string | 订单有效时间: DAY/GTC/GTD |

**返回示例 / Response example:**
```json
{
  "id": "28875370355884032",
  "account": "736845",
  "symbol": "CL",
  "identifier": "CL2312",
  "multiplier": 1000,
  "action": "BUY",
  "market": "US",
  "currency": "USD",
  "segment": "C",
  "secType": "FUT",
  "orderType": "LMT",
  "isLong": true,
  "totalQuantity": "1",
  "filledQuantity": "1",
  "avgFillPrice": 77.76,
  "limitPrice": 77.76,
  "status": "Filled",
  "outsideRth": true,
  "name": "WTI原油2312",
  "source": "android",
  "commissionAndFee": 4.0,
  "openTime": "1669200792000",
  "timestamp": "1669200782221"
}
```

> 订单如果分多次成交，`filledQuantity` 为历史累计成交总数。
> For orders filled in multiple parts, `filledQuantity` is the cumulative total.

---

### 订单执行明细回调字段 / Order Transaction Change Callback Fields

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | long | 订单执行ID |
| orderId | long | 订单号 |
| account | String | 资金账号 |
| symbol | String | 标的代码 |
| identifier | String | 标的标识符 |
| multiplier | int | 每手数量(期权、期货) |
| action | String | BUY/SELL |
| market | String | US/HK |
| currency | String | USD/HKD |
| segType | String | S股票/C期货 |
| secType | String | STK/FUT |
| filledPrice | double | 成交价格 |
| filledQuantity | long | 成交数量 |
| createTime | long | 创建时间 |
| updateTime | long | 更新时间 |
| transactTime | long | 成交时间 |
| timestamp | long | 时间戳 |

**返回示例 / Response example:**
```json
{
  "id": "28875370482237440",
  "orderId": "28875370355884032",
  "account": "736845",
  "symbol": "CL",
  "identifier": "CL2312",
  "multiplier": 1000,
  "action": "BUY",
  "market": "US",
  "currency": "USD",
  "segment": "C",
  "secType": "FUT",
  "filledPrice": 77.76,
  "filledQuantity": "1",
  "createTime": "1669200793664",
  "updateTime": "1669200793664",
  "transactTime": "1669200793593",
  "timestamp": "1669200782233"
}
```

---

## 完整示例 / Complete Example

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.socket.WebSocketClient;
import com.tigerbrokers.stock.openapi.client.struct.enums.Subject;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.TimeUnit;

public class WebSocketDemo {

  private static ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
  private static WebSocketClient client;

  static {
    clientConfig.configFilePath = "/data/tiger_config";
    // clientConfig.secretKey = "xxxxxx"; // institutional trader
    client = WebSocketClient.getInstance()
        .clientConfig(clientConfig)
        .apiComposeCallback(new DefaultApiComposeCallback());
  }

  public static void main(String[] args) throws Exception {
    // 建立连接 / Connect
    client.connect();

    // 订阅股票行情 / Subscribe stock quotes
    Set<String> symbols = new HashSet<>();
    symbols.add("AAPL");
    symbols.add("00700");
    client.subscribeQuote(symbols);

    // 订阅深度数据 / Subscribe depth
    client.subscribeDepthQuote(symbols);

    // 订阅逐笔成交 / Subscribe trade ticks
    client.subscribeTradeTick(symbols);

    // 订阅期权行情 / Subscribe options
    Set<String> optionSymbols = new HashSet<>();
    optionSymbols.add("AAPL 20230317 150.0 CALL");
    client.subscribeOption(optionSymbols);

    // 订阅期货行情 / Subscribe futures
    Set<String> futureSymbols = new HashSet<>();
    futureSymbols.add("ESmain");
    client.subscribeFuture(futureSymbols);

    // 订阅账户变动 / Subscribe account changes
    client.subscribe(Subject.OrderStatus);
    client.subscribe(Subject.Asset);
    client.subscribe(Subject.Position);
    client.subscribe(Subject.OrderTransaction);

    // 查询订阅详情 / Query subscriptions
    client.getSubscribedSymbols();

    // 等待接收数据 / Wait
    TimeUnit.SECONDS.sleep(60000);

    // 取消行情订阅 / Cancel quote subscriptions
    client.cancelSubscribeQuote(symbols);
    client.cancelSubscribeDepthQuote(symbols);
    client.cancelSubscribeTradeTick(symbols);
    client.cancelSubscribeOption(optionSymbols);
    client.cancelSubscribeFuture(futureSymbols);

    // 取消账户订阅 / Cancel account subscriptions
    client.cancelSubscribe(Subject.OrderStatus);

    // 断开连接（会自动注销全部订阅） / Disconnect (auto-unsubscribes all)
    // client.disconnect();
  }
}
```

---

## 所有回调接口一览 / All Callback Methods

| 回调方法 Callback Method | 说明 Description |
| --- | --- |
| `quoteChange(QuoteBasicData)` | 股票基本行情 Stock quote |
| `quoteAskBidChange(QuoteBBOData)` | 股票最优买卖价 Stock BBO |
| `optionChange(QuoteBasicData)` | 期权行情 Option quote |
| `optionAskBidChange(QuoteBBOData)` | 期权最优买卖价 Option BBO |
| `futureChange(QuoteBasicData)` | 期货行情 Future quote |
| `futureAskBidChange(QuoteBBOData)` | 期货最优买卖价 Future BBO |
| `ccChange(QuoteBasicData)` | 数字货币行情 Crypto quote |
| `ccAskBidChange(QuoteBBOData)` | 数字货币最优买卖价 Crypto BBO |
| `depthQuoteChange(QuoteDepthData)` | 深度行情 Depth quote |
| `tradeTickChange(TradeTick)` | 逐笔成交(快照) Trade tick snapshot |
| `fullTickChange(TickData)` | 全量逐笔成交 Full tick |
| `klineChange(KlineData)` | 分钟K线 Minute K-line |
| `stockTopPush(StockTopData)` | 股票榜单 Stock top ranking |
| `optionTopPush(OptionTopData)` | 期权榜单 Option top ranking |
| `orderStatusChange(OrderStatusData)` | 订单变动 Order change |
| `orderTransactionChange(OrderTransactionData)` | 订单执行明细 Order transaction |
| `positionChange(PositionData)` | 持仓变动 Position change |
| `assetChange(AssetData)` | 资产变动 Asset change |
| `subscribeEnd(int, String, String)` | 订阅完成 Subscribe end |
| `cancelSubscribeEnd(int, String, String)` | 取消订阅完成 Cancel subscribe end |
| `getSubscribedSymbolEnd(SubscribedSymbol)` | 查询已订阅 Query subscribed |
| `connectionAck()` | 连接成功 Connected |
| `connectionAck(int, int)` | 连接成功（含心跳间隔）Connected with heartbeat intervals |
| `connectionClosed()` | 连接关闭 Connection closed |
| `connectionKickout(int, String)` | 被踢出 Kicked out |
| `hearBeat(String)` | 心跳 Heartbeat |
| `serverHeartBeatTimeOut(String)` | 服务器心跳超时 Server heartbeat timeout |
| `error(String)` | 异常 Error |
| `error(int, int, String)` | 异常(带ID) Error with id |

---

## 注意事项 / Notes

- 同一 tiger_id 只能有一个推送连接 / Only one push connection per tiger_id
- 新连接会踢掉旧连接（触发 `connectionKickout` 回调） / New connection kicks old one
- 主动断开连接（`disconnect()`）会自动注销所有订阅 / `disconnect()` auto-unsubscribes all
- 行情推送需对应行情权限 / Quote push requires corresponding permissions
- 深度推送需 L2 权限 / Depth push requires L2 permission
- 全量逐笔需找管理员开通权限并配置 `useFullTick = true` / Full tick requires admin permission and `useFullTick = true`
- 夜盘数据需单独开通夜盘权限 / After-hours data requires separate permission
- 分钟K线需找管理员开通权限 / Minute K-line requires admin permission
- 订阅账户变动（OrderStatus/Asset/Position/OrderTransaction）任一主题会推送全部四种数据 / Subscribing to any account subject pushes all four
- 取消账户任一主题订阅会取消全部四种 / Cancelling any account subject cancels all four
- 非交易时间段建议关闭连接 / Recommended to close connection outside trading hours
- `subscribeOption` 使用空格分隔格式或 identifier 格式 / Option subscription supports space-separated or identifier format
- 传入空 `HashSet<>()` 可取消对应类型的全部订阅 / Pass empty `HashSet<>()` to cancel all subscriptions of that type
