
# Tiger OpenAPI C++ SDK — 账户管理 / Account Management

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/accounts

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/contract_util.h"
#include "tigerapi/order_util.h"

using namespace TIGER_API;

ClientConfig config(false, "path/to/config/dir/");
auto trade_client = make_shared<TradeClient>(config);
```

---

## 账户列表 / Account List

SDK 方法 / SDK method: `get_accounts()`

```cpp
// SDK 已封装 get_accounts()，返回所有账号（综合、环球、模拟）
// The SDK wraps this API; it returns all accounts (standard, global, paper).
value result = trade_client->get_accounts();
ucout << result << endl;

// 返回字段 / Response fields:
// account     - 账户号（综合5~10位数字，模拟17位，环球以U开头）
// capability  - 账户类型：CASH（现金）/ RegTMargin（保证金）/ PMGRN（组合保证金）
// status      - 状态：Funded（已入金）/ Open（已开户）/ Pending（待开户）/ Rejected（被拒）/ Closed（已注销）
// accountType - 分类：STANDARD（综合）/ GLOBAL（环球）/ PAPER（模拟）
```

---

## 账户资产 / Account Assets

### 环球账户 / Global Account (ASSETS)

```cpp
// SDK 已封装 get_asset(account="", sub_accounts=array(), segment=false, market_value=false)
// The SDK wraps this API; no raw post() needed.
// segment      - 按证券/期货分类 / group by security vs future segment
// market_value - 按市场分市值（仅环球）/ per-market values (global accounts only)
value result = trade_client->get_asset(U("DU000001"), value::array(), true, true);
ucout << result << endl;

// 主要返回字段 / Key response fields:
// netLiquidation  - 净清算值
// availableFunds  - 可用资金
// buyingPower     - 购买力
// cashValue       - 现金
// initMarginReq   - 初始保证金要求
// maintMarginReq  - 维持保证金要求
// unrealizedPnl   - 浮动盈亏
// realizedPnl     - 已实现盈亏
// segments        - 按交易品种（S=证券, C=期货）分类资产
// marketValues    - 按市场（USD/HKD）分类资产
```

### 综合/模拟账号 / Standard/Paper Account (PRIME_ASSETS)

```cpp
// SDK 已封装 get_prime_asset(account="", base_currency="USD")
// 另有 Currency 枚举重载 / An overload taking the Currency enum also exists.
// The SDK wraps this API; no raw post() needed.
value result = trade_client->get_prime_asset(U("123456"), U("USD"));
ucout << result << endl;

// 枚举重载 / Enum overload:
// value r2 = trade_client->get_prime_asset(U("123456"), Currency::USD);

// ⚠️ wrapper 未开放 consolidated（SEC+FUND 聚合显示）参数，
// 需要该字段时才退回裸 post(PRIME_ASSETS, obj)。
// The wrapper does not expose `consolidated`; use raw post() if you need it.

// segments 数组主要字段 / Segment key fields:
// category            - S（证券）/ C（期货）/ F（基金）/ D（数字货币）
// capability          - RegTMargin / Cash
// buyingPower         - 最大购买力（保证金账户日内4倍，隔夜2倍）
// cashAvailableForTrade - 可用资金（用于判断能否开仓）
// cashBalance         - 现金余额
// netLiquidation      - 净清算值
// initMargin          - 初始保证金
// maintainMargin      - 维持保证金（低于此值会强平）
// unrealizedPL        - 浮动盈亏
// currencyAssets      - 按币种（USD/HKD/SGD/CNH）细分资产
```

---

## 账户持仓 / Account Positions

```cpp
// SDK 已封装 get_positions(account, sec_type="", currency="ALL", market="ALL", symbol="", ...)
// 另有 SecType / Currency / Market 枚举重载
// The SDK wraps this API; enum overloads (SecType/Currency/Market) also exist.
value result = trade_client->get_positions(U("123456"), U("STK"), U("ALL"), U("ALL"));
ucout << result << endl;

// 枚举重载 / Enum overload:
// value r2 = trade_client->get_positions(U("123456"), SecType::STK,
//                                       Currency::ALL, Market::ALL);

// 返回数组，每条持仓主要字段 / Key position fields:
// symbol        - 股票代码
// positionQty   - 持仓数量
// averageCost   - 平均成本（FIFO）
// marketValue   - 市值
// unrealizedPnl - 浮动盈亏
// secType       - 证券类型（STK/OPT等）
// market        - 市场
// currency      - 币种
```

### 期权持仓 / Option Positions

```cpp
value result = trade_client->get_positions(U("123456"), U("OPT"), U("ALL"), U("ALL"));
// 期权持仓额外字段：strike（行权价）, expiry（到期日）, right（CALL/PUT）
```

---

## 历史资产分析 / Asset Analytics (PnL History)

```cpp
// SDK 已封装 get_analytics_asset(account, start_date, end_date,
//                                seg_type="SEC", currency="USD", sub_account="")
// The SDK wraps this API; no raw post() needed.
value result = trade_client->get_analytics_asset(U("123456"),
                                                U("2026-01-01"), U("2026-01-31"),
                                                U("SEC"), U("USD"));
ucout << result << endl;

// summary 字段 / Summary fields:
// pnl                - 盈亏金额
// pnlPercentage      - 收益率
// annualizedReturn   - 年化收益率

// history 数组每项字段 / History item fields:
// date               - 日期时间戳（毫秒）
// asset              - 总资产
// pnl                - 当日盈亏
// cashBalance        - 现金余额
// grossPositionValue - 持仓市值
// deposit            - 入金
// withdrawal         - 出金
```

---

## 最大可交易数量 / Estimate Tradable Quantity

```cpp
// SDK 已封装 get_estimate_tradable_quantity(order, seg_type="SEC")
// 入参是一个 Order 对象，SDK 内部拆出 symbol/sec_type/action/limit_price 等字段
// The SDK wraps this API; pass an Order and it extracts the request fields.
Contract est_contract = ContractUtil::stock_contract(U("AAPL"), U("USD"));
Order est_order = OrderUtil::limit_order(U("123456"), est_contract, U("BUY"), 100, 150.0);

value result = trade_client->get_estimate_tradable_quantity(est_order, U("SEC"));
ucout << result << endl;

// 返回字段 / Response fields:
// tradableQuantity          - 现金可买/卖数量
// financingQuantity         - 融资融券可买/卖数量
// positionQuantity          - 持仓数量
// tradablePositionQuantity  - 持仓可交易数量
```

---

## 资金转账（Segment 间）/ Segment Fund Transfer

### 查询可转出金额 / Query Available Amount

```cpp
// SDK 已封装 get_segment_fund_available(from_segment, currency="USD")
// account 由 ClientConfig 提供，无需显式传入
// The SDK wraps this API; the account comes from ClientConfig.
value result = trade_client->get_segment_fund_available(U("SEC"), U("USD"));  // SEC / FUT
ucout << result << endl;
// fromSegment, currency, amount
```

### 发起转账 / Transfer

```cpp
// SDK 已封装 transfer_segment_fund(from_segment, to_segment, amount, currency="USD")
// The SDK wraps this API; no raw post() needed.
value result = trade_client->transfer_segment_fund(U("SEC"), U("FUT"), 1000.0, U("USD"));
ucout << result << endl;
// 转账状态 / Transfer status: NEW / PROC / SUCC / FAIL / CANC
```

---

## 出入金记录 / Deposit & Withdrawal Records

```cpp
// SDK 已封装 get_funding_history(seg_type="")，内部走 TRANSFER_FUND
// The SDK wraps this API as get_funding_history() (posts to TRANSFER_FUND).
value result = trade_client->get_funding_history(U("SEC"));  // SEC / FUT，留空返回全部
ucout << result << endl;

// 每条记录字段 / Record fields:
// type         - 1(入金) / 3(出金) / 20(出金费用) 等
// typeDesc     - 类型描述
// currency     - 币种
// amount       - 金额
// businessDate - 业务日期
```

---

## 注意事项 / Notes

- 环球账户(Global)使用 `get_asset()`，综合/模拟账户(Standard/Paper)使用 `get_prime_asset()`
- Segment 分类：S=证券, C=期货, F=基金, D=数字货币
- 持仓中 `positionQty` 为正确字段，旧字段 `position`+`positionScale` 已废弃
- 维持保证金 `maintainMargin` < 0 时会被强制平仓
- 机构用户的 `secret_key` 由 SDK wrapper 自动注入（`set_secret_key`），`sub_account` 通过
  wrapper 的对应参数传入；wrapper 未开放的字段才退回裸 `post(CONSTANT, obj)`
