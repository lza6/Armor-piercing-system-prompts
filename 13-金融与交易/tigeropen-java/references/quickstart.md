
# Tiger Open API Java SDK

> 中文 | English — 本技能双语编写，代码示例通用。This skill is bilingual with shared code examples.

## 概述 / Overview

老虎量化开放平台 Java SDK (openapi-java-sdk) 为个人和机构用户提供交易与行情 API，支持美股、港股、A股、新加坡、澳洲市场的股票、期权、期货等品种。
The Tiger Open Platform Java SDK provides trading and market data APIs for stocks, options, futures across US, HK, China A-shares, Singapore, and Australia markets.

- 官方文档 / Docs: https://quant.itigerup.com/openapi/java/overview/introduction
- GitHub: https://github.com/tigerbrokers/openapi-java-sdk
- Gitee: https://gitee.com/tigerbrokers/openapi-java-sdk
- SDK Version: 2.6.0 | Java: JDK 1.8+ (64-bit)

### 支持的市场和品种 / Supported Markets

| 市场 Market | 股票/ETF | 期权 Options | 期货 Futures | 窝轮/牛熊证 Warrants/CBBC |
|-------------|---------|-------------|-------------|--------------------------|
| 美国 US     | ✅      | ✅          | ✅          | -                        |
| 香港 HK     | ✅      | ✅          | ✅          | ✅                       |
| 新加坡 SG   | ✅      | -           | ✅          | -                        |
| 澳洲 AU    | ✅      | -           | -           | -                        |
| A股 CN      | ✅      | -           | -           | -                        |

## 安装 / Installation

### Maven (推荐 / Recommended)

```xml
<dependency>
  <groupId>io.github.tigerbrokers</groupId>
  <artifactId>openapi-java-sdk</artifactId>
  <version>2.6.0</version>
</dependency>
```

如果无法下载，尝试添加仓库源 / If download fails, add repository:

```xml
<repositories>
  <repository>
    <id>sonatype-public</id>
    <name>sonatype-public</name>
    <url>https://oss.sonatype.org/content/groups/public/</url>
  </repository>
</repositories>
```

### Gradle

```groovy
dependencies {
    implementation 'io.github.tigerbrokers:openapi-java-sdk:2.6.0'
}
```

### 从源码构建 / Build from Source

```shell
git clone https://github.com/tigerbrokers/openapi-java-sdk.git
cd openapi-java-sdk
mvn clean install
```

## 前置条件 / Prerequisites

1. 开通老虎证券账户并入金 / Open a Tiger Brokers account and fund it
2. 个人用户访问 https://developer.itigerup.com/profile 激活 API 权限 / Individual users activate API permissions
   机构用户访问机构账户中心 / Institutional users visit institutional account center
3. 获取 `tiger_id`、RSA 私钥(**PKCS#8 格式**)和资金账号 / Obtain `tiger_id`, RSA private key (**PKCS#8 format**), and account
4. 导出配置文件 `tiger_openapi_config.properties` / Export config file

> 私钥不会保存在服务端，刷新页面后消失，请务必保存。Private key is not stored on server; save it before refreshing the page.
> Java SDK 使用 PKCS#8 格式（`private_key_pk8`）。Java SDK uses PKCS#8 format.

## 账户类型 / Account Types

| 类型 Type | 说明 Description | 推荐 |
|-----------|-----------------|------|
| 综合账户 Comprehensive (Live) | 支持所有市场、保证金交易、全品种。**推荐** | ✅ |
| 环球账户 Global (Live) | 以 U 开头，如 U12300123 | - |
| 模拟账户 Paper | 17位数字，测试用，无需真实资金 | 测试推荐 |

## 配置 / Configuration

### 配置文件格式 / Config File Format

从开发者网站导出 `tiger_openapi_config.properties`，放入 `clientConfig.configFilePath` 指定的目录。
Export `tiger_openapi_config.properties` from developer website and place it in the `configFilePath` directory.

```properties
tiger_id=YOUR_TIGER_ID
account=12345678
license=TBHK
private_key_pk1=MIICXgIBAAKBgQC.....your_pkcs1_private_key
private_key_pk8=MIICeAIBADANBgk.....your_pkcs8_private_key
env=PROD
# 机构用户需额外配置 / Institutional users also need:
# secret_key=your_secret_key
```

> Java SDK 同时支持 **PKCS#8**（`private_key_pk8`）和 **PKCS#1**（`private_key_pk1`）格式，优先读取 `private_key_pk8`。
> Java SDK supports both PKCS#8 (`private_key_pk8`) and PKCS#1 (`private_key_pk1`). `private_key_pk8` takes priority.

> 港股牌照(TBHK)还需将 `tiger_openapi_token.properties` 放入同一目录。Token 有效期 30 天。
> HK license (TBHK) also requires `tiger_openapi_token.properties` in the same directory. Token valid for 30 days.

### API 配置与客户端初始化 / API Configuration & Client Initialization

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.util.ApiLogger;

public static ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
public static TigerHttpClient client;

static {
    // 开启日志 / Enable logging
    ApiLogger.setEnabled(true, "/data/tiger_openapi/logs/");

    // 配置文件存放路径 / Config file directory
    clientConfig.configFilePath = "/data/tiger_config";

    // 可选配置 / Optional settings:
    // clientConfig.isSslSocket = true;              // 长连接是否使用SSL, default true
    // clientConfig.isAutoGrabPermission = true;     // 启动时抢占行情权限, default true
    // clientConfig.failRetryCounts = 2;             // API请求失败重试次数(最多5), default 2
    // clientConfig.timeZone = TimeZoneId.Shanghai;  // 默认时区
    // clientConfig.language = Language.en_US;        // 默认语言
    // clientConfig.secretKey = "xxxxxx";             // 机构用户私钥 / Institutional secret key
    // clientConfig.isAutoRefreshToken = false;       // TBHK牌照自动刷新token, default false
    // clientConfig.refreshTokenIntervalDays = 5;    // 自动刷新周期(天)
    // clientConfig.refreshTokenTime = "12:30:00";   // 自动刷新时间 HH:mm:ss

    client = TigerHttpClient.getInstance().clientConfig(clientConfig);
}
```

### 不使用配置文件 / Without Config File

```java
ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
clientConfig.tigerId = "your tiger id";
clientConfig.defaultAccount = "your account";
clientConfig.privateKey = ConfigFileUtil.readPrivateKey("/path/to/rsa_private_key_pkcs8.pem");
// clientConfig.token = "xx"; // TBHK牌照需要配置
// clientConfig.secretKey = "xxxxxx"; // 机构用户

TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(clientConfig);
```

### 配置说明 / Configuration Reference

| 配置项 Config | 说明 Description |
|--------------|-----------------|
| `configFilePath` | `tiger_openapi_config.properties` 和 token 文件存放目录 |
| `tigerId` | 开发者ID (配置文件优先) |
| `defaultAccount` | 资金账号，综合/模拟账号 (配置文件优先) |
| `privateKey` | RSA 私钥 PKCS#8 格式 (配置文件优先) |
| `secretKey` | 机构交易员密钥，个人用户不设置 |
| `token` | TBHK 牌照两步验证 |
| `isSslSocket` | 长连接是否使用 SSL |
| `isAutoGrabPermission` | 是否启动时抢占行情 |
| `failRetryCounts` | 失败重试次数，最多5 |
| `timeZone` | 默认时区 |
| `language` | 默认语言 |

## 基本功能示例 / Basic Usage Examples

### 查询行情 / Query K-line

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.https.request.quote.QuoteKlineRequest;
import com.tigerbrokers.stock.openapi.client.https.response.quote.QuoteKlineResponse;
import com.tigerbrokers.stock.openapi.client.struct.enums.KType;
import com.tigerbrokers.stock.openapi.client.struct.enums.RightOption;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class QuoteDemo {

  private static final TigerHttpClient client;

  static {
    ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
    clientConfig.configFilePath = "./src/main/resources";
    client = TigerHttpClient.getInstance().clientConfig(clientConfig);
  }

  public static void main(String[] args) {
    new QuoteDemo().kline();
  }

  public void kline() {
    List<String> symbols = new ArrayList<>();
    symbols.add("AAPL");
    QuoteKlineRequest request =
        QuoteKlineRequest.newRequest(symbols, KType.day, "2022-10-01", "2022-12-25")
            .withLimit(1000)
            .withRight(RightOption.br);
    QuoteKlineResponse response = client.execute(request);
    if (response.isSuccess()) {
      System.out.println(Arrays.toString(response.getKlineItems().toArray()));
    } else {
      System.out.println("response error:" + response.getMessage());
    }
  }
}
```

### 下单 / Place Order

```java
import com.alibaba.fastjson.JSONObject;
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.https.domain.contract.item.ContractItem;
import com.tigerbrokers.stock.openapi.client.https.request.trade.TradeOrderRequest;
import com.tigerbrokers.stock.openapi.client.https.response.trade.TradeOrderResponse;
import com.tigerbrokers.stock.openapi.client.struct.enums.ActionType;

public class TradeDemo {

  private static final TigerHttpClient client;

  static {
    ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
    clientConfig.configFilePath = "./src/main/resources";
    client = TigerHttpClient.getInstance().clientConfig(clientConfig);
  }

  public static void main(String[] args) {
    new TradeDemo().placeUsStockOrder();
  }

  public void placeUsStockOrder() {
    ContractItem contract = ContractItem.buildStockContract("AAPL", "USD");
    TradeOrderRequest request =
        TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, 1, 100.0);
    TradeOrderResponse response = client.execute(request);
    System.out.println(JSONObject.toJSONString(response));
  }
}
```

### 订阅 / Subscription (WebSocket)

订阅采用异步推送模式：自定义回调函数绑定事件，服务器推送时自动调用。
Subscription uses async push: bind custom callbacks to events, auto-called on server push.

**1. 定义回调接口 / Define Callback**

实现 `ApiComposeCallback` 接口：

`ApiComposeCallback` 是宽接口，共 **29 个方法**，实现类必须全部覆盖（可留空）。
完整可编译的实现见 [push.md](push.md)。
`ApiComposeCallback` declares 29 methods — all must be implemented. See push.md
for a complete, compilable implementation.

接口方法分组 / Method groups:

| 分组 | 方法 |
|------|------|
| 连接生命周期 | `connectionAck()`、`connectionAck(int,int)`、`connectionClosed()`、`connectionKickout(int,String)`、`hearBeat(String)`、`serverHeartBeatTimeOut(String)` |
| 错误 | `error(String)`、`error(int,int,String)` |
| 订阅结果 | `subscribeEnd(int,String,String)`、`cancelSubscribeEnd(int,String,String)`、`getSubscribedSymbolEnd(SubscribedSymbol)` |
| 账户推送 | `orderStatusChange`、`orderTransactionChange`、`positionChange`、`assetChange` |
| 行情推送 | `quoteChange`、`quoteAskBidChange`、`optionChange`、`optionAskBidChange`、`futureChange`、`futureAskBidChange`、`ccChange`、`ccAskBidChange`、`depthQuoteChange`、`klineChange`、`tradeTickChange`、`fullTickChange`、`stockTopPush`、`optionTopPush` |



**2. 进行订阅 / Subscribe**

```java
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.socket.WebSocketClient;
import java.util.HashSet;
import java.util.Set;

public class SubscribeDemo {

    private static ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
    private static WebSocketClient client;

    static {
        clientConfig.configFilePath = "/data/tiger_config";
        client = WebSocketClient.getInstance()
            .clientConfig(clientConfig)
            .apiComposeCallback(new DefaultApiComposeCallback());
    }

    public static void main(String[] args) {
        new SubscribeDemo().subscribe();
    }

    public void subscribe() {
        client.connect();

        Set<String> symbols = new HashSet<>();

        // 股票订阅 / Stock subscription
        symbols.add("AAPL");
        symbols.add("SPY");
        client.subscribeQuote(symbols);

        // 期货订阅 / Futures subscription
        symbols.clear();
        symbols.add("ESmain");
        client.subscribeFuture(symbols);

        // 期权订阅 / Options subscription
        symbols.clear();
        symbols.add("AAPL 20190614 200.0 CALL");
        // 或21位格式 / Or 21-char format:
        // symbols.add("SPY   190508C00290000");
        client.subscribeOption(symbols);

        // 深度数据 / Depth data
        client.subscribeDepthQuote(symbols);

        // 逐笔数据 / Trade tick
        client.subscribeTradeTick(symbols);

        // 查询订阅详情 / Query subscriptions
        client.getSubscribedSymbols();

        // 断开连接时自动注销订阅 / Auto-unsubscribe on disconnect
        // client.disconnect();
    }
}
```

## 核心对象参考 / Key Object Reference

### 客户端 / Clients

| 对象 Object | 包路径 Package | 功能 Purpose |
|-------------|---------------|-------------|
| `TigerHttpClient` | `client.https.client` | HTTP 客户端，发送交易类和行情类请求 |
| `WebSocketClient` | `client.socket` | WebSocket 客户端，处理订阅推送请求 |

### 常用包结构 / Package Structure

| 包路径 Package | 说明 Description |
|---------------|-----------------|
| `client.constant` | 常量：MethodName(API方法)、OrderChangeKey、PositionChangeKey、AssetChangeKey |
| `client.https.client` | TigerHttpClient HTTP请求客户端 |
| `client.https.domain` | 返回的业务数据对象 |
| `client.https.request` | 封装的请求对象 |
| `client.https.response` | 封装的返回对象 |
| `client.socket` | ApiComposeCallback(回调接口)、WebSocketClient(长连接客户端) |
| `client.struct.enums` | 枚举与数据结构 |
| `client.util` | 工具类 |

### 常用枚举 / Common Enums

| 枚举 Enum | 说明 Description | 常用值 Values |
|-----------|-----------------|--------------|
| `Market` | 市场 | US, HK, CN, SG, AU, ALL |
| `SecType` | 证券类型 | STK, OPT, FUT, WAR, IOPT, FUND, CASH, MLEG |
| `Currency` | 货币 | USD, HKD, CNH, SGD, AUD |
| `ActionType` | 交易方向 | BUY, SELL |
| `OrderType` | 订单类型 | MKT, LMT, STP, STP_LMT, TRAIL |
| `KType` | K线周期 | day, week, month, year, min1, min3, min5, min10, min15, min30, min60 |
| `RightOption` | 复权类型 | br(前复权), nr(不复权) |
| `Right` | 期权方向 | CALL, PUT |
| `TimeInForce` | 订单有效期 | DAY, GTC, GTD |
| `Category` | 资产分类 | S(股票), C(期货) |
| `Language` | 语言 | zh_CN, zh_TW, en_US |
| `TimeZoneId` | 时区 | Shanghai, NewYork, LosAngeles, Chicago, HongKong, Singapore |
| `MethodName` | API方法名 | ACTIVE_ORDERS, FILLED_ORDERS, INACTIVE_ORDERS, CANCEL_ORDER, POSITIONS, ASSETS, GRAB_QUOTE_PERMISSION 等 |
| `License` | 牌照 | TBUS, TBHK, TBSG, TBAU |

### ContractItem 合约构建 / Contract Building

```java
// 股票合约 / Stock contract
ContractItem stock = ContractItem.buildStockContract("AAPL", "USD");

// 期权合约 / Option contract
// 参数顺序：(symbol, expiry, strike, right)，expiry 用 YYYYMMDD
// Argument order is (symbol, expiry, strike, right)
ContractItem option = ContractItem.buildOptionContract("AAPL", "20260821", 190.0D, "CALL");

// 期货合约 / Futures contract
// buildFutureContract 只有 (symbol, currency) 和 5 参数完整形式两个重载
// Only (symbol, currency) and the 5-arg overload exist
ContractItem future = ContractItem.buildFutureContract("ES", "USD", "CME", "20240315", 50.0D);
```

### TradeOrderRequest 下单请求 / Order Request

```java
// 限价单 / Limit order
TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, quantity, limitPrice);

// 市价单 / Market order
TradeOrderRequest.buildMarketOrder(contract, ActionType.BUY, quantity);

// 止损单 / Stop order
TradeOrderRequest.buildStopOrder(contract, ActionType.SELL, quantity, auxPrice);

// 止损限价单 / Stop limit order
TradeOrderRequest.buildStopLimitOrder(contract, ActionType.SELL, quantity, limitPrice, auxPrice);
```

### TigerHttpRequest 通用请求 / Generic Request

用于调用未封装的API方法 / For calling unwrapped API methods.

已有专用请求类的接口应优先用专用类（如持仓用 `PositionsRequest`），
只有 SDK 未封装的接口才走 `TigerHttpRequest`。
Prefer the dedicated request class when one exists; `TigerHttpRequest` is the
fallback for APIs the SDK does not wrap.

```java
// 成交记录没有专用请求类，用通用请求
// ORDER_TRANSACTIONS has no dedicated request class
TigerHttpRequest request = new TigerHttpRequest(MethodName.ORDER_TRANSACTIONS);
String bizContent = AccountParamBuilder.instance()
    .account(clientConfig.defaultAccount)
    .secType(SecType.STK)
    .symbol("AAPL")
    .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
// 通用请求返回原始 JSON，需自行解析 / returns raw JSON, parse it yourself
```

## 完整策略示例：动量策略 / Full Example: Momentum Strategy

**注意：本策略仅为接口使用示例，策略本身未经验证，不构成投资建议。**
**Note: This strategy is only a usage example. Not investment advice.**

基本思路：定期选取纳斯达克100成份股中周期内涨幅最高的若干股票买入持有，对未入选的持仓平仓。
Logic: Periodically select top momentum stocks from Nasdaq-100, buy and hold, close positions not selected.

```java
import static java.lang.Thread.sleep;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.https.domain.contract.item.ContractItem;
import com.tigerbrokers.stock.openapi.client.https.domain.quote.item.KlineItem;
import com.tigerbrokers.stock.openapi.client.https.domain.quote.item.KlinePoint;
import com.tigerbrokers.stock.openapi.client.https.domain.trade.item.PrimeAssetItem;
import com.tigerbrokers.stock.openapi.client.https.request.TigerHttpRequest;
import com.tigerbrokers.stock.openapi.client.https.request.quote.QuoteKlineRequest;
import com.tigerbrokers.stock.openapi.client.https.request.quote.QuoteRealTimeQuoteRequest;
import com.tigerbrokers.stock.openapi.client.https.request.trade.PrimeAssetRequest;
import com.tigerbrokers.stock.openapi.client.https.request.trade.TradeOrderRequest;
import com.tigerbrokers.stock.openapi.client.https.response.TigerHttpResponse;
import com.tigerbrokers.stock.openapi.client.https.response.quote.QuoteKlineResponse;
import com.tigerbrokers.stock.openapi.client.https.response.quote.QuoteRealTimeQuoteResponse;
import com.tigerbrokers.stock.openapi.client.https.response.trade.PrimeAssetResponse;
import com.tigerbrokers.stock.openapi.client.https.response.trade.TradeOrderResponse;
import com.tigerbrokers.stock.openapi.client.struct.enums.*;
import com.tigerbrokers.stock.openapi.client.util.builder.AccountParamBuilder;
import com.tigerbrokers.stock.openapi.client.util.builder.TradeParamBuilder;
import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.logging.Logger;
import java.util.stream.Collectors;

public class Nasdaq100 {

  private static final ClientConfig clientConfig = ClientConfig.DEFAULT_CONFIG;
  private static final TigerHttpClient client;

  static {
    clientConfig.configFilePath = "./src/main/resources";
    client = TigerHttpClient.getInstance().clientConfig(clientConfig);
  }

  private static final int HISTORY_DAYS = 100;
  private static final int BATCH_SIZE = 50;
  private static final int REQUEST_SYMBOLS_SIZE = 50;
  private static final int MOMENTUM_PERIOD = 30;
  private static final int HOLDING_NUM = 5;
  private static final int ORDERS_CHECK_MAX_TIMES = 20;
  private static final double TARGET_OVERNIGHT_LIQUIDATION_RATIO = 0.6;

  private List<String> selectedSymbols = new ArrayList<>();
  private static final Logger logger = Logger.getLogger(Nasdaq100.class.getName());

  private final List<String> UNIVERSE_NDX = Arrays.asList(
      "AAPL", "ADBE", "ADI", "ADP", "ADSK", "AEP", "ALGN", "AMAT", "AMD", "AMGN", "AMZN",
      "ANSS", "ASML", "ATVI", "AVGO", "BIDU", "BIIB", "BKNG", "CDNS", "CDW", "CERN", "CHKP",
      "CHTR", "CMCSA", "COST", "CPRT", "CRWD", "CSCO", "CSX", "CTAS", "CTSH", "DLTR", "DOCU",
      "DXCM", "EA", "EBAY", "EXC", "FAST", "FB", "FISV", "FOX", "GILD", "GOOG", "HON", "IDXX",
      "ILMN", "INCY", "INTC", "INTU", "ISRG", "JD", "KDP", "KHC", "KLAC", "LRCX", "LULU", "MAR",
      "MCHP", "MDLZ", "MELI", "MNST", "MRNA", "MRVL", "MSFT", "MTCH", "MU", "NFLX", "NTES",
      "NVDA", "NXPI", "OKTA", "ORLY", "PAYX", "PCAR", "PDD", "PEP", "PTON", "PYPL", "QCOM",
      "REGN", "ROST", "SBUX", "SGEN", "SIRI", "SNPS", "SPLK", "SWKS", "TCOM", "TEAM", "TMUS",
      "TSLA", "TXN", "VRSK", "VRSN", "VRTX", "WBA", "WDAY", "XEL", "XLNX", "ZM");

  /** 运行策略 / Run strategy */
  public void run() throws InterruptedException {
    grabQuotePerm();
    screenStocks();
    rebalancePortfolio();
  }

  /** 选股：计算周期内涨跌幅，选取涨幅最高几只 / Screen: pick top momentum stocks */
  public void screenStocks() throws InterruptedException {
    Map<String, List<KlinePoint>> history = getHistory(UNIVERSE_NDX, HISTORY_DAYS, BATCH_SIZE);
    Map<String, Double> momentum = new HashMap<>();
    history.forEach((symbol, klinePoints) -> {
      int size = klinePoints.size();
      Double priceChange =
          (klinePoints.get(size - 1).getClose() - klinePoints.get(size - MOMENTUM_PERIOD).getClose())
              / klinePoints.get(size - MOMENTUM_PERIOD).getClose();
      momentum.put(symbol, priceChange);
    });
    Map<String, Double> sortedMap = momentum.entrySet().stream()
        .sorted(Map.Entry.comparingByValue(Comparator.reverseOrder()))
        .limit(HOLDING_NUM)
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (e1, e2) -> e1, LinkedHashMap::new));
    selectedSymbols = new ArrayList<>(sortedMap.keySet());
    logger.info("selected symbols:" + selectedSymbols);
  }

  /** 调仓 / Rebalance portfolio */
  public void rebalancePortfolio() throws InterruptedException {
    if (selectedSymbols.isEmpty()) {
      logger.warning("no selected symbols, strategy exit!");
      return;
    }
    closePosition();
    openPosition(getAdjustValue());
  }

  /** 获取调仓金额 / Get rebalance amount */
  private Double getAdjustValue() {
    PrimeAssetItem.Segment asset = getPrimeAsset();
    double targetOvernightLiquidation = asset.getEquityWithLoan() * TARGET_OVERNIGHT_LIQUIDATION_RATIO;
    return asset.getOvernightLiquidation() - targetOvernightLiquidation;
  }

  /** 平仓未入选股票 / Close positions not selected */
  private void closePosition() throws InterruptedException {
    Map<String, Long> positions = getPositions();
    Set<String> needCloseSymbols = positions.keySet();
    for (String selectedSymbol : selectedSymbols) {
      needCloseSymbols.remove(selectedSymbol);
    }
    if (!needCloseSymbols.isEmpty()) {
      Map<String, Double> latestPrice = getQuote(new ArrayList<>(needCloseSymbols));
      List<TradeOrderRequest> orderRequests = new ArrayList<>();
      needCloseSymbols.forEach(symbol -> {
        ContractItem contract = ContractItem.buildStockContract(symbol, Currency.USD.name());
        TradeOrderRequest request =
            // buildLimitOrder 的整股重载收 Integer；小数持仓需用 (Long, Integer quantityScale) 重载
            TradeOrderRequest.buildLimitOrder(contract, ActionType.SELL,
                positions.get(symbol).intValue(), latestPrice.get(symbol));
        orderRequests.add(request);
      });
      executeOrders(orderRequests);
    }
  }

  /** 开仓入选股票 / Open positions for selected stocks */
  private void openPosition(Double adjustValue) throws InterruptedException {
    double adjustValuePerStock = adjustValue / HOLDING_NUM;
    if (adjustValue <= 0) {
      logger.info("no enough liquidation");
      return;
    }
    Map<String, Double> latestPrice = getQuote(selectedSymbols);
    List<TradeOrderRequest> orders = new ArrayList<>();
    for (String symbol : selectedSymbols) {
      int quantity = (int) (adjustValuePerStock / latestPrice.get(symbol));
      if (quantity == 0) continue;
      ContractItem contract = ContractItem.buildStockContract(symbol, Currency.USD.name());
      TradeOrderRequest request =
          TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, quantity, latestPrice.get(symbol));
      orders.add(request);
    }
    executeOrders(orders);
  }

  /** 执行订单 / Execute orders */
  private void executeOrders(List<TradeOrderRequest> orderRequests) throws InterruptedException {
    orderRequests.forEach(order -> {
      TradeOrderResponse response = client.execute(order);
      logger.info("place order: " + response);
    });
    sleep(20000);
    int i = 0;
    while (i <= ORDERS_CHECK_MAX_TIMES) {
      JSONArray openOrders = getOrders(MethodName.ACTIVE_ORDERS);
      if (openOrders.isEmpty()) break;
      if (i >= ORDERS_CHECK_MAX_TIMES) {
        for (int k = 0; k < openOrders.size(); ++k) {
          cancelOrder(openOrders.getJSONObject(k).getLong("id"));
        }
      }
      i++;
      sleep(20000);
    }
  }

  /** 撤单 / Cancel order */
  private void cancelOrder(Long id) {
    TigerHttpRequest request = new TigerHttpRequest(MethodName.CANCEL_ORDER);
    String bizContent = TradeParamBuilder.instance().account(clientConfig.defaultAccount).id(id).buildJson();
    request.setBizContent(bizContent);
    client.execute(request);
  }

  /** 获取订单 / Get orders */
  private JSONArray getOrders(MethodName apiServiceType) {
    TigerHttpRequest request = new TigerHttpRequest(apiServiceType);
    String bizContent = AccountParamBuilder.instance()
        .account(clientConfig.defaultAccount)
        .startDate(Instant.now().minus(1, ChronoUnit.DAYS).toEpochMilli())
        .endDate(Instant.now().toEpochMilli())
        .secType(SecType.STK).market(Market.US).isBrief(false).buildJson();
    request.setBizContent(bizContent);
    TigerHttpResponse response = client.execute(request);
    return JSON.parseObject(response.getData()).getJSONArray("items");
  }

  /** 获取综合/模拟账户资产 / Get prime assets */
  private PrimeAssetItem.Segment getPrimeAsset() {
    PrimeAssetRequest assetRequest = PrimeAssetRequest.buildPrimeAssetRequest(clientConfig.defaultAccount);
    PrimeAssetResponse response = client.execute(assetRequest);
    return response.getSegment(Category.S);
  }

  /** 获取持仓 / Get positions */
  private Map<String, Long> getPositions() {
    Map<String, Long> result = new HashMap<>();
    // 持仓有专用请求类，返回强类型对象，无需手工解析 JSON
    PositionsRequest request = new PositionsRequest();
    String bizContent = AccountParamBuilder.instance()
        .account(clientConfig.defaultAccount).market(Market.US).secType(SecType.STK).buildJson();
    request.setBizContent(bizContent);
    PositionsResponse response = client.execute(request);
    if (!response.isSuccess() || response.getItem() == null) return result;
    for (PositionDetail pos : response.getItem().getPositions()) {
      // position 为 long，配合 positionScale 表示小数持仓
      result.put(pos.getSymbol(), pos.getPosition());
    }
    return result;
  }

  /** 抢占行情权限 / Grab quote permission */
  private void grabQuotePerm() {
    TigerHttpRequest request = new TigerHttpRequest(MethodName.GRAB_QUOTE_PERMISSION);
    request.setBizContent(AccountParamBuilder.instance().buildJson());
    TigerHttpResponse response = client.execute(request);
    logger.info("quote permissions: " + response.getData());
  }

  /** 获取实时行情 / Get real-time quotes */
  private Map<String, Double> getQuote(List<String> symbols) throws InterruptedException {
    Map<String, Double> quote = new HashMap<>();
    Collection<List<String>> partitions = partition(symbols, REQUEST_SYMBOLS_SIZE);
    for (List<String> part : partitions) {
      QuoteRealTimeQuoteResponse response = client.execute(QuoteRealTimeQuoteRequest.newRequest(part));
      if (response.isSuccess()) {
        response.getRealTimeQuoteItems().forEach(item -> quote.put(item.getSymbol(), item.getLatestPrice()));
      }
      sleep(1000);
    }
    return quote;
  }

  /** 获取历史行情 / Get historical klines */
  private Map<String, List<KlinePoint>> getHistory(List<String> symbols, int total, int batchSize)
      throws InterruptedException {
    Map<String, List<KlinePoint>> history = new HashMap<>();
    Collection<List<String>> partitions = partition(symbols, REQUEST_SYMBOLS_SIZE);
    for (List<String> part : partitions) {
      int i = 0;
      long endTime = Instant.now().toEpochMilli();
      while (i < total) {
        QuoteKlineRequest request = QuoteKlineRequest.newRequest(part, KType.day, -1L, endTime);
        request.withLimit(batchSize);
        QuoteKlineResponse response = client.execute(request);
        if (response.isSuccess()) {
          for (KlineItem item : response.getKlineItems()) {
            List<KlinePoint> points = history.getOrDefault(item.getSymbol(), new ArrayList<>());
            points.addAll(item.getItems());
            endTime = item.getItems().get(0).getTime();
            history.put(item.getSymbol(), points);
          }
        }
        i += batchSize;
        sleep(1000);
      }
    }
    Map<String, List<KlinePoint>> sorted = new HashMap<>();
    history.forEach((symbol, points) -> {
      points.sort(Comparator.comparingLong(KlinePoint::getTime));
      sorted.put(symbol, points);
    });
    return sorted;
  }

  static <T> Collection<List<T>> partition(List<T> list, int size) {
    final AtomicInteger counter = new AtomicInteger(0);
    return list.stream().collect(Collectors.groupingBy(s -> counter.getAndIncrement() / size)).values();
  }

  public static void main(String[] args) throws InterruptedException {
    new Nasdaq100().run();
  }
}
```

## 期权计算工具 / Option Calculation Tool

期权计算工具可用于希腊值计算、预测价格、隐含波动率计算，基于 `jquantlib` 库。
Option calc tool for Greeks, predicted price, implied volatility. Based on `jquantlib`.

> 不支持末日期权的价格预测 / Does not support expiry-day price prediction.

### 完整示例 / Full Example

```java
import com.alibaba.fastjson.JSONObject;
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.https.domain.option.item.OptionBriefItem;
import com.tigerbrokers.stock.openapi.client.https.domain.option.model.OptionCommonModel;
import com.tigerbrokers.stock.openapi.client.https.domain.quote.item.RealTimeQuoteItem;
import com.tigerbrokers.stock.openapi.client.https.request.option.OptionBriefQueryRequest;
import com.tigerbrokers.stock.openapi.client.https.request.quote.QuoteRealTimeQuoteRequest;
import com.tigerbrokers.stock.openapi.client.https.response.option.OptionBriefResponse;
import com.tigerbrokers.stock.openapi.client.https.response.quote.QuoteRealTimeQuoteResponse;
import com.tigerbrokers.stock.openapi.client.struct.OptionFundamentals;
import com.tigerbrokers.stock.openapi.client.struct.enums.Right;
import com.tigerbrokers.stock.openapi.client.struct.enums.TimeZoneId;
import com.tigerbrokers.stock.openapi.client.util.DateUtils;
import com.tigerbrokers.stock.openapi.client.util.OptionCalcUtils;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;

public class OptionCalcTool {

    public static void main(String[] args) {
        // 初始化客户端 (省略配置代码) / Init client (config omitted)
        TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(ClientConfig.DEFAULT_CONFIG);

        String symbol = "AAPL";
        Right optionRight = Right.PUT;
        String strike = "185.0";
        String settlementDate = "2024-02-05"; // 一般是当前日期 / Usually current date
        String expiryDate = "2024-02-09";
        long expiryTimestamp = DateUtils.parseEpochMill(expiryDate, TimeZoneId.NewYork);

        // 查询期权盘口价格和国债利率 / Query option brief and treasury rates
        OptionCommonModel model = new OptionCommonModel(symbol, optionRight.name(), strike, expiryTimestamp);
        OptionBriefResponse optionBriefResponse = client.execute(OptionBriefQueryRequest.of(model));
        OptionBriefItem optionBrief = optionBriefResponse.getOptionBriefItems().get(0);

        // 查询标的最新价格 / Get underlying latest price
        QuoteRealTimeQuoteResponse quoteResponse =
            client.execute(QuoteRealTimeQuoteRequest.newRequest(Arrays.asList(symbol)));
        Double latestPrice = quoteResponse.getRealTimeQuoteItems().get(0).getLatestPrice();

        // 注意：结算日期=过期日时，需改为前一天 / If settlement == expiry, use day before
        if (settlementDate.equals(expiryDate)) {
            settlementDate = LocalDate.parse(settlementDate).minusDays(1).toString();
        }

        // 美式期权计算 / American option calculation
        OptionFundamentals result = OptionCalcUtils.calcOptionIndex(
            optionRight, latestPrice, Double.valueOf(strike),
            optionBrief.getRatesBonds(), 0,
            optionBrief.getAskPrice(), optionBrief.getBidPrice(),
            LocalDate.parse(settlementDate), LocalDate.parse(expiryDate));

        // 欧式期权(指数期权)使用 / For European options (index options):
        // OptionCalcUtils.calcEuropeanOptionIndex(...)

        System.out.println(JSONObject.toJSONString(result));
    }
}
```

### 直接计算 / Direct Calculation

```java
import com.tigerbrokers.stock.openapi.client.struct.OptionFundamentals;
import com.tigerbrokers.stock.openapi.client.struct.enums.Right;
import com.tigerbrokers.stock.openapi.client.util.OptionCalcUtils;
import java.time.LocalDate;

OptionFundamentals result = OptionCalcUtils.calcOptionIndex(
    Right.CALL,
    243.35,     // 标的资产价格 / Underlying price
    240,        // 行权价格 / Strike price
    0.0241,     // 无风险利率(美国国债) / Risk-free rate (treasury)
    0,          // 股息率 / Dividend yield
    0.4648,     // 隐含波动率 / Implied volatility
    LocalDate.of(2022, 8, 12),  // 预测日期(< 到期日) / Prediction date
    LocalDate.of(2022, 8, 19)); // 到期日 / Expiry date

System.out.println("predictedValue: " + result.getPredictedValue());
System.out.println("delta: " + result.getDelta());
System.out.println("gamma: " + result.getGamma());
System.out.println("theta: " + result.getTheta());
System.out.println("vega: " + result.getVega());
System.out.println("rho: " + result.getRho());
```

输出示例 / Sample output:

```text
predictedValue: 8.09628869758489
delta: 0.6005809882595446
gamma: 0.024620648675961896
theta: -0.4429512650168838
vega: 0.13035018339603335
rho: 0.026470369092588868
```

## 更多示例 / More Examples

官方示例仓库 / Official demo repository:
https://github.com/tigerfintech/openapi-java-sdk-demo

## 注意事项 / Notes

- Java SDK 使用 **PKCS#8** 格式私钥 / Java SDK uses PKCS#8 format private key
- `TigerHttpClient` 和 `ClientConfig` 建议配置为全局静态变量复用 / Recommend as global static singletons
- 行情权限需单独购买，API 与 App 独立 / Quote permissions require separate purchase
- 同一 tiger_id 只能有一个 WebSocket 长连接 / Only one WebSocket connection per tiger_id
- 机构用户需额外配置 `secretKey` / Institutional users must set `secretKey`
- TBHK 牌照需配置 token，有效期 30 天，可配置自动刷新 / TBHK license needs token (30-day expiry, auto-refresh available)
- 官方文档 / Official docs: https://quant.itigerup.com/openapi/java/overview/introduction
