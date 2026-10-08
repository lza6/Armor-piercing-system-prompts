
# Tiger Open API Java SDK 交易 / Trading

> 中文 | English — 双语技能，代码通用。Bilingual skill with shared code examples.
> 官方文档 Docs: https://docs.itigerup.com/docs/place-order-java

## 安全规范 / Safety Rules

> **默认使用模拟账户。Default to Paper Trading.**

### 模拟交易 vs 实盘交易 / Paper vs Live Trading

| 特性 Feature | 模拟 Paper | 实盘 Live |
|------|------|------|
| 资金 Funds | 虚拟，无风险 Virtual, no risk | 真实资金 Real money |
| 账户类型 `accountType` | `PAPER` | `STANDARD` / `GLOBAL` |
| 默认 Default | **是** Yes | 需用户明确要求 User must explicitly request |

### 实盘下单工作流 / Live Order Workflow

当用户要求实盘交易时，**每步均为必须，缺少任何步骤不得下单** / Every step is mandatory — skip any and do NOT place the order:

1. **确认账户 Verify account**: 获取账户列表，筛选 `accountType != "PAPER"` 的实盘账户 / Get account list, filter for non-PAPER accounts
2. **预览订单 Preview**: 调用 `TradeOrderPreviewRequest` 查看预估佣金和保证金，将结果展示给用户 / Call preview, show estimated commission and margin to user
3. **等待用户明确确认 Wait for explicit confirmation**: 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止并等待用户明确回复**；未收到确认前**禁止执行下单** / Display order details in a table, **stop and wait**; do NOT submit until user explicitly confirms
4. **执行下单 Execute**: 用户确认后执行下单 / Place the order only after confirmation
5. **检查状态 Check status**: 下单返回成功仅表示提交，需查询订单确认成交 / Submission success ≠ execution; query order to confirm fill

---

## 初始化 / Initialize

```java
import com.tigerbrokers.stock.openapi.client.https.client.TigerHttpClient;
import com.tigerbrokers.stock.openapi.client.config.ClientConfig;

TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(ClientConfig.DEFAULT_CONFIG);
```

### 获取账户列表 / Get Managed Accounts

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.ACCOUNTS);
String bizContent = AccountParamBuilder.instance()
        .account("123456")
        .buildJsonWithoutDefaultAccount();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);

JSONArray accounts = JSON.parseObject(response.getData()).getJSONArray("items");
JSONObject account1 = accounts.getJSONObject(0);
String account = account1.getString("account");
String capability = account1.getString("capability");
String accountType = account1.getString("accountType");
```

**返回字段 / Response Fields:**

| 字段 Field | 说明 Description |
|-----------|-----------------|
| account | 交易账户 Trading account (综合账号5-10位数字, 环球账号U开头, 模拟账号17位数字) |
| capability | 账户类型 Account type (CASH/RegTMargin/PMGRN) |
| status | 账号状态 Status (Funded/Open/Pending/Rejected/Closed) |
| accountType | 账户分类 Category (GLOBAL:环球/STANDARD:综合/PAPER:模拟) |

---

## 合约 / Contracts

交易前必须获取合约对象。Contracts must be obtained before trading.

### 合约介绍 / Contract Introduction

合约是指交易的买卖对象或者标的物。大多数合约可通过以下基础属性唯一确定:
- 标的代码 symbol: 如 TIGR, 00700
- 合约类型 sec_type: STK(股票), OPT(期权), FUT(期货), WAR(窝轮), IOPT(牛熊证), FUND(基金), CC(数字货币)
- 货币类型 currency: USD, HKD, CNH
- 交易所 exchange: STK类型一般不需要, 期货需要

### 构建合约对象 / Build Contract Objects

```java
// 美股股票合约 / US Stock
ContractItem contract = ContractItem.buildStockContract("SPY", "USD");

// 港股股票合约 / HK Stock
ContractItem contract = ContractItem.buildStockContract("00700", "HKD");

// 美股期权合约 / US Option (两种方式)
ContractItem contract = ContractItem.buildOptionContract("AAPL  190118P00160000");
ContractItem contract = ContractItem.buildOptionContract("AAPL", "20211119", 150.0D, "CALL");

// 港股窝轮合约 / HK Warrant (需要注意同一个symbol，环球账号和综合账号的expiry可能不同)
ContractItem contract = ContractItem.buildWarrantContract("13745", "20211217", 719.38D, Right.CALL.name());

// 港股牛熊证合约 / HK CBBC
ContractItem contract = ContractItem.buildCbbcContract("50296", "20220331", 457D, Right.CALL.name());

// 期货合约 / Future
// 环球账户 / Global account
ContractItem contract = ContractItem.buildFutureContract("CL", "USD", "SGX", "20190328", 1.0D);
// 综合账户 / Standard account
ContractItem contract = ContractItem.buildFutureContract("CL2112", "USD");

// 基金合约 / Fund
ContractItem contract = ContractItem.buildFundContract("IE00B464Q616.USD", "USD");
```

也可以手动设置属性构建 / Manual construction:

```java
// 股票 / Stock
ContractItem contract = new ContractItem();
contract.setSymbol("TIGR");
contract.setSecType("STK");
contract.setCurrency("USD"); // 非必填，下单时默认为USD
contract.setMarket("US");    // 非必填

// 期权 / Option
ContractItem contract = new ContractItem();
contract.setSymbol("AAPL");
contract.setSecType("OPT");
contract.setCurrency("USD");
contract.setExpiry("20180821");
contract.setStrike(30D);
contract.setRight("CALL");
contract.setMultiplier(100.0D);

// 期货 / Future
ContractItem contract = new ContractItem();
contract.setSymbol("CL1901");
contract.setSecType("FUT");
contract.setExchange("SGX");
contract.setCurrency("USD");
contract.setExpiry("20190328");
contract.setMultiplier(1.0D);

// 数字货币 / Crypto
ContractItem contract = new ContractItem();
contract.setSymbol("BTC/USD");    // actual crypto symbol
contract.setSecType("CC");         // security type = Cryptocurrency

// 基金 / Fund
ContractItem contract = new ContractItem();
contract.setSymbol("IE00B11XZ988.USD");
contract.setSecType("FUND");
```

### 获取单个合约信息 / Get Single Contract

**对应的请求类：ContractRequest**

获取交易需要的合约信息。需要注意环球账户、综合账户返回 `ContractItem` 字段值数目会不同，建议获取合约和下单使用相同的账户。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 如：572386 |
| symbol | string | Yes | 股票代码 如：00700/AAPL |
| sec_type | string | Yes | STK/OPT/FUT/CC |
| currency | string | No | USD/HKD/CNH |
| expiry | string | No | 到期日 (期权时必传) yyyyMMdd |
| strike | double | No | 行权价 (期权时必传) |
| right | string | No | CALL/PUT (期权时必传) |
| exchange | string | No | 交易所 (美股 SMART 港股 SEHK 沪港通 SEHKNTL 深港通 SEHKSZSE) |
| secret_key | string | No | 交易员密钥，机构用户专用 |

```java
// 获取股票合约 / Get stock contract
ContractRequest contractRequest = ContractRequest.newRequest(new ContractModel("AAPL"));
ContractResponse contractResponse = client.execute(contractRequest);
ContractItem contract = contractResponse.getItem();

// 获取期权合约 / Get option contract
ContractModel model = new ContractModel("AAPL", SecType.OPT.name(), Currency.USD.name(),
    "20211126", 150D, Right.CALL.name());
contractRequest = ContractRequest.newRequest(model);
contractResponse = client.execute(contractRequest);

// 获取窝轮合约 / Get warrant contract
ContractModel contractModel = new ContractModel("13745", SecType.WAR.name());
contractModel.setStrike(719.38D);
contractModel.setRight(Right.CALL.name());
contractModel.setExpiry("20211223");
ContractRequest contractRequest = ContractRequest.newRequest(contractModel);
ContractResponse contractResponse = client.execute(contractRequest);

// 获取期货合约 / Get future contract
ContractRequest contractRequest = ContractRequest.newRequest(
        new ContractModel("JPY2306", SecType.FUT.name()), "572386");
ContractResponse contractResponse = client.execute(contractRequest);
```

**ContractItem 返回字段 / Response Fields:**

| 名称 Field | 说明 Description |
|-----------|-----------------|
| identifier | 唯一标识 Unique ID (股票=symbol, 期权=21位标识符, 期货=如CL2109) |
| symbol | 股票代码 Symbol |
| secType | STK/OPT/FUT/WAR/IOPT 等 |
| name | 名称 Name |
| localSymbol | 环球账户专有, 港股用于识别窝轮和牛熊证 |
| currency | USD/HKD/CNH |
| exchange | 交易所 Exchange |
| primaryExchange | 上市交易所 Primary exchange |
| market | 市场 US/HK/CN |
| expiry | 期权和期货专有, 过期日 |
| contractMonth | 期货专有, 合约交割月份 |
| right | 期权专有, CALL/PUT |
| strike | 期权专有, 行权价 |
| multiplier | 期权和期货专有, 乘数 |
| lotSize | 每手数量 Lot size |
| minTick | 最小报价单位 Min tick |
| tickSizes | 最小报价单位价格区间 (begin, end, tickSize, type) |
| marginable | 是否可融资 |
| shortable | 能否做空 |
| longInitialMargin | 做多初始保证金 |
| longMaintenanceMargin | 做多维持保证金 |
| shortInitialMargin | 做空初始保证金比例 |
| shortMaintenanceMargin | 做空维持保证金比例 (综合账号有值) |
| shortableCount | 可做空数目 |
| shortFeeRate | 做空费率 |
| tradeable | 是否可交易 (仅STK) |
| supportOvernightTrading | 是否支持夜盘交易 (仅美股) |
| supportFractionalShare | 是否支持碎股交易 (仅综合/模拟) |
| isEtf | 是否ETF |
| etfLeverage | ETF杠杆倍数 |
| continuous | 期货专有, 是否连续合约 |
| lastTradingDate | 期货专有, 最后交易日 |
| firstNoticeDate | 期货专有, 第一通知日 |

### 获取多个合约信息 / Get Multiple Contracts

**对应的请求类：ContractsRequest**

获取交易需要的合约信息，只支持STK和FUT。综合账户返回的批量合约里没有可做空数量和保证金字段。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| symbols | `List<String>` | Yes | 股票代码列表, 单次上限50 |
| sec_type | string | Yes | STK/FUT/CC |
| currency | string | No | USD/HKD/CNH |

```java
List<String> symbols = new ArrayList<>();
symbols.add("AAPL");
symbols.add("TSLA");
ContractsModel models = new ContractsModel(symbols, SecType.STK.name());
ContractsRequest contractsRequest = ContractsRequest.newRequest(models, "13810712");
ContractsResponse contractsResponse = client.execute(contractsRequest);
```

### 获取期权/窝轮/牛熊证合约列表 / Get Derivative Contracts

**对应的请求类：QuoteContractRequest**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| symbol | string | Yes | 股票代码 |
| sec_type | string | Yes | OPT/WAR/IOPT |
| expiry | String | No | 到期日 yyyyMMdd (OPT必须有值) |
| lang | string | No | en_US/zh_CN/zh_TW |

```java
QuoteContractResponse response = client.execute(
    QuoteContractRequest.newRequest("00700", SecType.WAR, "20211223"));
if (response.isSuccess()) {
    System.out.println(response.getContractItems());
}
```

---

## 下单 / Place Orders
<!-- 当用户提到 "买入"、"卖出"、"下单"、"buy"、"sell"、"order"、"place order" 时 -->

**对应的请求类：TradeOrderRequest**

交易下单接口。请在运行程序前检查您的账户是否支持所请求的订单类型，并检查交易规则是否允许在程序运行时段对特定标的下单。

> **注意事项 / CAUTION:**
> 1. 市价单（MKT）和止损单（STP）不支持盘前盘后阶段交易, outside_rth 需设为 false
> 2. 可做空标的不支持锁仓, 无法同时持有多头与空头头寸
> 3. 附加订单的主订单类型仅支持限价单
> 4. 限价价格不匹配 tickSize 时, 可参考合约返回 tickSizes 字段, 利用 StockPriceUtils 工具类修复价格
> 5. 市价单（MKT）和模拟账号不支持 time_in_force 设置为 GTC
> 6. 模拟账号暂不支持窝轮和牛熊证的订单
> 7. 禁止直接开立反向头寸, 需先平掉现有头寸再进行新操作

### 下单参数总览 / Order Parameters Overview

| 参数 Parameter | 类型 Type | 描述 Description | MKT | LMT | STP | STP_LMT | TRAIL |
|---------------|---------|-----------------|-----|-----|-----|---------|-------|
| account | string | 用户授权账户 | 必填 | 必填 | 必填 | 必填 | 必填 |
| symbol | string | 股票代码 | 必填 | 必填 | 必填 | 必填 | 必填 |
| sec_type | string | 合约类型 (STK/OPT/WAR/IOPT/FUT/FUND/CC) | 必填 | 必填 | 必填 | 必填 | 必填 |
| action | string | 交易方向 BUY/SELL | 必填 | 必填 | 必填 | 必填 | 必填 |
| order_type | string | 订单类型 | MKT | LMT | STP | STP_LMT | TRAIL |
| total_quantity | long | 下单数量 | 必填 | 必填 | 必填 | 必填 | 必填 |
| limit_price | double | 限价 | - | 必填 | - | 必填 | - |
| aux_price | double | 止损触发价/跟踪额 | - | - | 必填 | 必填 | 选填 |
| trailing_percent | double | 跟踪止损百分比 (与aux_price互斥, 优先) | - | - | - | - | 选填 |
| outside_rth | boolean | 允许盘前盘后 (美股, 默认true) | - | 选填 | 选填 | - | 选填 |
| trading_session_type | TradingSessionType | 美股订单时段 (仅限价单) | - | 选填 | - | 选填 | - |
| time_in_force | string | DAY/GTC/GTD, 默认DAY | 选填 | 选填 | 选填 | 选填 | 选填 |
| expire_time | long | GTD截止时间 (13位时间戳) | - | 选填 | 选填 | - | 选填 |
| total_quantity_scale | int | 下单数量偏移量 (碎股用) | 选填 | 选填 | 选填 | 选填 | 选填 |
| cash_amount | Double | 订单金额 (基金等金额订单) | 选填 | - | - | - | - |
| adjust_limit | double | 价格微调幅度 (0.001=向上0.1%, -0.001=向下0.1%) | - | 选填 | 选填 | 选填 | 选填 |
| market | string | 市场 (US/HK/CN) | 选填 | 选填 | 选填 | 选填 | 选填 |
| currency | string | 货币 (USD/HKD/CNH) | 选填 | 选填 | 选填 | 选填 | 选填 |
| user_mark | string | 下单备注信息, 下单后不可修改 | 选填 | 选填 | 选填 | 选填 | 选填 |
| expiry | string | 过期日 (期权/窝轮/牛熊证) | 选填 | 选填 | 选填 | 选填 | 选填 |
| strike | string | 行权价 (期权/窝轮/牛熊证) | 选填 | 选填 | 选填 | 选填 | 选填 |
| right | string | PUT/CALL (期权/窝轮/牛熊证) | 选填 | 选填 | 选填 | 选填 | 选填 |
| secret_key | string | 交易员密钥, 机构用户专用 | 选填 | 选填 | 选填 | 选填 | 选填 |

**返回 / Response:**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|---------|-----------------|
| id | long | 唯一单号ID, 可用于查询/修改/取消订单 |
| subIds | `List<Long>` | 附加单时, 返回子订单号ID列表 |
| orders | `List<TradeOrder>` | 订单详细信息 |

### 市价单 MKT / Market Order

```java
// 获取合约 (使用默认账户) / Get contract (default account)
ContractRequest contractRequest = ContractRequest.newRequest(new ContractModel("AAPL"));
ContractResponse contractResponse = client.execute(contractRequest);
ContractItem contract = contractResponse.getItem();

// 市价单 (使用默认账户) / Market order (default account)
TradeOrderRequest request = TradeOrderRequest.buildMarketOrder(contract, ActionType.BUY, 10);
TradeOrderResponse response = client.execute(request);
System.out.println(JSONObject.toJSONString(response));

// 市价单 (指定账户) / Market order (specific account)
request = TradeOrderRequest.buildMarketOrder("402901", contract, ActionType.BUY, 10);
response = client.execute(request);
```

### 限价单 LMT / Limit Order

```java
// 使用默认账户 / Default account
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, 1, 100.0d);
TradeOrderResponse response = client.execute(request);

// 指定账户 + userMark + GTD / Specific account + userMark + GTD
request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 100.0d);
request.setUserMark("test001");
request.setTimeInForce(TimeInForce.GTD);
request.setExpireTime(1669363583804L);
response = client.execute(request);
```

### 夜盘/全时段订单 / Overnight/Full-time Order

仅支持美股 / US only

```java
// 夜盘订单 / Overnight order
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 200.0d);
request.setTradingSessionType(TradingSessionType.OVERNIGHT);
TradeOrderResponse response = client.execute(request);

// 全时段订单 / Full-time order
request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 200.0d);
request.setTradingSessionType(TradingSessionType.FULL);
response = client.execute(request);
```

### 竞价单 AM/AL / Auction Order

港股竞价时段 / HK Auction session

```java
// 竞价限价单 / Auction limit order
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 100, 100.0d);
// 盘前竞价: AM or AL + OPG; 盘后竞价: AM or AL + DAY
request.setAuctionOrder(OrderType.AL, TimeInForce.OPG);
TradeOrderResponse response = client.execute(request);

// 竞价市价单 / Auction market order
TradeOrderRequest marketAuctionRequest =
    TradeOrderRequest.buildMarketOrder("402901", contract, ActionType.BUY, 100);
marketAuctionRequest.setAuctionOrder(OrderType.AM, TimeInForce.OPG);
TradeOrderResponse marketAuctionResponse = client.execute(marketAuctionRequest);
```

### 止损单 STP / Stop Order

```java
// 默认账户 / Default account
TradeOrderRequest request = TradeOrderRequest.buildStopOrder(contract, ActionType.BUY, 1, 120.0d);
TradeOrderResponse response = client.execute(request);

// 指定账户 / Specific account
request = TradeOrderRequest.buildStopOrder("402901", contract, ActionType.BUY, 1, 120.0d);
response = client.execute(request);
```

### 止损限价单 STP_LMT / Stop Limit Order

```java
// buildStopLimitOrder(contract, action, quantity, limitPrice, auxPrice)
TradeOrderRequest request = TradeOrderRequest.buildStopLimitOrder(contract, ActionType.BUY, 1, 150d, 130.0d);
TradeOrderResponse response = client.execute(request);

// 指定账户 / Specific account
request = TradeOrderRequest.buildStopLimitOrder("402901", contract, ActionType.BUY, 1, 150d, 130.0d);
response = client.execute(request);
```

### 跟踪止损单 TRAIL / Trailing Stop Order

```java
// buildTrailOrder(contract, action, quantity, trailingPercent, auxPrice)
TradeOrderRequest request = TradeOrderRequest.buildTrailOrder(contract, ActionType.BUY, 1, 10d, 130.0d);
TradeOrderResponse response = client.execute(request);

// 指定账户 (综合账户暂不支持) / Specific account (standard account currently not supported)
request = TradeOrderRequest.buildTrailOrder("402901", contract, ActionType.BUY, 1, 10d, 130.0d);
response = client.execute(request);
```

### 主订单+附加止盈单 / Primary + Profit Taker

```java
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, 1, 199d);
TradeOrderRequest.addProfitTakerOrder(request, 250D, TimeInForce.DAY, Boolean.FALSE);
TradeOrderResponse response = client.execute(request);

// 指定账户 / Specific account
request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 199d);
TradeOrderRequest.addProfitTakerOrder(request, 250D, TimeInForce.DAY, Boolean.FALSE);
response = client.execute(request);
```

### 主订单+附加止损单 / Primary + Stop Loss

```java
// 附加止损市价单 (不支持期权标的) / Attached stop loss market order (not for options)
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 129d);
TradeOrderRequest.addStopLossOrder(request, 100D, TimeInForce.DAY);
TradeOrderResponse response = client.execute(request);

// 期权可以使用附加止损限价单 / Options can use stop loss limit order
ContractItem optionContract = ContractItem.buildOptionContract("AAPL", "20211231", 175.0D, "CALL");
request = TradeOrderRequest.buildLimitOrder("402901", optionContract, ActionType.BUY, 1, 2.0d);
// 第一个价格是触发价, 第二个是挂单限价 (暂只支持综合账号)
TradeOrderRequest.addStopLossLimitOrder(request, 1.7D, 1.69D, TimeInForce.DAY);
response = client.execute(request);
```

### 主订单+附加跟踪止损单 / Primary + Trailing Stop Loss

```java
ContractItem contract = ContractItem.buildStockContract("AAPL", "USD");
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder("402901", contract, ActionType.BUY, 1, 165D);
// addStopLossTrailOrder(request, trailingPercent, trailingAmount, timeInForce)
TradeOrderRequest.addStopLossTrailOrder(request, 10.0D, null, TimeInForce.DAY);
TradeOrderResponse response = client.execute(request);
```

### 主订单+附加括号订单 / Primary + Bracket Order

```java
TradeOrderRequest request = TradeOrderRequest.buildLimitOrder(contract, ActionType.BUY, 1, 199d);
// addBracketsOrder(request, profitPrice, profitTif, profitOutsideRth, stopLossPrice, stopLossTif)
TradeOrderRequest.addBracketsOrder(request, 250D, TimeInForce.DAY, Boolean.FALSE, 180D, TimeInForce.GTC);
TradeOrderResponse response = client.execute(request);
```

### 附加订单参数 / Attached Order Parameters

附加订单（Attached Order）是指通过附加子订单对主订单起到止盈或止损效果的订单。主订单类型必须为限价单。

| 参数 Parameter | 类型 Type | 描述 Description | 附加止损 | 附加止盈 | 附加跟踪止损 | 附加括号 |
|---------------|---------|-----------------|--------|--------|----------|--------|
| attach_type | string | 附加类型: PROFIT/LOSS/BRACKETS | 必填 | 必填 | 必填 | 必填 |
| profit_taker_price | double | 止盈价格 | - | 必填 | - | 必填 |
| profit_taker_tif | string | 止盈有效期 DAY/GTC | - | 必填 | - | 必填 |
| profit_taker_rth | boolean | 同outside_rth | - | 必填 | - | 必填 |
| stop_loss_price | double | 止损触发价 | 必填 | - | - | 必填 |
| stop_loss_limit_price | double | 止损限价 (暂只对综合账号有效) | 选填 | - | - | 选填 |
| stop_loss_tif | string | 止损有效期 DAY/GTC | 必填 | - | 必填 | 必填 |
| stop_loss_trailing_percent | double | 跟踪止损百分比 | 选填 | - | 二选一 | 选填 |
| stop_loss_trailing_amount | double | 跟踪止损额 | 选填 | - | 二选一 | 选填 |

### OCA括号单 / OCA Brackets Order

模拟盘不支持。OCA括号订单内的两个订单标的相同，一个止盈限价单，另一个为止损单或者止损限价单。其中一个成交时，自动取消另一个订单。

```java
ContractItem contract = ContractItem.buildStockContract("BILI", "USD");
// 带 account 的重载签名（12 参数）：
// (account, contract, action, Long quantity, Integer quantityScale,
//  profitTakerPrice, profitTakerTif, profitTakerRth,
//  stopLossPrice, stopLossLimitPrice, stopLossTif, stopLossRth)
// quantityScale 是 2024-03 碎股支持时新增的必填位（不需要时传 null）
// quantityScale was added for fractional shares; pass null when unused
TradeOrderRequest request = TradeOrderRequest.buildOCABracketsOrder(
        "13810712", contract, ActionType.SELL, 1L, null,
        17.0D, TimeInForce.DAY, Boolean.TRUE,    // 止盈: price, tif, outsideRth
        12.0D, null, TimeInForce.DAY, Boolean.FALSE); // 止损: price, limitPrice, tif, outsideRth
request.setLang(Language.en_US).setUserMark("test-oca");

TradeOrderResponse response = client.execute(request);
if (response.isSuccess()) {
    List<TradeOrder> ocaOrders = response.getItem().getOrders();
}
```

### TWAP/VWAP 算法单 / Algo Orders

只支持美股股票，只支持盘中下单。不能改单，可以撤单。

```java
// TWAP order
TradeOrderRequest twapRequest = TradeOrderRequest.buildTWAPOrder(
    "572386", "DM", ActionType.BUY, 500,
    DateUtils.getTimestamp("2023-06-20 09:30:00", TimeZoneId.NewYork),
    DateUtils.getTimestamp("2023-06-20 11:00:00", TimeZoneId.NewYork),
    1.5D)  // limitPriceOffset
  .setUserMark("testTWAP001")
  .setLang(Language.en_US);
TradeOrderResponse twapResponse = client.execute(twapRequest);

// VWAP order
TradeOrderRequest vwapRequest = TradeOrderRequest.buildVWAPOrder(
    "572386", "DM", ActionType.BUY, 500,
    DateUtils.getTimestamp("2023-06-20 09:30:00", TimeZoneId.NewYork),
    DateUtils.getTimestamp("2023-06-20 11:00:00", TimeZoneId.NewYork),
    0.5D,   // participationRate
    1.5D)   // limitPriceOffset
  .setUserMark("testVWAP001")
  .setLang(Language.en_US);
TradeOrderResponse vwapResponse = client.execute(vwapRequest);
```

**TWAP/VWAP 算法参数 / Algo Parameters:**

| 参数 Parameter | 类型 Type | 描述 Description | TWAP | VWAP |
|---------------|---------|-----------------|------|------|
| order_type | string | TWAP/VWAP | 必填 | 必填 |
| account | string | 资金账号 | 必填 | 必填 |
| symbol | string | 股票代码 | 必填 | 必填 |
| sec_type | string | 只支持STK | 必填 | 必填 |
| total_quantity | long | 订单数量 | 必填 | 必填 |
| algo_params.start_time | long | 策略开始时间(时间戳) | 选填 | 选填 |
| algo_params.end_time | long | 策略结束时间(时间戳) | 选填 | 选填 |
| algo_params.participation_rate | string | 最大参与率(0.01-0.5) | - | 选填 |

### 冰山单 ICEBERG / Iceberg Order

大单拆分为小单分批展示，减少市场冲击。SDK 2.5.1 起支持。
Splits a large order into smaller visible slices. Added in SDK 2.5.1.

```java
ContractItem icebergContract = ContractItem.buildStockContract("AAPL", "USD");

// 基础形式：(account, contract, action, quantity, limitPrice, displaySize)
// 默认 minDisplaySize = displaySize、priceType = PriceType.LIMIT_PRICE
TradeOrderRequest icebergBasic = TradeOrderRequest.buildIcebergOrder(
        account, icebergContract, ActionType.BUY, 1000, 180.0D, 100);
TradeOrderResponse icebergBasicResp = client.execute(icebergBasic);

// 完整形式：额外指定 minDisplaySize / checkIntervals / priceType / startTime / endTime
TradeOrderRequest icebergFull = TradeOrderRequest.buildIcebergOrder(
        account, icebergContract, ActionType.BUY, 1000, 180.0D,
        100,                       // displaySize 每次展示数量
        50,                        // minDisplaySize 最小展示数量
        30,                        // checkIntervals 检查间隔（秒）
        PriceType.LIMIT_PRICE,     // 价格类型
        null,                      // startTime（epoch ms，可为 null）
        null);                     // endTime（epoch ms，可为 null）
TradeOrderResponse icebergFullResp = client.execute(icebergFull);
```

`PriceType` 枚举（`struct.enums.PriceType`）共 4 个值 / 4 values:
`LIMIT_PRICE`（限价）、`ASK_PRICE`（卖一价）、`BID_PRICE`（买一价）、`LATEST_PRICE`（最新价）。

> ⚠️ **不存在 `IcebergPriceType` 枚举**，也没有 `OPPONENT_PRICE` 值；
> 构建方法在 `TradeOrderRequest` 上，不在 `TradeOrderModel` 上。
> （SDK CHANGELOG 在这三点上均有误，以源码为准。）
> There is no `IcebergPriceType` enum and no `OPPONENT_PRICE` value; the builder
> lives on `TradeOrderRequest`. The SDK CHANGELOG is wrong on all three points.

冰山单改单：重新用 `buildIcebergOrder` 构造，再改 methodName 与 id / To modify:

```java
ContractItem icebergModifyContract = ContractItem.buildStockContract("AAPL", "USD");
TradeOrderRequest icebergModify = TradeOrderRequest.buildIcebergOrder(
        account, icebergModifyContract, ActionType.BUY, 1000, 181.0D, 100);
icebergModify.setApiMethodName(MethodName.MODIFY_ORDER);
((TradeOrderModel) icebergModify.getApiModel()).setId(12345678901234567L);
TradeOrderResponse icebergModifyResp = client.execute(icebergModify);
```

---

### 期权提前行权 / Option Early Exercise

SDK 2.5.0 起支持。所有请求类的构造函数是 private，必须用静态工厂方法。
Added in SDK 2.5.0. All request classes have private ctors — use the factories.

```java
long optContractId = 123456789L;

// 1. 行权检验（预估行权后持仓变化）/ Check
OptionExerciseCheckRequest checkReq = OptionExerciseCheckRequest.buildRequest(
        account, optContractId, OptionExerciseType.Exercise);
OptionExerciseCheckResponse checkResp = client.execute(checkReq);
OptionExerciseCheckItem checkItem = checkResp.getItem();

// 放弃行权时可传价内率阈值（0-10，仅 Expire 适用）
OptionExerciseCheckRequest expireCheck = OptionExerciseCheckRequest
        .buildRequest(account, optContractId, OptionExerciseType.Expire)
        .setItmRate(5);

// 2. 查询可行权持仓 / Exercisable positions
OptionExercisePositionResponse posResp = client.execute(
        OptionExercisePositionRequest.buildRequest(account, OptionExerciseType.Exercise));
OptionExercisePositionPageItem posPage = posResp.getItem();

// 3. 提交行权 / Submit exercise
OptionExerciseSubmitResponse submitResp = client.execute(
        OptionExerciseSubmitRequest.buildExerciseRequest(
                account, optContractId, 1.0D, "2026-08-21", Boolean.FALSE));

// 提交放弃行权 / Submit expire
OptionExerciseSubmitResponse expireResp = client.execute(
        OptionExerciseSubmitRequest.buildExpireRequest(account, optContractId, 1.0D, 5));

// 4. 查询行权申请记录（分页）/ Records
OptionExerciseRecordResponse recordResp = client.execute(
        OptionExerciseRecordRequest.buildRequest(account, 1, 10));
OptionExerciseRecordPageItem recordPage = recordResp.getItem();

// 5. 撤销行权申请 / Cancel
Long exerciseRecordId = 987654321L;
OptionExerciseCancelResponse cancelExResp = client.execute(
        OptionExerciseCancelRequest.buildRequest(account, exerciseRecordId));
```

`OptionExerciseType` 枚举只有两个值，**注意大小写**（`.name()` 直接上报服务端）：
`Exercise`（提前行权）、`Expire`（提前放弃行权）。不是 `EXERCISE` / `EXPIRE`。
The enum values are `Exercise` / `Expire` — mixed case is load-bearing.

`OptionExerciseSubmitResponse` 与 `OptionExerciseCancelResponse` **没有 item 字段**，
成功时只需检查 `isSuccess()`。
Those two responses carry no business data — just check `isSuccess()`.

> ⚠️ SDK CHANGELOG 提到的 `OptionExercisePageRequest` 不存在，
> 实际类名是 `OptionExerciseRecordRequest`。
> `OptionExercisePageRequest` (per the CHANGELOG) does not exist.

---

### 期权多腿订单 / Multi-Leg Option Order

```java
List<ContractLeg> contractLegs = new ArrayList<>();
ContractLeg leg1 = new ContractLeg(SecType.OPT, "AAPL",
    "170.0", "20231013", Right.CALL,
    ActionType.BUY, 1);
contractLegs.add(leg1);
ContractLeg leg2 = new ContractLeg(SecType.OPT, "AAPL",
    "170.0", "20231013", Right.PUT,
    ActionType.BUY, 1);
contractLegs.add(leg2);

TradeOrderRequest request = TradeOrderRequest.buildMultiLegOrder(
    "572386", contractLegs, ComboType.CUSTOM,
    ActionType.BUY, 3,
    OrderType.LMT, 2.01d, null, null)
  .setLang(Language.en_US)
  .setUserMark("test_multi_leg");
TradeOrderResponse response = client.execute(request);
```

### 换汇单 / Forex Order

```java
ForexTradeOrderRequest request = ForexTradeOrderRequest.buildRequest("402901",
    SegmentType.SEC, Currency.HKD, 1000D, Currency.USD);
ForexTradeOrderResponse response = client.execute(request);
```

### 基金金额单 / Fund Amount Order

```java
ContractItem contract = ContractItem.buildFundContract("IE00B464Q616.USD", "USD");
TradeOrderRequest request = TradeOrderRequest.buildAmountOrder(
    "13810712", contract, ActionType.BUY, 100.0D);
request.setUserMark("test-amount-order");
TradeOrderResponse response = client.execute(request);
```

---

## 预览订单 / Preview Order
<!-- 当用户提到 "预览"、"佣金"、"保证金"、"preview"、"commission" 时 -->

**对应的请求类：TradeOrderPreviewRequest**

预览订单，返回是否可提交订单以及资产信息。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| symbol | string | Yes | 股票代码 |
| sec_type | string | Yes | STK/OPT/WAR/IOPT |
| action | string | Yes | BUY/SELL |
| order_type | string | Yes | MKT/LMT/STP/STP_LMT/TRAIL |
| total_quantity | int | Yes | 订单数量 |
| limit_price | double | No | 限价 |
| aux_price | double | No | 止损价/跟踪额 |
| trailing_percent | double | No | 跟踪止损百分比 |
| outside_rth | boolean | No | 是否允许盘前盘后 |
| market | string | No | US/HK/CN |
| currency | string | No | USD/HKD/CNH |
| time_in_force | string | No | DAY/GTC |

**返回字段 / Response Fields:**

| 字段 Field | 类型 Type | 描述 Description |
|-----------|---------|-----------------|
| account | String | 账户id |
| initMargin | Double | 下单后初始保证金 (不支持期货) |
| maintMargin | Double | 下单后维持保证金 (不支持期货) |
| equityWithLoan | Double | 下单后可借贷资产 (不支持期货) |
| initMarginBefore | Double | 下单前初始保证金 (不支持期货) |
| maintMarginBefore | Double | 下单前维持保证金 (不支持期货) |
| equityWithLoanBefore | Double | 下单前可借贷资产 (不支持期货) |
| marginCurrency | String | 保证金币种 |
| commission | Double | 预估佣金 |
| gst | Double | 预估消费税 |
| commissionCurrency | String | 预估佣金币种 |
| availableEe | Double | 可用剩余资产 (不支持期货) |
| excessLiquidity | Double | 剩余流动性 (不支持期货) |
| overnightLiquidation | Double | 隔夜剩余流动性 (不支持期货) |
| isPass | Boolean | 是否可提交订单 |
| message | String | 不可提交订单的错误原因 |

```java
ContractItem contract = ContractItem.buildStockContract("SPY", "USD");
TradeOrderPreviewRequest request = TradeOrderPreviewRequest.buildLimitOrder(contract, ActionType.BUY, 1, 100.0d);
TradeOrderPreviewResponse response = client.execute(request);
System.out.println(JSONObject.toJSONString(response));
```

---

## 修改订单 / Modify Order
<!-- 当用户提到 "改单"、"修改订单"、"modify"、"change order" 时 -->

**对应的请求类：TigerHttpRequest(MethodName.MODIFY_ORDER)**

修改订单，订单类型不支持修改。仅 Submitted 或部分成交状态可改。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| id | long | Yes | 订单id, 下单时返回 |
| total_quantity | long | No | 订单数量 |
| total_quantity_scale | int | No | 下单数量偏移量 (碎股用) |
| limit_price | double | No | 限价 |
| aux_price | double | No | 止损价/跟踪差额 |
| trailing_percent | double | No | 跟踪止损百分比 |
| secret_key | string | No | 交易员密钥, 机构用户专用 |

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.MODIFY_ORDER);
String bizContent = TradeParamBuilder.instance()
    .account("DU000001")
    .id(100000000000000001L)
    .totalQuantity(200)
    .limitPrice(60.0)
    .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
JSONObject data = JSON.parseObject(response.getData());
Long id = data.getLong("id");
```

---

## 撤销订单 / Cancel Order
<!-- 当用户提到 "撤单"、"取消订单"、"cancel order" 时 -->

**对应的请求类：TigerHttpRequest(MethodName.CANCEL_ORDER)**

撤销已下的订单。撤单为异步执行，调用后表示撤单请求发送成功。已成交或被系统拒绝的订单无法被撤销，只有订单处于已提交或部分成交的状态才可被撤销。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| id | long | Yes | 订单号, 下单时返回的ID |
| secret_key | string | No | 交易员密钥, 机构用户专用 |

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.CANCEL_ORDER);
String bizContent = TradeParamBuilder.instance()
    .account("DU000001")
    .id(100000000000000001L)
    .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
JSONObject data = JSON.parseObject(response.getData());
Long id = data.getLong("id");
```

> 对于批量下单，可用 ACTIVE_ORDERS 取得待成交的订单列表依次撤单。对于已经请求过撤单的订单，不要重复请求。
> For batch orders, use ACTIVE_ORDERS to get pending orders and cancel them one by one. Don't repeat cancel requests.

---

## 查询订单 / Query Orders
<!-- 当用户提到 "订单"、"委托"、"orders"、"order status"、"order list" 时 -->

### 获取单个订单 / Get Single Order

**对应的请求类：QuerySingleOrderRequest**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| id | int | Yes | 下单成功后返回的订单号 |
| secret_key | string | No | 交易员密钥, 机构用户专用 |
| show_charges | bool | No | 是否返回订单的费用明细 |

```java
QuerySingleOrderRequest request = new QuerySingleOrderRequest();
String bizContent = AccountParamBuilder.instance()
        .account("572386")
        .id(31227598058424320L)
        .isShowCharges(true)
        .lang(Language.en_US)
        .buildJson();
request.setBizContent(bizContent);
SingleOrderResponse response = client.execute(request);

if (response.isSuccess()) {
    Long id = response.getItem().getId();
    String action = response.getItem().getAction();
}
```

### 获取订单列表 / Get Order List

**对应的请求类：QueryOrderRequest (MethodName.ORDERS)**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| seg_type | SegmentType | No | SEC/FUT/FUND/ALL, 默认SEC |
| sec_type | string | No | ALL/STK/OPT/FUT/FOP/CASH, 默认ALL |
| market | string | No | ALL/US/HK/CN, 默认ALL |
| symbol | string | No | 股票代码 |
| expiry | string | No | 过期日 (期权/窝轮/牛熊证) |
| strike | string | No | 行权价 |
| right | string | No | PUT/CALL |
| start_date | string | No | 起始时间 ('2018-05-01' 或 '2018-05-01 10:00:00'), 闭区间 |
| end_date | string | No | 截止时间, 开区间 |
| states | array | No | (仅环球账号) 订单状态筛选 |
| isBrief | boolean | No | (仅环球账号) 是否返回精简订单信息 |
| limit | integer | No | 默认100, 最大300 |
| sort_by | OrderSortBy | No | (仅综合账号) LATEST_CREATED/LATEST_STATUS_UPDATED |
| lang | string | No | zh_CN/zh_TW/en_US |
| page_token | string | No | 分页查询token |

```java
QueryOrderRequest request = new QueryOrderRequest();
String bizContent = AccountParamBuilder.instance()
    .account("572386")
    .startDate("2023-04-01 00:00:00", TimeZoneId.NewYork)
    .endDate("2023-06-20 23:59:59", TimeZoneId.NewYork)
    .secType(SecType.STK)
    .sortBy(OrderSortBy.LATEST_CREATED)
    .limit(5)
    .buildJson();
request.setBizContent(bizContent);
BatchOrderResponse response = client.execute(request);

if (response.isSuccess()) {
    List<TradeOrder> orders = response.getItem().getOrders();
}
```

### 分页查询 / Pagination with pageToken

```java
List<JSONObject> results = new ArrayList<>();
int page = 1;
String pageToken = "";

SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
long startTime = sdf.parse("2023-01-01").getTime();
long endTime = sdf.parse("2025-08-01").getTime();

while (true) {
    TigerHttpRequest request = new TigerHttpRequest(MethodName.ORDERS);
    String bizContent = AccountParamBuilder.instance()
        .account("402501")
        .symbol("AAPL")
        .startDate(String.valueOf(startTime))
        .endDate(String.valueOf(endTime))
        .limit(10)
        .pageToken(pageToken)
        .buildJson();
    request.setBizContent(bizContent);
    TigerHttpResponse response = client.execute(request);

    JSONObject responseData = JSON.parseObject(response.getData());
    JSONArray items = responseData.getJSONArray("items");
    page++;

    if (items != null && !items.isEmpty()) {
        for (int i = 0; i < items.size(); i++) {
            results.add(items.getJSONObject(i));
        }
    }

    pageToken = responseData.getString("nextPageToken");
    if (StringUtils.isEmpty(pageToken)) {
        break;
    }
}
```

### 获取已成交订单列表 / Get Filled Orders

```java
QueryOrderRequest request = new QueryOrderRequest(MethodName.FILLED_ORDERS);
String bizContent = AccountParamBuilder.instance()
        .account("402901")
        .secType(SecType.STK)
        .startDate("2023-05-15 22:34:30")
        .endDate("2023-06-06 22:34:31")
        .buildJson();
request.setBizContent(bizContent);
BatchOrderResponse response = client.execute(request);
```

### 获取待成交订单列表 / Get Active (Open) Orders

```java
QueryOrderRequest request = new QueryOrderRequest(MethodName.ACTIVE_ORDERS);
String bizContent = AccountParamBuilder.instance()
        .account("DU000001")
        .secType(SecType.STK)
        .buildJson();
request.setBizContent(bizContent);
BatchOrderResponse response = client.execute(request);
```

### 获取已撤销订单列表 / Get Inactive (Cancelled) Orders

```java
QueryOrderRequest request = new QueryOrderRequest(MethodName.INACTIVE_ORDERS);
String bizContent = AccountParamBuilder.instance()
        .account("DU000001")
        .secType(SecType.STK)
        .buildJson();
request.setBizContent(bizContent);
BatchOrderResponse response = client.execute(request);
```

### Order 对象属性 / Order Attributes

| 名称 Field | 说明 Description |
|-----------|-----------------|
| id | 订单全局唯一ID Global order ID |
| orderId | 用户本地自增订单ID, 非全局唯一 |
| parentId | 父订单ID |
| account | 交易账户 |
| action | BUY/SELL |
| orderType | MKT/LMT/STP/STP_LMT/TRAIL |
| limitPrice | 限价单价格 |
| auxPrice | 止损辅助价格/跟踪额 |
| trailingPercent | 跟踪百分比 |
| totalQuantity | 下单数量 |
| totalQuantityScale | 数量偏移量 (碎股) |
| filledQuantity | 成交数量 |
| avgFillPrice | 平均成交价 (含佣金) |
| status | 订单状态 |
| timeInForce | DAY/GTC/GTD |
| expireTime | GTD有效截止时间 |
| outsideRth | 是否允许盘前盘后 |
| symbol | 股票代码 |
| currency | 货币 |
| market | 交易市场 |
| secType | 交易类型 |
| commission | 佣金+印花税等费用 |
| realizedPnl | 已实现盈亏 |
| openTime | 下单时间 |
| updateTime | 最后更新时间 |
| latestTime | 状态更新时间 |
| userMark | 下单备注 |
| canModify | 是否可修改 |
| canCancel | 是否可撤销 |
| liquidation | 是否强制平仓 |
| isOpen | 是否为开仓 |
| remark | 错误描述 |
| attrDesc | 订单描述信息 |
| replaceStatus | 改单状态 |
| cancelStatus | 撤单状态 |
| charges | 订单费用明细 (仅单个查询) |

### 订单状态 / Order Status

| 状态 Status | 说明 Description |
|------------|-----------------|
| Initial | 初始化 Created |
| PendingSubmit | 待提交 Pending submit |
| Submitted | 已提交 Submitted |
| PartiallyFilled | 部分成交 Partially filled |
| Filled | 全部成交 Fully filled |
| Cancelled | 已取消 Cancelled |
| Inactive | 已失效/拒绝 Rejected |
| PendingCancel | 待取消 Pending cancel |
| Invalid | 无效订单 Invalid |

**订单状态判断说明:**
1. 综合和模拟账号: 当状态不是 Initial 和 Filled 时 (可能是 PendingSubmit, Cancelled, Invalid, Inactive), 可通过成交数量>0判断是否部分成交
2. 环球账号: 状态为 Filled 且成交数量>0

---

## 成交记录 / Transaction Records

**对应的请求类：TigerHttpRequest(MethodName.ORDER_TRANSACTIONS)**

获取订单的成交记录。目前仅支持综合账户。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | String | Yes | 账户 |
| order_id | long | Yes* | 全局订单ID (order_id和symbol二选一, orderId优先) |
| symbol | String | Yes* | 股票代码 (order_id和symbol二选一) |
| sec_type | String | 指定symbol时必传 | STK/FUT/OPT/WAR/IOPT |
| expiry | String | OPT/WAR/IOPT时必传 | 到期日 |
| right | String | OPT/WAR/IOPT时必传 | CALL/PUT |
| start_date | long | No | 起始时间 (毫秒时间戳) |
| end_date | long | No | 截止时间 (毫秒时间戳) |
| since_date | String | No | 起始日期 (yyyyMMdd) |
| to_date | String | No | 截止日期 (yyyyMMdd) |
| limit | int | No | 数量限制, 默认20, 最大100 |
| page_token | String | No | 分页查询token |

**返回字段 / Response Fields:**

| 字段 Field | 说明 Description |
|-----------|-----------------|
| id | 成交记录ID |
| accountId | 账号 |
| orderId | 订单ID |
| secType | 证券类型 |
| symbol | 代码 |
| currency | 币种 |
| market | 市场 |
| action | BUY/SELL |
| filledQuantity | 成交数量 |
| filledQuantityScale | 成交数量偏移量 |
| filledPrice | 成交价 |
| filledAmount | 成交金额 |
| transactedAt | 成交时间 |
| transactionTime | 成交时间戳 |
| nextPageToken | 下一页token |

```java
// 按symbol查询 / Query by symbol
TigerHttpRequest request = new TigerHttpRequest(MethodName.ORDER_TRANSACTIONS);
String bizContent = AccountParamBuilder.instance()
    .account("402501")
    .secType(SecType.STK)
    .symbol("CII")
    .limit(30)
    .startDate("2021-11-15 22:34:30")
    .endDate("2021-11-15 22:34:31")
    .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
JSONArray data = JSON.parseObject(response.getData()).getJSONArray("items");

// 按orderId查询 / Query by orderId
TigerHttpRequest request = new TigerHttpRequest(MethodName.ORDER_TRANSACTIONS);
String bizContent = AccountParamBuilder.instance()
    .account("402501")
    .orderId(24637316162520064L)
    .limit(30)
    .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);
```

分页获取成交记录 / Pagination:

```java
List<JSONObject> results = new ArrayList<>();
String pageToken = "";
while (true) {
    TigerHttpRequest request = new TigerHttpRequest(MethodName.ORDER_TRANSACTIONS);
    String bizContent = AccountParamBuilder.instance()
        .account("402501")
        .secType(SecType.STK)
        .startDate(String.valueOf(startTime))
        .endDate(String.valueOf(endTime))
        .limit(10)
        .pageToken(pageToken)
        .buildJson();
    request.setBizContent(bizContent);
    TigerHttpResponse response = client.execute(request);
    JSONObject responseData = JSON.parseObject(response.getData());
    JSONArray items = responseData.getJSONArray("items");
    if (items != null && !items.isEmpty()) {
        for (int i = 0; i < items.size(); i++) {
            results.add(items.getJSONObject(i));
        }
    }
    pageToken = responseData.getString("nextPageToken");
    if (StringUtils.isEmpty(pageToken)) break;
}
```

---

## 账户资产 / Account Assets
<!-- 当用户提到 "资产"、"资金"、"余额"、"购买力"、"assets"、"balance"、"buying power"、"cash" 时 -->

### 环球账户资产 / Global Account Assets

**对应的请求类：TigerHttpRequest(MethodName.ASSETS)**

主要适用于环球账户。综合/模拟账号建议使用 PrimeAssetRequest。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| segment | boolean | No | 是否包含证券/期货分类, 默认False |
| market_value | boolean | No | 是否包含分市场市值, 默认False (仅环球) |

**返回字段 / Response Fields:**

| 名称 Field | 说明 Description |
|-----------|-----------------|
| account | 交易账户 |
| capability | 账户类型 (RegTMargin/Cash) |
| netLiquidation | 净清算值 Net liquidation |
| equityWithLoan | 含借贷值股权 Equity with loan |
| initMarginReq | 初始保证金要求 Initial margin |
| maintMarginReq | 维持保证金要求 Maintenance margin |
| availableFunds | 可用资金 Available funds |
| buyingPower | 购买力 Buying power |
| cashValue | 现金金额 Cash value |
| excessLiquidity | 剩余流动性 Excess liquidity |
| cushion | 剩余流动性/总资产比例 |
| realizedPnl | 实际盈亏 Realized PnL |
| unrealizedPnl | 浮动盈亏 Unrealized PnL |
| dayTradesRemaining | 剩余日内交易次数 (-1无限制) |
| segments | 按交易品种区分 (S:证券, C:期货) |
| marketValues | 分市场市值 (USD:美股, HKD:港股) |

```java
TigerHttpRequest request = new TigerHttpRequest(MethodName.ASSETS);
String bizContent = AccountParamBuilder.instance()
        .account("DU000001")
        .buildJson();
request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);

JSONArray assets = JSON.parseObject(response.getData()).getJSONArray("items");
JSONObject asset1 = assets.getJSONObject(0);
String account = asset1.getString("account");
Double cashBalance = asset1.getDouble("cashBalance");
// segments: S=股票, C=期货
JSONArray segments = asset1.getJSONArray("segments");
```

### 综合/模拟账号资产 / Prime Assets (Recommended for Standard/Paper)

**对应的请求类：PrimeAssetRequest**

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| base_currency | string | No | HKD/USD |
| consolidated | boolean | No | 是否聚合Segment资产, 默认true (SEC和FUND会聚合) |

**PrimeAssetItem.Segment 字段 / Fields:**

| 名称 Field | 说明 Description |
|-----------|-----------------|
| category | 交易品种 S(证券)/C(期货)/F(基金)/D(数字货币) |
| capability | 账户类型 RegTMargin/Cash |
| buyingPower | 最大购买力 Buying power |
| cashAvailableForTrade | 可用资金 Available funds |
| cashAvailableForWithdrawal | 可出金金额 Withdrawable |
| cashBalance | 现金额 Cash balance |
| grossPositionValue | 证券总价值 Gross position value |
| initMargin | 初始保证金 Initial margin |
| maintainMargin | 维持保证金 Maintenance margin |
| overnightMargin | 隔夜保证金 Overnight margin |
| excessLiquidation | 当前剩余流动性 Excess liquidity |
| overnightLiquidation | 隔夜剩余流动性 Overnight liquidity |
| netLiquidation | 总资产(净清算值) Net liquidation |
| equityWithLoan | 含贷款价值总权益 Equity with loan |
| realizedPL | 已实现盈亏 Realized PnL |
| unrealizedPL | 持仓盈亏 Unrealized PnL |
| totalTodayPL | 当日盈亏 Today PnL |
| leverage | 杠杆 Leverage |
| lockedFunds | 锁定资产 Locked funds |
| currencyAssets | 按币种的资产信息 Currency assets |

**PrimeAssetItem.CurrencyAssets 字段:**

| 名称 Field | 说明 Description |
|-----------|-----------------|
| currency | 货币币种 USD/HKD/SGD/CNH |
| cashBalance | 现金余额 Cash balance |
| cashAvailableForTrade | 可交易现金 Available for trade |
| forexRate | 对baseCurrency汇率 Forex rate |

```java
PrimeAssetRequest assetRequest = PrimeAssetRequest.buildPrimeAssetRequest("572386", Currency.USD);
assetRequest.setConsolidated(Boolean.TRUE);
PrimeAssetResponse primeAssetResponse = client.execute(assetRequest);

// 查询证券相关资产 / Get securities segment
PrimeAssetItem.Segment segment = primeAssetResponse.getSegment(Category.S);

// 查询美元资产 / Get USD currency assets
if (segment != null) {
    PrimeAssetItem.CurrencyAssets assetByCurrency = segment.getAssetByCurrency(Currency.USD);
}
```

---

## 持仓 / Positions
<!-- 当用户提到 "持仓"、"仓位"、"我的股票"、"positions"、"holdings"、"portfolio" 时 -->

**对应的请求类：PositionsRequest**

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account | string | Yes | 用户授权账户 |
| sec_type | string | No | STK/OPT/FUT/WAR/IOPT/CASH/FOP/FUND/CC, 默认STK |
| currency | string | No | ALL/USD/HKD/CNH, 默认ALL |
| market | string | No | ALL/US/HK/CN, 默认ALL |
| symbol | string | No | 股票代码 |
| expiry | string | No | 过期日 (期权/窝轮/牛熊证), yyyyMMdd |
| strike | double | No | 行权价 |
| right | string | No | PUT/CALL |
| asset_quote_type | string | No | 资产行情模式 (仅综合账号) |

**PositionDetail 返回字段 / Response Fields:**

| 名称 Field | 说明 Description |
|-----------|-----------------|
| account | 交易账户 |
| positionQty | 持仓数量 Position quantity |
| salableQty | 可卖数量 Salable quantity |
| averageCost | 平均FIFO成本 Average FIFO cost |
| averageCostByAverage | 平均均价成本 Average cost |
| averageCostOfCarry | 摊薄持仓成本 (A股模式) Cost of carry |
| latestPrice | 市价 Market price |
| marketValue | 市值 Market value |
| realizedPnl | 已实现盈亏 Realized PnL |
| unrealizedPnl | 浮动盈亏 Unrealized PnL |
| unrealizedPnlPercent | 浮动收益率 Unrealized PnL % |
| multiplier | 每手数量 Multiplier |
| market | 市场 Market |
| currency | 币种 Currency |
| secType | 交易类型 Security type |
| identifier | 标的标识 Identifier |
| symbol | 股票代码 Symbol |
| strike | 期权行权价 (仅期权) |
| expiry | 期权过期日 (仅期权) |
| right | 期权方向 (仅期权) |
| todayPnl | 今日盈亏额 Today PnL |
| todayPnlPercent | 今日盈亏率 Today PnL % |
| mmValue | 维持保证金 Maintenance margin |

```java
PositionsRequest request = new PositionsRequest();
String bizContent = AccountParamBuilder.instance()
        .account("13810712")
        .secType(SecType.STK)
        .buildJson();
request.setBizContent(bizContent);
PositionsResponse response = client.execute(request);

if (response.isSuccess()) {
    for (PositionDetail detail : response.getItem().getPositions()) {
        String account = detail.getAccount();
        String symbol = detail.getSymbol();
        double qty = detail.getPositionQty();
        double avgCost = detail.getAverageCost();
        double latestPrice = detail.getLatestPrice();
        double unrealizedPnl = detail.getUnrealizedPnl();
        double salableQty = detail.getSalableQty();
    }
}
```

---

## 持仓划转 / Position Transfer (Institutional)

机构用户专用。通过SDK执行内部头寸划转并查询划转记录，同时支持查询外部划转记录（如 DWAC、FOP 等）。

### 发起内部持仓划转 / Initiate Internal Position Transfer

**对应的请求类：PositionTransferRequest**

该接口用于机构用户发起内部持仓划转。用户需已开通OpenAPI权限，并具有对转出账户执行头寸划转的权限。

如需查询可转的持仓和数量，请查询账户持仓接口，其中 `salableQty` 代表可转数量。

**参数 / Parameters:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| from_account | string | Yes | 转出账户 ID |
| to_account | string | Yes | 转入账户 ID |
| market | string | Yes | 市场 US/HK |
| transfers | `List<Transfer>` | Yes | 持仓划转列表 |

**Transfer 参数:**

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| symbol | string | Yes | 标的代码 |
| quantity | long | Yes | 划转数量 |
| sec_type | string | No | STK/OPT/FUND, 默认STK |
| expiry | string | No | 到期时间 (期权), yyyyMMdd |
| strike | string | No | 行权价 (期权) |
| right | string | No | CALL/PUT (期权) |

**返回字段 / Response Fields:**

| 字段 Field | 说明 Description |
|-----------|-----------------|
| id | 内部划转记录 ID |
| accountId | 转出账户 ID |
| counterpartyAccountId | 转入账户 ID |
| method | 划转方式 (INTERNAL_POSITION) |
| direction | 划转方向 (OUT) |
| status | 状态: PendingNew/NEW/ManualProcessing/Succ/Fail/Canceled/PartialSucc |
| memo | 附加信息 |
| finishedAt | 完成时间 |
| createdAt | 创建时间 |

```java
List<Transfer> transfers = new ArrayList<>();
transfers.add(new Transfer("TSLA", 1L));
PositionTransferRequest request = PositionTransferRequest.buildRequest(
    "600021044671",
    "600021044659",
    Market.US.name(),
    transfers
);
```

### 获取内部持仓划转记录 / Get Internal Transfer Records

**对应的请求类：PositionTransferRecordsRequest**

查询机构账户在指定时间范围内（不超过 90 天）的内部持仓划转记录。

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account_id | string | Yes | 账户 ID |
| since_date | string | Yes | 起始时间 yyyy-MM-dd |
| to_date | string | Yes | 结束时间 yyyy-MM-dd |
| status | string | No | 状态过滤 |
| market | string | No | 市场过滤 US/HK |
| symbol | string | No | 标的过滤 (仅股票) |

```java
PositionTransferRecordsRequest request = PositionTransferRecordsRequest.buildRequest(
    "600021044671",
    "2025-12-21",
    "2025-12-23"
);
```

### 获取内部持仓划转详情 / Get Internal Transfer Detail

**对应的请求类：PositionTransferDetailRequest**

查询指定划转记录的划转详情，包括每个标的的划转状态。

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| id | long | Yes | 划转记录 ID |
| account_id | string | Yes | 转出账户 ID |

**TransferDetail 字段:**

| 字段 Field | 说明 Description |
|-----------|-----------------|
| transferId | 划转记录 ID |
| symbol | 标的代码 |
| formattedSymbol | 格式化标的 (期权适用) |
| market | 市场 |
| quantity | 划转数量 |
| status | 划转状态 |
| message | 失败原因 |

```java
PositionTransferDetailRequest request = PositionTransferDetailRequest.buildRequest(8189L, "600021044671");
```

### 获取外部持仓划转记录 / Get External Transfer Records

**对应的请求类：PositionTransferExternalRecordsRequest**

查询机构账户在指定时间范围内（不超过 90 天）的外部划转记录，支持 DWAC、DRS、FOP 等。

| 参数 Parameter | 类型 Type | 必填 Required | 描述 Description |
|---------------|---------|-------------|-----------------|
| account_id | string | Yes | 账户 ID |
| since_date | string | Yes | 起始时间 yyyy-MM-dd |
| to_date | string | Yes | 结束时间 yyyy-MM-dd |
| status | string | No | 状态过滤 |
| market | string | No | 市场过滤 US/HK |
| symbol | string | No | 代码过滤 (仅股票) |

**返回字段 / Response Fields:**

| 字段 Field | 说明 Description |
|-----------|-----------------|
| id | 划转记录 ID |
| status | 划转整体状态 (如 SUCCESS) |
| accountId | 账户 ID |
| transferMethod | DWAC/DRS/FOP |
| side | IN(老虎接收)/OUT(老虎转出) |
| institutionName | 对手方名称 |
| remoteAccount | 对手方账户 ID |
| transferPropertyInfos | 划转明细列表 |
| cancelable | 是否可取消 |

```java
PositionTransferExternalRecordsRequest request = PositionTransferExternalRecordsRequest.buildRequest(
    "600021044671",
    "2025-11-21",
    "2025-12-23"
);
```

---

## 交易时间参考 / Trading Hours Reference

| 市场 Market | 时段 Session | 时间 Time (当地/Local) |
|-------------|-------------|----------------------|
| 美股 US | 盘前 Pre-market | 04:00-09:30 ET |
| | 常规 Regular | 09:30-16:00 ET |
| | 盘后 After-hours | 16:00-20:00 ET |
| | 夜盘 Overnight | 20:00-04:00 ET |
| 港股 HK | 竞价 Pre-opening | 09:00-09:30 |
| | 上午 Morning | 09:30-12:00 |
| | 下午 Afternoon | 13:00-16:00 |
| | 收盘竞价 Closing | 16:00-16:10 |

> 更多交易规则见 / More: https://docs.itigerup.com/docs/trade-rules

---

## 注意事项 / Notes

- `place_order` 返回成功仅表示提交成功，需查询确认成交 / Success only means submitted; check order status
- 市价单(MKT)和止损单(STP)不支持盘前盘后 / MKT and STP don't support pre/after-hours
- 同一证券不能同时持有多头和空头 / Cannot hold long and short simultaneously
- 禁止直接开立反向头寸, 需先平掉现有头寸 / Must close existing position before opening reverse
- 下单前建议 preview_order / Use preview before placing real orders
- 已成交订单查询需指定时间范围，最多90天 / Filled orders require time range, max 90 days
- 港股下单数量须为 lotSize 的整数倍 / HK orders must be multiples of lotSize
- 碎股精度: total_quantity + total_quantity_scale, 如 qty=111 scale=2 -> 1.11
- TWAP/VWAP算法单不能改单, 可以撤单 / TWAP/VWAP orders can be cancelled but not modified
- 限价价格不匹配 tickSize 时, 可利用 StockPriceUtils 工具类修复 / Use StockPriceUtils to fix tick size mismatch
- 模拟账号暂不支持窝轮、牛熊证、OCA订单 / Paper account doesn't support WAR/IOPT/OCA
