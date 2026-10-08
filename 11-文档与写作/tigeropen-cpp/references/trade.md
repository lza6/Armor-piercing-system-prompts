# Tiger OpenAPI C++ SDK — Trading / 交易

> C++ SDK 交易 API 参考 / Trade API Reference
<!-- 当用户提到"下单"、"买入"、"卖出"、"撤单"、"改单"、"持仓"、"资产"、"order"、"trade"时 -->

## 安全规范 / Safety Rules

> ⚠️ **默认使用模拟账户。Default to Paper Trading.**

实盘下单前，**每步均为必须，缺少任何步骤不得下单**：
1. 调用 `preview_order()` 查看预估佣金和保证金，展示给用户
2. 将订单详情（标的、方向、数量、价格、账户、预估佣金）以表格展示，**停止等待用户明确确认**；未收到确认前**禁止调用 `place_order()`**
3. 用户确认后调用 `place_order()`
4. 下单后通过 `get_orders()` 确认订单状态

---

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/trade_client.h"
#include "tigerapi/contract_util.h"
#include "tigerapi/order_util.h"

using namespace std;
using namespace web;
using namespace web::json;
using namespace TIGER_API;

ClientConfig config(false, "path/to/config/");
auto tc = make_shared<TradeClient>(config);
```

---

## 构造合约 / Build Contract
<!-- ContractUtil 是构造 Contract 对象的工厂 -->

```cpp
// 股票合约 / Stock contract
Contract stock = ContractUtil::stock_contract(U("AAPL"), U("USD"));
Contract hk_stock = ContractUtil::stock_contract(U("00700"), U("HKD"));

// 期权合约 / Option contract
Contract opt = ContractUtil::option_contract(
    U("AAPL"),    // 标的
    U("20250829"), // 到期日 YYYYMMDD
    U("150.0"),    // 行权价
    U("CALL"),     // CALL / PUT
    U("USD")       // 货币
);

// 从 identifier 构造期权 / From identifier
Contract opt2 = ContractUtil::option_contract(U("AAPL 250829C00150000"));

// 期货合约 / Future contract
Contract fut = ContractUtil::future_contract(U("CL2509"), U("USD"));
```

---

## 构造订单 / Build Order
<!-- OrderUtil 是构造 Order 对象的工厂 -->

```cpp
utility::string_t account = config.account;

// 限价单 / Limit order（limit_order 有不带 account 的重载）
Order lmt_order = OrderUtil::limit_order(stock, U("BUY"), 100, 150.0);

// 市价单 / Market order
Order mkt_order = OrderUtil::market_order(account, stock, U("BUY"), 100);

// 止损限价单 / Stop-limit order
Order stp_lmt = OrderUtil::stop_limit_order(account, stock, U("SELL"), 100, 145.0, 148.0);
// 参数: account, contract, action, qty, limitPrice, auxPrice(触发价)
// ⚠️ 除 limit_order 外，其他 OrderUtil 工厂方法【都必须】传 account 作为第一个参数
// Only limit_order has an account-less overload; all other factories require account first.

// 盘后交易 / After-hours
lmt_order.outside_rth = true;
lmt_order.time_in_force = U("GTC");
```

---

## 预览订单 / Preview Order

```cpp
Contract contract = ContractUtil::stock_contract(U("AAPL"), U("USD"));
Order order = OrderUtil::limit_order(contract, U("BUY"), 1, 100.0);

value preview = tc->preview_order(order);
ucout << U("preview: ") << preview << endl;
// 返回预估保证金、佣金等信息
```

---

## 下单 / Place Order
<!-- 当用户提到"买入"、"卖出"、"下单"、"buy"、"sell"时 -->

```cpp
Contract contract = ContractUtil::stock_contract(U("AAPL"), U("USD"));
Order order = OrderUtil::limit_order(contract, U("BUY"), 1, 100.0);

value result = tc->place_order(order);
unsigned long long id = result[U("id")].as_number().to_uint64();
ucout << U("order id: ") << id << endl;
ucout << U("result: ") << result << endl;
// 返回: id, orderId, status: "PendingSubmit"
```

---

## 修改订单 / Modify Order

```cpp
// modify_order(order, newLimitPrice) — 修改限价
value res = tc->modify_order(order, 105.0);
ucout << U("modify result: ") << res << endl;
```

---

## 撤单 / Cancel Order

```cpp
long long order_id = 100000000000000002;
value res = tc->cancel_order(order_id);
ucout << U("cancel result: ") << res << endl;
```

---

## 查询订单 / Query Orders
<!-- 当用户提到"订单"、"委托"、"orders"时 -->

```cpp
// 所有订单 / All orders
value orders = tc->get_orders();
ucout << orders << endl;

// 待成交订单 / Active (pending) orders
value active = tc->get_active_orders();

// 已成交订单 / Filled orders (需提供时间范围)
auto now_ms = chrono::duration_cast<chrono::milliseconds>(
    chrono::system_clock::now().time_since_epoch()).count();
auto ninety_days_ago = now_ms - 90LL * 24 * 3600 * 1000;
value filled = tc->get_filled_orders(U(""), U(""), U("ALL"), U(""), ninety_days_ago, now_ms);

// 已撤销订单 / Inactive (cancelled) orders
value inactive = tc->get_inactive_orders();

// 单个订单 / Single order
Order my_order = tc->get_order(31318009878020096);
ucout << my_order.to_string() << endl;

// 成交明细 / Transactions
value txns = tc->get_transactions(config.account, 31318009878020096);
// 按 symbol 查询成交
value txns2 = tc->get_transactions(config.account, U("AAPL"));
```

---

## 持仓查询 / Query Positions
<!-- 当用户提到"持仓"、"仓位"、"positions"时 -->

```cpp
// 所有持仓 / All positions (json)
value positions = tc->get_positions();
ucout << positions << endl;

// 结构化持仓列表 / Structured list
vector<Position> pos_list = tc->get_position_list();
for (auto& p : pos_list) {
    ucout << p.to_string() << endl;
}

// 按条件查询 / Filtered query
// get_positions(account, sec_type, currency, market, symbol, sub_accounts, expiry, strike, right)
value filtered = tc->get_positions(U(""), U("STK"), U(""), U("US"), U("AAPL"));
```

---

## 资产查询 / Query Assets
<!-- 当用户提到"资产"、"资金"、"余额"、"assets"时 -->

```cpp
// 综合账户资产 (Prime) / Prime account assets
value prime = tc->get_prime_asset();
ucout << prime << endl;

// 结构化持仓组合 / Structured portfolio
PortfolioAccount portfolio = tc->get_prime_portfolio();
ucout << portfolio.to_string() << endl;
for (auto& seg : portfolio.segments) {
    ucout << U("segment: ") << seg.to_string() << endl;
}

// 普通账户资产 / Standard account assets
value asset = tc->get_asset();
ucout << asset << endl;

// 综合资产（多货币汇总）/ Aggregate assets
value aggregate = tc->get_aggregate_assets();
```

---

## 合约查询 / Contract Query

```cpp
// 单个合约 / Single contract
value contract_info = tc->get_contract(U("AAPL"));
ucout << contract_info << endl;

// 批量合约 / Multiple contracts
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));
symbols[1] = value::string(U("TSLA"));
value contracts = tc->get_contracts(symbols, U("STK"));

// 港股期权合约 / HK option contract
value hk_opt = tc->get_quote_contract(U("00700"), U("IOPT"), U("20230728"));
```

---

## 账户管理 / Account Management

```cpp
// 获取账户列表 / Get accounts
value accounts = tc->get_accounts();
ucout << accounts << endl;

// 预估可交易数量 / Estimate tradable quantity
Contract c = ContractUtil::stock_contract(U("AAPL"), U("USD"));
Order o = OrderUtil::limit_order(c, U("BUY"), 1, 100.0);
value qty = tc->get_estimate_tradable_quantity(o);

// 资金历史分析 / Analytics asset
value analytics = tc->get_analytics_asset(config.account, U("2024-01-01"), U("2024-12-31"));
```

---

## Order 对象字段 / Order Fields

| 字段 Field | 说明 | 示例 |
|-----------|------|------|
| `symbol` | 标的代码 | `"AAPL"` |
| `sec_type` | 证券类型 | `"STK"/"OPT"/"FUT"` |
| `action` | 买卖方向 | `"BUY"/"SELL"` |
| `order_type` | 订单类型 | `"LMT"/"MKT"/"STP_LMT"` |
| `total_quantity` | 数量 | `100` |
| `limit_price` | 限价 | `150.0` |
| `aux_price` | 止损触发价 | `148.0` |
| `time_in_force` | 有效期 | `"DAY"/"GTC"/"GTD"` |
| `outside_rth` | 允许盘前盘后 | `true` |
| `id` | 内部订单 ID（下单后由服务端返回） | — |

---

## 注意事项 / Notes

- `place_order()` 返回成功仅表示订单**已提交**，不代表已成交
- 需通过 `get_orders()` 或推送回调确认最终成交状态
- `get_filled_orders()` 需要提供 `start_date`（建议不超过 90 天）
- 期权下单需先通过 `qc->get_option_chain()` 获取合约 identifier
- `ContractUtil` 和 `OrderUtil` 是构造合约和订单的辅助工具类
