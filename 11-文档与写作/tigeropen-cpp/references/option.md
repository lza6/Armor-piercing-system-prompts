
# Tiger OpenAPI C++ SDK — 期权 / Options Trading

> 中文 | English — 双语技能。Bilingual skill.
> 官方文档 Docs: https://docs.itigerup.com/docs/quote-option

## 期权操作工作流 / Option Workflow

当用户提到期权时，按以下流程操作 / When user mentions options, follow this workflow:

### 查询期权 / Query Options

1. **查到期日 Get expirations**: `get_option_expiration()` → 获取可选到期日列表
2. **查期权链 Get chain**: `get_option_chain()` → 获取指定到期日的所有合约，可按 Greeks 筛选
3. **查行情 Get quotes**: `get_option_brief()` → 获取期权实时行情和希腊字母

### 港股期权特殊处理 / HK Option Special Handling

- 港股期权标的代码不同于正股 / HK option underlyings differ from stock codes: `00700` → `TCH`（腾讯）
- 使用 `get_option_symbols(U("HK"))` 查询港股期权代码映射

---

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/quote_client.h"
#include "tigerapi/contract_util.h"
#include "tigerapi/order_util.h"

using namespace TIGER_API;

ClientConfig config(false, "path/to/config/dir/");
auto trade_client = make_shared<TradeClient>(config);
auto quote_client = make_shared<QuoteClient>(config);
```

---

## 期权到期日 / Option Expirations

```cpp
// SDK 已封装 get_option_expiration(symbols)
// The SDK wraps this API; no raw post() needed.
value exp_symbols = value::array();
exp_symbols[0] = value::string(U("AAPL"));

value result = quote_client->get_option_expiration(exp_symbols);
ucout << result << endl;

// 返回字段 / Response fields:
// symbol     - 股票代码
// count      - 到期日数量
// dates      - 到期日数组 (e.g. "2024-06-28")
// timestamps - 到期日时间戳数组（毫秒，纽约时间）
// periodTags - 期权周期标签: "m"=月期权, "w"=周期权, "q"=季度
```

---

## 期权链 / Option Chain

```cpp
// 不带筛选时用 wrapper（expiry 支持 "yyyy-MM-dd" 或毫秒时间戳 time_t 重载）
// Unfiltered: use the wrapper. expiry accepts "yyyy-MM-dd" or an epoch-ms time_t.
value result = quote_client->get_option_chain(U("AAPL"), U("2026-08-21"));
ucout << result << endl;

// 返回字段 / Response fields (items[].call / items[].put):
// identifier   - 期权完整代码 (e.g. "AAPL  250829C00150000")
// strike       - 行权价
// right        - "CALL" / "PUT"
// askPrice     - 卖价
// bidPrice     - 买价
// latestPrice  - 最新价
// volume       - 成交量
// openInterest - 持仓量
// impliedVol   - 隐含波动率
// delta        - Delta（需 return_greek_value=true）
// gamma        - Gamma
// theta        - Theta
// vega         - Vega
// rho          - Rho
```

> ⚠️ **`get_option_chain` 的 `option_filter` 参数当前不生效**：wrapper 只用
> symbol + expiry 构造请求体，从不读取 `option_filter`
> （`src/quote_client.cpp:260-269`）。需要按 Greeks / IV / 持仓量筛选时，
> **必须走裸 `post`**，否则会静默拿到未筛选的全量期权链。
> The `option_filter` argument is accepted but never read by the wrapper, so
> filters silently have no effect. Use the raw `post` form below when filtering.

```cpp
// 需要筛选时用裸 post / raw post when you need filtering
value obj = value::object(true);
value basic = value::object(true);
basic[U("symbol")] = value::string(U("AAPL"));
basic[U("expiry")] = value::string(U("2026-08-21"));
value option_basic = value::array();
option_basic[0] = basic;
obj[U("option_basic")] = option_basic;

// 筛选条件 / filter criteria
obj[U("in_the_money")] = value::boolean(true);
obj[U("implied_volatility_min")] = value::number(0.15);
obj[U("implied_volatility_max")] = value::number(0.80);
obj[U("delta_min")] = value::number(0.2);
obj[U("delta_max")] = value::number(0.8);
obj[U("open_interest_min")] = value::number(100);
obj[U("return_greek_value")] = value::boolean(true);  // 返回 Greeks

value filtered = quote_client->post(OPTION_CHAIN, obj);
ucout << filtered << endl;
```

---

## 港股期权代码映射 / HK Option Symbol Mapping

```cpp
// 港股期权标的代码与股票代码不同，需先查询映射
// HK option underlying symbols differ from stock codes
// SDK 已封装 get_option_symbols(market="HK", lang="")
// The SDK wraps this API; no raw post() needed.

value result = quote_client->get_option_symbols(U("HK"));
ucout << result << endl;

// 返回字段 / Response fields:
// symbol           - 期权四要素 symbol (e.g. "TCH.HK")
// name             - 标的名称
// underlyingSymbol - 正股代码 (e.g. "00700")

// 然后使用映射后的代码查询港股期权 / Then use mapped symbol:
value hk_symbols = value::array();
hk_symbols[0] = value::string(U("TCH.HK"));
value hk_expirations = quote_client->get_option_expiration(hk_symbols);
```

---

## 期权实时行情 / Option Brief (Real-time Quotes)

```cpp
// SDK 已封装 get_option_brief(identifiers) / get_option_brief(identifier)
// 入参是期权完整代码（identifier），SDK 内部拆解为 option_basic（最多30条）
// The SDK wraps this API; pass option identifiers (max 30), not option_basic objects.

value brief_ids = value::array();
brief_ids[0] = value::string(U("AAPL  260821C00150000"));
brief_ids[1] = value::string(U("AAPL  260821P00150000"));

value result = quote_client->get_option_brief(brief_ids);
ucout << result << endl;

// 单个合约也可直接传字符串 / Single contract overload:
// value one = quote_client->get_option_brief(U("AAPL  260821C00150000"));

// 返回字段 / Response fields:
// identifier    - 期权完整代码
// symbol        - 标的代码
// bidPrice      - 买盘价格
// askPrice      - 卖盘价格
// latestPrice   - 最新价
// volume        - 成交量
// openInterest  - 未平仓量
// high/low/open - 最高/最低/开盘
// impliedVol    - 隐含波动率（通过分析接口获取）
// midPrice      - 中间价
// markPrice     - 标记价格
// sellingReturn - 卖出年化收益率
```

---

## 期权深度行情 / Option Depth Quotes

```cpp
// SDK 已封装 get_option_depth(symbols, market="US")
// symbols 即 option_basic 数组，SDK 内部包成 {option_basic, market}
// The SDK wraps this API; `symbols` is the option_basic array.

value depth_basic = value::array();
value depth_item = value::object(true);
depth_item[U("symbol")] = value::string(U("AAPL"));
depth_item[U("right")] = value::string(U("PUT"));
depth_item[U("expiry")] = value::string(U("2026-08-21"));
depth_item[U("strike")] = value::string(U("210.0"));
depth_basic[0] = depth_item;

value result = quote_client->get_option_depth(depth_basic, U("US"));
ucout << result << endl;

// 返回字段 / Response fields:
// ask[]/bid[] - 卖/买盘挂单列表
//   price    - 委托价
//   volume   - 委托量
//   code     - 交易所代码 (CBOE, PHLX 等)
//   timestamp - 交易所时间
```

---

## 期权逐笔成交 / Option Trade Ticks

```cpp
// 仅支持美股期权 / US market only
// SDK 已封装 get_option_trade_tick(identifiers)
// 入参是期权完整代码（identifier），SDK 内部拆解为 option_basic
// The SDK wraps this API; pass option identifiers, not option_basic objects.

value tick_ids = value::array();
tick_ids[0] = value::string(U("AAPL  260821P00185000"));

value result = quote_client->get_option_trade_tick(tick_ids);
ucout << result << endl;

// 返回字段 / Response fields:
// items[] - 逐笔成交列表
//   price  - 成交价
//   volume - 成交量
//   time   - 成交时间（毫秒）
```

---

## 期权K线 / Option K-line

```cpp
// SDK 已封装 get_option_kline_value(identifiers, begin_time, end_time=4070880000000)
// 入参是期权完整代码（identifier）+ 毫秒时间戳；wrapper 固定 period="day"
// The SDK wraps this API; identifiers + epoch-ms range, period is fixed to "day".

value kline_ids = value::array();
kline_ids[0] = value::string(U("AAPL  260821C00170000"));

value result = quote_client->get_option_kline_value(kline_ids,
                                                   1755734400000LL,   // 2026-08-21 00:00 ET
                                                   1755820799000LL);  // 2026-08-21 23:59 ET
ucout << result << endl;

// 需要 1min/5min/30min/60min 等分钟级周期时，SDK wrapper 未开放 period 参数，
// 此时才退回裸 post(OPTION_KLINE, obj) 自行构造 option_query。
// Minute-level periods are not exposed by the wrapper; fall back to raw post() then.

// 返回字段 / Response fields:
// items[] - K线数据点列表
//   open/high/low/close - 开高低收
//   volume    - 成交量
//   time      - 时间戳（毫秒）
//   openInterest - 持仓量（仅日K线）
```

---

## 期权分时数据 / Option Timeline

```cpp
// 目前仅支持港股期权 / HK market only currently
// SDK 已封装 get_option_timeline(symbols, market="US", begin_time=-1)
// symbols 会被包成 option_query 字段（注意不是 option_list）
// The SDK wraps this API; `symbols` is placed under the option_query field.

value tl_query = value::array();
value tl_item = value::object(true);
tl_item[U("symbol")] = value::string(U("ALB.HK"));
tl_item[U("right")] = value::string(U("CALL"));
tl_item[U("expiry")] = value::number(1753878054000LL);  // 毫秒时间戳
tl_item[U("strike")] = value::string(U("117.50"));
tl_query[0] = tl_item;

value result = quote_client->get_option_timeline(tl_query, U("HK"));
ucout << result << endl;

// 返回字段 / Response fields:
// preClose    - 昨日收盘价
// minutes[]  - 分时数据点
//   price     - 最新价
//   avgPrice  - 均价
//   volume    - 成交量
//   time      - 时间戳（毫秒）
```

---

## 期权分析 / Option Analysis

```cpp
// SDK 已封装 get_option_analysis(symbols, market="US", lang="")
// The SDK wraps this API; no raw post() needed.
value symbols = value::array();
value sym_item = value::object(true);
sym_item[U("symbol")] = value::string(U("AAPL"));
sym_item[U("period")] = value::string(U("52week"));  // 3year/52week/26week/13week
symbols[0] = sym_item;

value result = quote_client->get_option_analysis(symbols, U("US"));
ucout << result << endl;

// 返回字段 / Response fields:
// symbol          - 标的代码
// impliedVol30Days - 30日隐含波动率
// hisVolatility   - 历史波动率（30天）
// ivHisVRatio     - IV/HV 比率
// callPutRatio    - Call/Put 比率
// impliedVolMetric:
//   percentile  - IV百分位（0%-100%）
//   rank        - IV排名（0-1）
//   period      - 分析周期
```

---

## 单合约指标与 Greeks / Per-contract Greeks

SDK **没有** `OPTION_INDICATOR` 接口。单个合约的 Greeks / 隐含波动率通过
期权链（带 Greeks）或期权行情获取。
There is no `OPTION_INDICATOR` API; get Greeks from the option chain or option brief.

```cpp
// 方式一：期权链带 Greeks（option_filter 可传 greeks 条件）
value chain_greeks = quote_client->get_option_chain(U("AAPL"), U("2026-08-21"));

// 方式二：单合约行情（identifier 形式）
value brief_one = quote_client->get_option_brief(U("AAPL  260821C00150000"));
ucout << brief_one << endl;

// 返回含 delta/gamma/theta/vega/rho、隐含波动率、未平仓量、内在价值与时间价值等
```

---

## 期权合约 / Option Contract

```cpp
// 构造期权合约参数用于下单
// Build option contract parameters for order placement

// 期权代码格式 / Option identifier format:
// 美股 US: "AAPL  250829C00150000"
//   格式: 标的(padding至6位) + YYMMDD + C/P + 行权价*1000(8位)
// 港股 HK: "TCH.HK 230616C00550000"
//   使用映射后的代码 / Use mapped symbol from get_option_symbols(U("HK"))
```

---

## 单腿期权下单 / Single-leg Option Order

```cpp
// 买入看涨期权 / Buy call option
// SDK 已封装：ContractUtil::option_contract + OrderUtil + place_order
// The SDK wraps this flow; no raw post(PLACE_ORDER, obj) needed.

// option_contract(symbol, expiry, strike, right, currency="USD", multiplier=100, ...)
// expiry 为 YYYYMMDD / expiry is YYYYMMDD
Contract opt_contract = ContractUtil::option_contract(U("AAPL"), U("20260821"),
                                                      U("150.0"), U("CALL"), U("USD"));

// 1张 = 100股 / 1 contract = 100 shares
Order opt_order = OrderUtil::limit_order(U("123456"), opt_contract, U("BUY"), 1, 5.0);

value result = trade_client->place_order(opt_order);
ucout << result << endl;

// 也可用 identifier 重载构造合约 / Or build the contract from an identifier:
// Contract c2 = ContractUtil::option_contract(U("AAPL  260821C00150000"));
```

---

## 多腿组合策略 / Multi-leg Combo Strategies

组合单用 `OrderUtil::multi_leg_order` 构造后交给 `place_order`。
**没有** `PLACE_COMBO_ORDER` 常量。
Build combos with `OrderUtil::multi_leg_order` + `place_order`; there is no
`PLACE_COMBO_ORDER` constant.

```cpp
// 牛市看涨价差 / Bull Call Spread (VERTICAL)
// ContractLeg(sec_type, symbol, strike, expiry, right, action, ratio)
std::vector<ContractLeg> legs;
legs.push_back(ContractLeg(U("OPT"), U("AAPL"), U("145.0"), U("20260821"),
                           U("CALL"), U("BUY"), 1));
legs.push_back(ContractLeg(U("OPT"), U("AAPL"), U("155.0"), U("20260821"),
                           U("CALL"), U("SELL"), 1));

// multi_leg_order(account, combo_type, legs, action, quantity, order_type,
//                 limit_price = 0, aux_price = 0, trailing_percent = 0)
Order combo = OrderUtil::multi_leg_order(U("123456"), U("VERTICAL"), legs,
                                         U("BUY"), 1, U("LMT"), 3.0);
value combo_result = trade_client->place_order(combo);
ucout << combo_result << endl;
```

> ⚠️ 组合单（以及 OCA / 附加止盈止损单）**不支持 `preview_order`**，服务端返回
> `OCA/ATTACHED order preview not supported`（错误码 1010），请直接 `place_order`。
> Combo/OCA/attached orders do NOT support `preview_order` (server error 1010).

### 组合策略类型 / Combo Strategy Types

| ComboType | 策略 Strategy | 说明 Description |
|-----------|--------------|-----------------|
| `VERTICAL` | 垂直价差 | 同到期日不同行权价 Same expiry, different strikes |
| `STRADDLE` | 跨式 | 同行权价同到期日 Call+Put, same strike & expiry |
| `STRANGLE` | 宽跨式 | 不同行权价同到期日 Call+Put, different strikes |
| `CALENDAR` | 日历价差 | 同行权价不同到期日 Same strike, different expiries |
| `DIAGONAL` | 对角线价差 | 不同行权价不同到期日 Different strikes & expiries |
| `COVERED` | 备兑 | 持有股票+卖Call Long stock + short call |
| `PROTECTIVE` | 保护性 | 持有股票+买Put Long stock + long put |
| `SYNTHETIC` | 合成 | 合成多/空头 Synthetic long/short |
| `CUSTOM` | 自定义 | 4条腿组合（Iron Condor等） 4-leg combos |

---

## 查询期权持仓 / Query Option Positions

```cpp
// SDK 已封装 get_positions(account, sec_type, currency, market, symbol, ...)
// The SDK wraps this API; no raw post() needed.
value result = trade_client->get_positions(U("123456"), U("OPT"));
ucout << result << endl;

// 期权持仓额外字段 / Option-specific position fields:
// strike  - 行权价
// expiry  - 到期日
// right   - CALL/PUT
```

---

## 注意事项 / Notes

- 港股期权需先用 `get_option_symbols(U("HK"))` 获取代码映射 / HK options require symbol mapping
- 期权每张合约通常代表100股标的 / Each contract = 100 shares
- 期权链返回的 Greeks 为上一交易日收盘值 / Chain Greeks are from previous close
- 行权价小数位须和期权链一致 / Strike decimals must match the option chain
- 期权行情需要期权行情权限 / Option quotes require option quote permission
- 机构用户的 `secret_key` 由 SDK wrapper 自动注入；wrapper 未开放的字段
  （如 `sub_account`）才退回裸 `post(CONSTANT, obj)` 自行构造
