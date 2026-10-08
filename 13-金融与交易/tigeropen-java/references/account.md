
# Tiger Open API Java SDK - 账户管理 / Account Management

> 中文 | English — 双语技能，代码通用。Bilingual skill with shared code examples.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts-java

## 初始化 / Initialize

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
```

---

## 账户列表 / Managed Accounts

**请求类 Request Class：`TigerHttpRequest(MethodName.ACCOUNTS)`**

获取管理的账户列表，机构账户返回主账户及所有子账户。
Get managed account list. Institutional accounts return primary + all sub-accounts.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | No | 用户授权账号（不支持传入模拟账号），不传会返回全部账号，包括：综合，环球，模拟。Authorized account (paper accounts not supported). Omit to return all accounts (standard, global, paper). |

**返回字段 Response Fields**

| 字段 Field | 示例 Example | 说明 Description |
|-----------|-------------|-----------------|
| account | 综合账号：50129912，环球：U5755619，模拟账号：20191221901212121 | 交易的资金账户。综合账号为 5~10 位数字，模拟账号为 17 位数字，环球账号以字母 U 开头。Trading account. Standard: 5-10 digits, Paper: 17 digits, Global: starts with 'U'. |
| capability | RegTMargin | 账户类型 Account type（CASH：现金账户 Cash，RegTMargin：Reg T 保证金 Margin，PMGRN：投资组合保证金 Portfolio Margin） |
| status | Funded | 账号状态 Account status：Funded（已入金），Open（已开户），Pending（待开户），Rejected（开户被拒绝），Closed（已注销） |
| accountType | STANDARD | 账户分类 Account category（GLOBAL：环球账号，STANDARD：综合账号，PAPER：模拟账号） |

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
TigerHttpRequest request = new TigerHttpRequest(MethodName.ACCOUNTS);

String bizContent = AccountParamBuilder.instance()
        .account("123456")
        .buildJsonWithoutDefaultAccount();
// 查询ClientConfig.DEFAULT_CONFIG配置的默认账号
// String bizContent = AccountParamBuilder.instance().buildJson();
// 查询所有账号列表
// String bizContent = AccountParamBuilder.instance().buildJsonWithoutDefaultAccount();

request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);

// 获取具体字段数据
JSONArray accounts = JSON.parseObject(response.getData()).getJSONArray("items");
JSONObject account1 = accounts.getJSONObject(0);
String capability = account1.getString("capability");
String accountType = account1.getString("accountType");
String account = account1.getString("account");
String status = account1.getString("status");
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [
      {
        "account": "123456",
        "capability": "RegTMargin",
        "status": "Funded",
        "accountType": "STANDARD"
      }
    ]
  }
}
```

---

## 环球账户资产 / Global Account Assets

**请求类 Request Class：`TigerHttpRequest(MethodName.ASSETS)`**

获取环球账户资产。综合/模拟账号建议使用 PrimeAssetRequest。
Get global account assets. For standard/paper accounts, use PrimeAssetRequest instead.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账户 Authorized account, e.g. DU000001 |
| segment | boolean | No | 是否包含证券/期货分类 Include securities/futures segments, 默认 False |
| market_value | boolean | No | 是否包含分市场市值 Include market values by market, 默认 False，仅环球账户支持 Global only |
| secret_key | string | No | 交易员密钥，机构用户专用 Trader secret key, institutional only |

**返回字段 Response Fields**

| 名称 Field | 示例 Example | 说明 Description |
|-----------|-------------|-----------------|
| account | DU000001 | 交易账户 Trading account |
| capability | RegTMargin | 账户类型 Account type (RegTMargin: 保证金 Margin, Cash: 现金 Cash) |
| netLiquidation | 1233662.93 | 净清算值 Net liquidation value |
| equityWithLoan | 1233078.69 | 含借贷值股权 Equity with loan. 证券Segment: 现金价值+股票价值, 期货Segment: 现金价值-维持保证金 |
| initMarginReq | 292046.91 | 初始保证金要求 Initial margin requirement |
| maintMarginReq | 273170.84 | 维持保证金要求 Maintenance margin requirement |
| availableFunds | 941031.78 | 可用资金 Available funds = equity_with_loan - initial_margin_requirement |
| dayTradesRemaining | -1 | 剩余日内交易次数 Remaining day trades (-1 = unlimited) |
| excessLiquidity | 960492.09 | 剩余流动性 Excess liquidity (日内风险数值) |
| buyingPower | 6273545.18 | 购买力 Buying power |
| cashValue | 469140.39 | 现金金额 Cash value (证券+期货) |
| accruedCash | -763.2 | 累积应付利息 Accrued interest |
| accruedDividend | 0.0 | 累计分红 Accrued dividends |
| grossPositionValue | 865644.18 | 证券总价值 Gross position value |
| SMA | 0.0 | 特殊备忘录账户 Special Memorandum Account |
| regTEquity | 0.0 | RegT equity (证券Segment) |
| regTMargin | 0.0 | RegT margin (证券Segment) |
| cushion | 0.778569 | 剩余流动性占比 = excess_liquidity / net_liquidation |
| currency | USD | 货币币种 Currency |
| realizedPnl | -248.72 | 实际盈亏 Realized P&L |
| unrealizedPnl | -17039.09 | 浮动盈亏 Unrealized P&L |
| updateTime | 0 | 更新时间 Update time |
| **segments** | | 按交易品种区分的账户信息 Map: 'S'=证券 Securities, 'C'=期货 Commodities |
| **marketValues** | | 分市场市值信息 Map: 'USD'=US, 'HKD'=HK |

**`segments` 说明 Fields**

| 名称 Field | 示例 Example | 说明 Description |
|-----------|-------------|-----------------|
| account | DU000001 | 交易账户 Trading account |
| category | S | 分类 Category: C (Commodities 期货) / S (Securities 证券) |
| title | US Securities | 标题 Title |
| netLiquidation | 1233662.93 | 净清算值 Net liquidation |
| cashValue | 469140.39 | 现金金额 Cash value |
| availableFunds | 941031.78 | 可用资金 Available funds |
| equityWithLoan | 1233078.69 | 含借贷值股权 Equity with loan |
| excessLiquidity | 960492.09 | 剩余流动性 Excess liquidity |
| accruedCash | -763.2 | 净累计利息 Net accrued interest |
| accruedDividend | 0.0 | 净累计分红 Net accrued dividend |
| initMarginReq | 292046.91 | 初始保证金要求 Initial margin |
| maintMarginReq | 273170.84 | 维持保证金要求 Maintenance margin |
| regTEquity | 0.0 | RegT 资产 RegT equity |
| regTMargin | 0.0 | RegT 保证金 RegT margin |
| SMA | 0.0 | 特殊备忘录账户 SMA |
| grossPositionValue | 865644.18 | 持仓市值 Gross position value |
| leverage | 1 | 杠杆 Leverage |
| updateTime | 1526368181000 | 更新时间 Update time |

**`marketValues` 说明 Fields**

| 名称 Field | 示例 Example | 说明 Description |
|-----------|-------------|-----------------|
| account | DU000001 | 交易账户 Trading account |
| currency | USD | 货币币种 Currency |
| netLiquidation | 1233662.93 | 总资产 (净清算价值) Net liquidation |
| cashBalance | 469140.39 | 现金 Cash balance |
| exchangeRate | 0.1273896 | 对账户主币种的汇率 Exchange rate to base currency |
| netDividend | 0.0 | 应付/应收股息净值 Net dividend |
| futuresPnl | 0.0 | 盯市盈亏 Futures mark-to-market P&L |
| realizedPnl | -248.72 | 已实现盈亏 Realized P&L |
| unrealizedPnl | -17039.09 | 浮动盈亏 Unrealized P&L |
| updateTime | 1526368181000 | 更新时间 Update time |
| stockMarketValue | 943588.78 | 股票市值 Stock market value |
| optionMarketValue | 0.0 | 期权市值 Option market value |
| futureOptionValue | 0.0 | 期货市值 Future option value |
| warrantValue | 10958.0 | 窝轮市值 Warrant value |

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
TigerHttpRequest request = new TigerHttpRequest(MethodName.ASSETS);

String bizContent = AccountParamBuilder.instance()
        .account("DU000001")
        .buildJson();

request.setBizContent(bizContent);
TigerHttpResponse response = client.execute(request);

// 解析具体字段
JSONArray assets = JSON.parseObject(response.getData()).getJSONArray("items");
JSONObject asset1 = assets.getJSONObject(0);
String account = asset1.getString("account");
Double cashBalance = asset1.getDouble("cashBalance");
JSONArray segments = asset1.getJSONArray("segments");
JSONObject segment = segments.getJSONObject(0);
String category = segment.getString("category"); // "S" 股票， "C" 期货
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [{
      "account": "DU000001",
      "accruedCash": -763.2,
      "accruedDividend": 0.0,
      "availableFunds": 941031.78,
      "buyingPower": 6273545.18,
      "capability": "Reg T Margin",
      "cashBalance": 469140.39,
      "cashValue": 469140.39,
      "currency": "USD",
      "cushion": 0.778569,
      "dayTradesRemaining": -1,
      "equityWithLoan": 1233078.69,
      "excessLiquidity": 960492.09,
      "grossPositionValue": 865644.18,
      "initMarginReq": 292046.91,
      "maintMarginReq": 273170.84,
      "netLiquidation": 1233662.93,
      "realizedPnl": -31.68,
      "unrealizedPnl": 1814.01,
      "segments": [{
        "account": "DU000001",
        "availableFunds": 65.55,
        "cashValue": 65.55,
        "category": "S",
        "equityWithLoan": 958.59,
        "grossPositionValue": 893.04,
        "initMarginReq": 893.04,
        "leverage": 0.93,
        "maintMarginReq": 893.04,
        "netLiquidation": 958.59,
        "title": "US Securities"
      }],
      "marketValues": [{
        "account": "DU000001",
        "currency": "HKD",
        "exchangeRate": 0.1273896,
        "netLiquidation": 11223.29,
        "stockMarketValue": 943588.78,
        "unrealizedPnl": -17039.09,
        "warrantValue": 10958.0
      }]
    }]
  }
}
```

---

## 综合/模拟账号获取资产 / Prime Account Assets

**请求类 Request Class：`PrimeAssetRequest`**

查询综合/模拟账户的资产（推荐用于综合/模拟账户）。
Query standard/paper account assets (recommended for standard/paper accounts).

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账户 Authorized account, e.g. 123123 |
| base_currency | string | No | 币种 Currency: HKD/USD |
| secret_key | string | No | 交易员密钥，机构用户专用 Trader secret key, institutional only |
| consolidated | boolean | No | 是否展示聚合Segment资产指标 Show consolidated segment metrics，只有SEC和FUND类别资产会聚合 Only SEC and FUND are consolidated。默认为true |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.PrimeAssetResponse`

可用 `PrimeAssetItem.Segment segment = primeAssetResponse.getSegment(Category.S)` 获取按交易品种划分的资产；
对于每个品种的资产，可用 `PrimeAssetItem.CurrencyAssets assetByCurrency = segment.getAssetByCurrency(Currency.USD)` 获取对应币种的资产。

**`PrimeAssetItem.Segment` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| account | String | 交易账户 Trading account |
| currency | String | 货币的币种 Currency |
| category | String | 交易品种分类 Category: S (Securities 证券) / C (Commodities 期货) / F (Funds 基金) / D (Digi 数字货币) |
| capability | String | 账户类型 Account type: RegTMargin (保证金 Margin) / Cash (现金 Cash) |
| buyingPower | Double | 最大购买力 Buying power。保证金账户日内最多4倍，隔夜最多2倍 Margin: up to 4x intraday, 2x overnight |
| cashAvailableForTrade | Double | 可用资金 Cash available for trade。用来检查是否可以开仓或打新 Used to check if new positions can be opened |
| cashAvailableForWithdrawal | Double | 可出金的现金金额 Cash available for withdrawal |
| cashBalance | Double | 现金额 Cash balance。所有币种现金余额之和 Sum of cash balance in all currencies |
| grossPositionValue | Double | 证券总价值 Gross position value。全部持仓的总市值 Total market value of all positions |
| initMargin | Double | 初始保证金 Initial margin。当前所有持仓合约所需的初始保证金要求之和 Sum of initial margin for all positions |
| maintainMargin | Double | 维持保证金 Maintenance margin。低于此值会引发强平 Triggers forced liquidation if below |
| overnightMargin | Double | 隔夜保证金 Overnight margin。收盘前15分钟开始检查 Checked 15 min before close |
| excessLiquidation | Double | 当前剩余流动性 Excess liquidity = equityWithLoan - maintainMargin。小于0时会被强制平仓 Forced liquidation if < 0 |
| overnightLiquidation | Double | 隔夜剩余流动性 Overnight excess liquidity = equityWithLoan - overnightMargin |
| netLiquidation | Double | 总资产(净清算值) Net liquidation = 证券总市值 + 现金余额 + 应计分红 - 应计利息 |
| equityWithLoan | Double | 含贷款价值总权益 Equity with loan (ELV)。保证金账户 = 现金余额 + 证券总市值 - 美股期权市值 |
| realizedPL | Double | 当天已实现盈亏 Today realized P&L (仅期货有效 futures only) |
| totalTodayPL | Double | 当日盈亏总和 Total today P&L |
| unrealizedPL | Double | 持仓盈亏 Unrealized P&L = 当前价 * 股数 - 持仓成本 |
| leverage | Double | 杠杆 Leverage = 证券市值绝对值之和 / 总资产 |
| lockedFunds | Double | 锁定资产 Locked funds |
| uncollected | Double | 在途资金 Uncollected funds |
| consolidatedSegTypes | String | 聚合的Segment Consolidated segments (SEC and FUND) |
| **currencyAssets** | CurrencyAssets | 按交易币种区分的账户资产信息 Assets by currency |

**`PrimeAssetItem.CurrencyAssets` 说明 Fields**

| 名称 Field | 示例 Example | 说明 Description |
|-----------|-------------|-----------------|
| currency | USD | 当前货币币种 Currency: USD/HKD/SGD/CNH |
| cashBalance | 469140.39 | 可交易现金+已锁定现金 Tradable cash + locked cash |
| cashAvailableForTrade | 0.1273896 | 当前账号内可交易的现金金额 Cash available for trade |
| forexRate | 1.0 | 当前币种对 baseCurrency 的汇率 Forex rate to base currency |

**示例 Example**

```java
PrimeAssetRequest assetRequest = PrimeAssetRequest.buildPrimeAssetRequest("572386", Currency.USD);
assetRequest.setConsolidated(Boolean.TRUE);
PrimeAssetResponse primeAssetResponse = client.execute(assetRequest);

// 查询证券相关资产信息
PrimeAssetItem.Segment segment = primeAssetResponse.getSegment(Category.S);
System.out.println("segment: " + JSONObject.toJSONString(segment));

// 查询账号中美元相关资产信息
if (segment != null) {
  PrimeAssetItem.CurrencyAssets assetByCurrency = segment.getAssetByCurrency(Currency.USD);
  System.out.println("assetByCurrency: " + JSONObject.toJSONString(assetByCurrency));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": {
    "accountId": "572386",
    "segments": [
      {
        "buyingPower": 878.52,
        "capability": "CASH",
        "cashAvailableForTrade": 878.52,
        "cashBalance": 6850.79,
        "category": "S",
        "consolidatedSegTypes": ["SEC", "FUND"],
        "currency": "USD",
        "currencyAssets": [
          {
            "cashAvailableForTrade": 35.89,
            "cashBalance": 6008.16,
            "currency": "USD",
            "forexRate": 1.0
          },
          {
            "cashAvailableForTrade": 5351.53,
            "cashBalance": 5351.53,
            "currency": "HKD",
            "forexRate": 0.128
          }
        ],
        "equityWithLoan": 8891.76,
        "grossPositionValue": 438.78,
        "initMargin": 2040.96,
        "leverage": 0.23,
        "maintainMargin": 2040.96,
        "netLiquidation": 7289.57,
        "overnightLiquidation": 6850.79,
        "overnightMargin": 2040.96,
        "realizedPL": 0,
        "unrealizedPL": 73.29
      },
      {
        "buyingPower": 878.52,
        "capability": "CASH",
        "cashAvailableForTrade": 878.52,
        "cashBalance": 0,
        "category": "F",
        "consolidatedSegTypes": ["SEC", "FUND"],
        "currency": "USD",
        "currencyAssets": [
          { "cashAvailableForTrade": 0, "cashBalance": 0, "currency": "USD" },
          { "cashAvailableForTrade": 0, "cashBalance": 0, "currency": "HKD" }
        ],
        "netLiquidation": 1602.18,
        "realizedPL": 1.25,
        "unrealizedPL": 89.35
      }
    ],
    "updateTimestamp": 1704355652720
  },
  "message": "success"
}
```

---

## 账户持仓 / Account Positions

**请求类 Request Class：`PositionsRequest`**

查询账户持仓。Query account positions.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账户 Authorized account |
| sec_type | string | No | 证券类型 Security type: STK/OPT/FUT/WAR/IOPT/CASH/FOP/FUND/CC，默认 STK |
| currency | string | No | 货币类型 Currency: ALL/USD/HKD/CNH，默认 ALL |
| market | string | No | 市场分类 Market: ALL/US/HK/CN，默认 ALL |
| symbol | string | No | 股票代码 Symbol。期权可传底层代码或 identifier (如 'AAPL  190111C00095000'，21位固定格式) |
| secret_key | string | No | 交易员密钥，机构用户专用 Trader secret key, institutional only |
| expiry | string | No | 过期日 Expiry (期权/窝轮/牛熊证)，格式 'yyyyMMdd' |
| strike | double | No | 行权价 Strike (期权/窝轮/牛熊证) |
| right | string | No | 看涨或看跌 Put/Call: 'PUT' / 'CALL' (期权/窝轮/牛熊证) |
| asset_quote_type | string | No | 资产行情模式 Asset quote type (仅限综合账号 standard only) |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.PositionsResponse`

可用 `List<PositionDetail> items = response.getItem().getPositions()` 获取持仓列表。

**`PositionDetail` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| account | string | 交易账户 Trading account |
| positionQty | double | 持仓数量 Position quantity |
| salableQty | double | 可卖数量 Salable quantity |
| position | long | 持仓数量（废弃 Deprecated） |
| positionScale | int | 持仓数量偏移量（废弃 Deprecated）。如 position=11123, positionScale=2, 实际 position=111.23 |
| averageCost | double | 平均FIFO成本 Average FIFO cost |
| averageCostByAverage | double | 平均均价成本 Average cost by average |
| averageCostOfCarry | double | 摊薄持仓成本 Cost of carry (A股模式 A-share mode) |
| latestPrice | double | 市价 Market price（交易时段为市场价格，美股非交易时间：综合账号为盘后收盘价，环球账号为盘中收盘价） |
| isLevel0Price | boolean | 是否为lv0（延迟）行情 Level 0 (delayed) quote |
| marketValue | double | 市值 Market value |
| realizedPnl | double | 已实现盈亏 Realized P&L |
| realizedPnlByAverage | double | 均价模式下已实现盈亏 Realized P&L by average |
| unrealizedPnl | double | 浮动盈亏 Unrealized P&L |
| unrealizedPnlByAverage | double | 均价成本浮动盈亏 Unrealized P&L by average |
| unrealizedPnlPercent | double | 浮动收益率 Unrealized P&L % (ROI) |
| unrealizedPnlPercentByAverage | double | 均价成本浮动盈亏率 Unrealized P&L % by average |
| unrealizedPnlByCostOfCarry | double | 浮动盈亏 (A股模式) Unrealized P&L (A-share mode) |
| unrealizedPnlPercentByCostOfCarry | double | 浮动盈亏率 (A股模式) Unrealized P&L % (A-share mode) |
| multiplier | double | 每手数量 Multiplier |
| market | string | 市场 Market |
| currency | string | 交易货币币种 Trading currency |
| secType | string | 交易类型 Security type |
| identifier | string | 标的标识 Identifier |
| symbol | string | 股票代码 Symbol |
| strike | double | 期权底层价格 Strike (仅限期权 options only) |
| expiry | string | 期权过期日 Expiry (仅限期权 options only) |
| right | string | 期权方向 Put/Call (仅限期权 options only) |
| updateTimestamp | long | 更新时间 Update timestamp |
| mmPercent | double | 保证金占比 Maintenance margin percent |
| mmValue | double | 维持保证金 Maintenance margin value |
| todayPnl | double | 今日盈亏额 Today P&L |
| todayPnlPercent | double | 今日盈亏率 Today P&L % |
| yesterdayPnl | double | 基金昨日盈亏 Yesterday P&L (fund) |
| lastClosePrice | double | 最后盘中收盘价(前复权) Last close price (adjusted) |
| categories | `List<string>` | 合约类型 Contract categories |

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
PositionsRequest request = new PositionsRequest();

String bizContent = AccountParamBuilder.instance()
        .account("13810712")
        .secType(SecType.STK)
        .buildJson();

request.setBizContent(bizContent);
PositionsResponse response = client.execute(request);

// 解析具体字段
if (response.isSuccess()) {
    System.out.println(JSONObject.toJSONString(response));
    for (PositionDetail detail : response.getItem().getPositions()) {
      String account = detail.getAccount();
      String symbol = detail.getSymbol();
      long position = detail.getPosition();
      // ...
    }
} else {
    System.out.println(response.getMessage());
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "account": "13810712",
        "averageCost": 295.8904,
        "averageCostByAverage": 295.8904,
        "averageCostOfCarry": 591.7807,
        "currency": "USD",
        "identifier": "AAPL",
        "lastClosePrice": 232.98,
        "latestPrice": 232.44,
        "market": "US",
        "marketValue": -232.44,
        "multiplier": 1,
        "positionQty": -1,
        "realizedPnl": 0,
        "salableQty": -1,
        "secType": "STK",
        "symbol": "AAPL",
        "todayPnl": 0.54,
        "todayPnlPercent": 0.0023178,
        "unrealizedPnl": 63.4504,
        "unrealizedPnlByAverage": 63.4504,
        "unrealizedPnlPercent": 0.2144,
        "updateTimestamp": 1720685551097
      }
    ]
  },
  "message": "success"
}
```

---

## 历史资产分析 / Asset Analytics (PnL History)

**请求类 Request Class：`PrimeAnalyticsAssetRequest`**

获取综合/模拟账户的历史资产分析。
Get historical asset analytics for standard/paper accounts.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权资金账户 Authorized account |
| start_date | string | No | 起始日期 Start date, 格式 yyyy-MM-dd。如不传则使用 end_date 往前30天 Defaults to end_date - 30 days |
| end_date | string | No | 截止日期 End date, 格式 yyyy-MM-dd。如不传则使用当前日期 Defaults to today |
| seg_type | SegmentType | No | 账户划分类型 Segment type: SegmentType.SEC (证券 Securities) / SegmentType.FUT (期货 Futures) |
| currency | Currency | No | 币种 Currency: ALL/USD/HKD/CNH |
| sub_account | str | No | 子账户 Sub-account (仅适用于机构账户 institutional only) |
| secret_key | string | No | 机构用户专用，交易员密钥 Trader secret key, institutional only |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.PrimeAnalyticsAssetResponse`

可用 `PrimeAnalyticsAssetItem.Summary summary = primeAssetResponse.getSummary()` 获取资产分析汇总数据；
使用 `List<PrimeAnalyticsAssetItem.HistoryItem> historyItems = primeAssetResponse.getHistory()` 获取历史资产列表。

**`PrimeAnalyticsAssetItem.Summary` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| pnl | double | 盈亏金额 P&L amount |
| pnlPercentage | double | 收益率 Return rate |
| annualizedReturn | double | 年化收益率(推算) Annualized return (estimated) |
| overUserPercentage | double | 超过用户的百分比 Percentile rank among users |

**`PrimeAnalyticsAssetItem.HistoryItem` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| date | long | 日期时间戳，单位毫秒 Date timestamp (ms) |
| pnl | double | 与上一日比的盈亏金额 P&L vs previous day |
| pnlPercentage | double | 与上一日比的收益率 Return vs previous day |
| asset | double | 总资产金额 Total asset value |
| cashBalance | double | 现金余额 Cash balance |
| grossPositionValue | double | 市值 Gross position value |
| deposit | double | 入金金额 Deposit amount |
| withdrawal | double | 出金金额 Withdrawal amount |

**示例 Example**

```java
PrimeAnalyticsAssetRequest assetRequest = PrimeAnalyticsAssetRequest.buildPrimeAnalyticsAssetRequest(
    "402901").segType(SegmentType.SEC).startDate("2021-12-01").endDate("2021-12-07");
PrimeAnalyticsAssetResponse primeAssetResponse = client.execute(assetRequest);
if (primeAssetResponse.isSuccess()) {
  JSONObject.toJSONString(primeAssetResponse.getSummary());
  JSONObject.toJSONString(primeAssetResponse.getHistory());
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "summary": {
      "pnl": 691.18,
      "pnlPercentage": 0,
      "annualizedReturn": 0,
      "overUserPercentage": 0
    },
    "history": [
      {
        "date": 1638334800000,
        "asset": 48827609.65,
        "pnl": 0,
        "pnlPercentage": 0,
        "cashBalance": 48811698.59,
        "grossPositionValue": 15911.06,
        "deposit": 0,
        "withdrawal": 0
      },
      {
        "date": 1638421200000,
        "asset": 48827687.69,
        "pnl": 78.04,
        "pnlPercentage": 0,
        "cashBalance": 48811698.59,
        "grossPositionValue": 15989.1,
        "deposit": 0,
        "withdrawal": 0
      }
    ]
  }
}
```

---

## 获取最大可交易数量 / Estimate Tradable Quantity

**请求类 Request Class：`EstimateTradableQuantityRequest`**

查询账户下某个标的的最大可买可卖数量，支持股票、期权，暂不支持期货。
Query maximum tradable quantity for a symbol. Supports stocks and options, not futures.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | String | Yes | 账户 Account (目前仅支持综合账户 standard only) |
| symbol | String | Yes | 股票代码 Symbol |
| expiry | String | No | 到期日 Expiry (OPT/WAR/IOPT时必传)，格式 yyyyMMdd |
| right | String | No | CALL/PUT (OPT/WAR/IOPT时必传) |
| strike | String | No | 行权价 Strike (OPT/WAR/IOPT时必传) |
| seg_type | SegmentType | No | SEC，暂只支持SEC |
| sec_type | SecType | No | STK/FUT/OPT/WAR/IOPT (期货暂不支持 futures not supported) |
| action | ActionType | Yes | 交易方向 Action: BUY/SELL |
| order_type | OrderType | Yes | 订单类型 Order type |
| limit_price | double | No | 限价 Limit price (order_type 为 LMT/STP_LMT 时必需) |
| stop_price | double | No | 止损价 Stop price (order_type 为 STP/STP_LMT 时必需) |
| secretKey | String | No | 交易员密钥，机构用户专用 Trader secret key, institutional only |

**返回字段 Response Fields**

| 字段 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| tradableQuantity | Double | 现金可买可卖数量 Cash tradable quantity (BUY=可买, SELL=可卖) |
| financingQuantity | Double | 融资融券可买可卖数量 Financing tradable quantity (不适用于现金账户 N/A for cash accounts) |
| positionQuantity | Double | 持仓数量 Position quantity |
| tradablePositionQuantity | Double | 持仓可交易数量 Tradable position quantity |

**示例 Example**

```java
EstimateTradableQuantityRequest request = EstimateTradableQuantityRequest.buildRequest(
        SecType.STK, "AAPL", ActionType.BUY, OrderType.LMT, 150D, null);

EstimateTradableQuantityResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("fail." + JSONObject.toJSONString(response));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": {
    "positionQuantity": 1,
    "tradablePositionQuantity": 1,
    "tradableQuantity": 45
  },
  "message": "success"
}
```

---

## 综合/模拟账号可转出资金 / Segment Fund Available

**请求类 Request Class：`SegmentFundAvailableRequest`**

获取账户在对应 Segment 下的可转出资金。
Get available funds for transfer from a segment.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账号 Authorized account (标准/模拟 standard/paper) |
| from_segment | string | Yes | 转出 segment: FUT 或 SEC |
| currency | string | No | 转出币种 Currency: USD 或 HKD |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.SegmentFundAvailableResponse`

可用 `List<SegmentFundAvailableItem> items = response.getSegmentFundAvailableItems()` 获取列表。

**`SegmentFundAvailableItem` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| fromSegment | string | 转出 segment: FUT 或 SEC |
| currency | string | 转出币种 Currency: USD 或 HKD |
| amount | double | 可转资金 Available amount (单位：对应币种的元) |

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);

SegmentFundAvailableRequest request = SegmentFundAvailableRequest.buildRequest(
        SegmentType.SEC, Currency.HKD);

SegmentFundAvailableResponse response = client.execute(request);
System.out.println(JSONObject.toJSONString(response));

// 获取具体数据
for (SegmentFundAvailableItem item : response.getSegmentFundAvailableItems()) {
  System.out.println(JSONObject.toJSONString(item));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": [
    {
      "amount": 17607412.84,
      "currency": "HKD",
      "fromSegment": "SEC"
    }
  ],
  "message": "success"
}
```

---

## 综合/模拟账号内部资金转账 / Segment Fund Transfer

**请求类 Request Class：`SegmentFundTransferRequest`**

对账户在不同 Segment 下的资金转账，如证券 segment 转期货 segment。
Transfer funds between segments (e.g., securities to futures).

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账号 Authorized account (标准/模拟 standard/paper) |
| from_segment | string | Yes | 转出 segment: FUT 或 SEC |
| to_segment | string | Yes | 转入 segment: FUT 或 SEC（必须与 from_segment 不同 must differ from from_segment） |
| currency | string | Yes | 转出币种 Currency: USD 或 HKD |
| amount | double | Yes | 转账金额 Transfer amount (单位：对应币种的元) |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.SegmentFundResponse`

可用 `SegmentFundItem item = response.getSegmentFundItem()` 获取转账结果对象。

**`SegmentFundItem` 说明 Fields**

| 名称 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| id | long | 转账记录 ID Transfer record ID |
| fromSegment | string | 转出 segment: FUT 或 SEC |
| toSegment | string | 转入 segment: FUT 或 SEC |
| currency | string | 转出币种 Currency: USD 或 HKD |
| amount | double | 转账金额 Transfer amount |
| status | string | 状态 Status: NEW/PROC/SUCC/FAIL/CANC |
| statusDesc | string | 状态描述 Status description: 已提交/处理中/已到账/转账失败/已取消 |
| message | string | 失败信息 Error message |
| settledAt | long | 到账时间戳 Settled timestamp |
| updatedAt | long | 更新时间戳 Updated timestamp |
| createdAt | long | 创建时间戳 Created timestamp |

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
SegmentFundTransferRequest request = SegmentFundTransferRequest.buildRequest(
        SegmentType.SEC, SegmentType.FUT, Currency.HKD, 1000D);

SegmentFundResponse response = client.execute(request);
System.out.println(JSONObject.toJSONString(response));
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": {
    "amount": 1000,
    "createdAt": 1680076000672,
    "currency": "HKD",
    "fromSegment": "SEC",
    "id": 30300805635506176,
    "status": "NEW",
    "statusDesc": "已提交",
    "toSegment": "FUT",
    "updatedAt": 1680076000672
  },
  "message": "success"
}
```

---

## 取消内部资金转账 / Cancel Segment Fund Transfer

**请求类 Request Class：`SegmentFundCancelRequest`**

取消提交的资金转账。如果转账已成功，则无法取消，可以交换 segment 反向转回来。
Cancel a submitted fund transfer. If already successful, cannot cancel -- reverse transfer instead.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账号 Authorized account (标准/模拟 standard/paper) |
| id | long | Yes | 转账记录 id Transfer record ID |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.SegmentFundResponse`

返回字段同 SegmentFundItem（见上方资金转账章节）。
Response fields same as SegmentFundItem (see fund transfer section above).

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);
SegmentFundCancelRequest request = SegmentFundCancelRequest.buildRequest(30300805635506176L);

SegmentFundResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("cancel fail." + JSONObject.toJSONString(response));
}
```

---

## 内部资金转账历史记录 / Segment Fund Transfer History

**请求类 Request Class：`SegmentFundHistoryRequest`**

查询账号不同 Segment 间的历史转账记录，按时间倒序排列。
Query historical fund transfer records between segments, ordered by time descending.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | string | Yes | 用户授权账号 Authorized account (标准/模拟 standard/paper) |
| limit | Integer | No | 返回最近转账记录数 Max records。默认100，最大500 Default 100, max 500 |

**返回类 Response Class**

`com.tigerbrokers.stock.openapi.client.https.response.trade.SegmentFundsResponse`

可用 `List<SegmentFundItem> items = response.getSegmentFundItems()` 获取历史转账记录列表。

返回字段同 SegmentFundItem（见上方资金转账章节）。
Response fields same as SegmentFundItem (see fund transfer section above).

**示例 Example**

```java
TigerHttpClient client = TigerHttpClient.getInstance().clientConfig(
      ClientConfig.DEFAULT_CONFIG);

SegmentFundHistoryRequest request = SegmentFundHistoryRequest.buildRequest(30);
SegmentFundsResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("query fail." + JSONObject.toJSONString(response));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": [
    {
      "amount": 1000,
      "createdAt": 1680076001000,
      "currency": "HKD",
      "fromSegment": "SEC",
      "id": 30300805635506176,
      "settledAt": 1680076001000,
      "status": "SUCC",
      "statusDesc": "已到账",
      "toSegment": "FUT",
      "updatedAt": 1680076001000
    }
  ],
  "message": "success"
}
```

---

## 获取出入金记录 / Deposit & Withdrawal Records

**请求类 Request Class：`DepositWithdrawRequest`**

查询账户的出入金记录。Query deposit and withdrawal records.

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| account | String | Yes | 账户 Account (目前仅支持综合账户 standard only) |
| secretKey | String | No | 机构用户专用，交易员密钥 Trader secret key, institutional only |
| lang | Language | No | 语言 Language: en_US / zh_CN / zh_TW，默认 en_US |

**返回字段 Response Fields**

| 字段 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| id | long | ID |
| refId | string | 关联业务ID Reference ID |
| type | int | 资金类型 Fund type: 1(入金 Deposit), 3(出金 Withdrawal), 20(出金费用), 21(出金退款), 22(出金失败-退款), 23(出金费用-退款) |
| typeDesc | string | 资金类型描述 Type description |
| currency | string | 币种 Currency |
| amount | double | 金额 Amount |
| businessDate | string | 业务日期 Business date |
| completedStatus | bool | 是否结束 Completed |
| createdAt | long | 创建时间戳 Created timestamp |
| updatedAt | long | 更新时间戳 Updated timestamp |

**示例 Example**

```java
DepositWithdrawRequest request = DepositWithdrawRequest.newRequest();
request.lang(Language.en_US);

DepositWithdrawResponse response = client.execute(request);
if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("fail." + JSONObject.toJSONString(response));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "data": [
    {
      "amount": 300000,
      "businessDate": "2024/11/15",
      "completedStatus": true,
      "createdAt": 1731664787127,
      "currency": "USD",
      "id": 24924145879575129,
      "type": 1,
      "typeDesc": "Deposit",
      "updatedAt": 1731664787127
    }
  ],
  "message": "success"
}
```

---

## 获取资金明细 / Fund Details

**请求类 Request Class：`FundDetailsRequest`**

获取资金明细（按类型、币种、日期筛选）。
Get fund details (filter by type, currency, date).

**参数 Parameters**

| 参数 Parameter | 类型 Type | 是否必填 Required | 描述 Description |
|---------------|----------|----------------|-----------------|
| segTypes | List\<String> | Yes | Segment 类型列表, 如 ["SEC", "FUT"] |
| account | String | Yes | 账户 Account (目前仅支持综合账户 standard only) |
| fundType | FundType | No | 资金类型 Fund type: ALL/DEPOSIT_WITHDRAW/TRADE/FEE/FUNDS_TRANSFER/FOREX/CORPORATE_ACTION/ACTIVITY_AWARD/OTHER。默认 ALL |
| currency | String | No | 币种 Currency |
| startDate | String | No | 开始日期 Start date, 格式 yyyy-MM-dd |
| endDate | String | No | 结束日期 End date, 格式 yyyy-MM-dd |
| start | Long | No | 起始序号 Offset, 从0开始。如每页50条，前两页共100条，第3页传100 |
| limit | Long | No | 最大返回条数 Max records, 默认50，最大100 |
| secretKey | String | No | 机构用户专用，交易员密钥 Trader secret key, institutional only |
| lang | Language | No | 语言 Language: en_US / zh_CN / zh_TW，默认 en_US |

**返回字段 Response Fields**

| 字段 Field | 类型 Type | 说明 Description |
|-----------|----------|-----------------|
| id | Long | 记录ID Record ID |
| desc | String | 描述 Description |
| currency | String | 币种 Currency |
| segType | String | Segment type |
| type | String | 资金类型 Fund type |
| amount | Double | 金额 Amount |
| businessDate | String | 业务日期 Business date |
| updatedAt | Long | 流水更新时间戳 Updated timestamp |
| contractName | String | 合约名称 Contract name |
| page | Integer | 当前页码 Current page |
| limit | Integer | 每页记录数量 Records per page |
| itemCount | Integer | 总记录数量 Total record count |
| pageCount | Integer | 总页数 Total pages |
| timestamp | Long | 时间戳 Timestamp |

**示例 Example**

```java
FundDetailsRequest request = FundDetailsRequest.buildFundDetailsRequest(account,
    Lists.newArrayList(SegmentType.SEC.name()), 0L, 5L);
request.setCurrency("HKD");
request.setFundType("ALL");
request.setStartDate("2025-01-01");
request.setEndDate("2025-04-01");
FundDetailsResponse response = client.execute(request);

if (response.isSuccess()) {
  System.out.println(JSONObject.toJSONString(response));
} else {
  System.out.println("fail." + JSONObject.toJSONString(response));
}
```

**返回示例 Response Example**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "page": 1,
    "limit": 5,
    "itemCount": 5,
    "pageCount": 1,
    "items": [
      {
        "id": "3969803304",
        "currency": "HKD",
        "type": "Internal Funds Transfer Out",
        "segType": "SEC",
        "amount": -1,
        "businessDate": "2025-03-25",
        "updatedAt": 1742875771000
      },
      {
        "id": "3942009038",
        "currency": "HKD",
        "type": "Financing Tnterest",
        "segType": "SEC",
        "amount": -2.43,
        "businessDate": "2025-03-06",
        "updatedAt": 1741247923000
      }
    ]
  }
}
```

---

## 注意事项 / Notes

- 环球账户(Global)使用 `MethodName.ASSETS` 查询资产，综合/模拟账户(Standard/Paper)推荐使用 `PrimeAssetRequest`。Global accounts use `MethodName.ASSETS`; standard/paper accounts should use `PrimeAssetRequest`.
- Segment 分类 Categories: S (Securities 证券), C (Commodities 期货), F (Funds 基金), D (Digi 数字货币)。
- `consolidated=true` 时，SEC 和 FUND 类别的部分指标会聚合显示。When consolidated is true, some metrics for SEC and FUND segments are aggregated.
- 资金转账状态 Transfer statuses: NEW(已提交), PROC(处理中), SUCC(已到账), FAIL(转账失败), CANC(已取消)。
- 持仓中 `position` 和 `positionScale` 字段已废弃，请使用 `positionQty`。The `position` and `positionScale` fields are deprecated; use `positionQty` instead.
