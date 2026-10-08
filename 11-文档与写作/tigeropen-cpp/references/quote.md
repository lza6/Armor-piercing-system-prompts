# Tiger OpenAPI C++ SDK — Market Data / 行情查询

> C++ SDK 行情 API 参考 / Quote API Reference
<!-- 当用户提到"行情"、"报价"、"K线"、"价格"、"深度"、"quote"、"kline"、"price"时 -->

## 初始化 / Initialize

```cpp
#include "tigerapi/client_config.h"
#include "tigerapi/quote_client.h"

using namespace std;
using namespace web;
using namespace web::json;
using namespace TIGER_API;

ClientConfig config(false, "path/to/config/");
auto qc = make_shared<QuoteClient>(config);
```

---

## 市场状态 / Market State

```cpp
// market: "US" / "HK" / "CN" / "SG"
value result = qc->get_market_state(U("US"));
ucout << result << endl;
```

---

## 实时报价 / Real-time Quotes
<!-- 当用户提到"实时报价"、"最新价"、"real-time"时 -->

```cpp
// 返回 web::json::value（原始 JSON）
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));
symbols[1] = value::string(U("TSLA"));

value result = qc->get_brief(symbols);
ucout << result << endl;

// 返回结构化 vector / Structured result
vector<RealtimeQuote> quotes = qc->get_quote_real_time(symbols);
for (auto& q : quotes) {
    ucout << q.to_string() << endl;
}
```

`get_brief` 签名：
```cpp
value get_brief(const value& symbols,
                bool include_hour_trading = false,
                bool include_ask_bid = false,
                QuoteRight right = QuoteRight::br);
```

---

## K 线 / Kline
<!-- 当用户提到"K线"、"kline"、"bar"、"日线"时 -->

```cpp
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));

// period: "day"/"week"/"month"/"year"/"1min"/"3min"/"5min"/"15min"/"30min"/"60min"
value result = qc->get_kline(symbols, U("day"), -1, -1, U("br"), 251, U(""));
ucout << result << endl;

// 结构化返回 / Structured return
vector<Kline> klines = qc->get_kline(symbols, U("day"), -1, -1, 251);
for (auto& k : klines) {
    ucout << k.to_string() << endl;
}

// 使用枚举 / Use enum
value result2 = qc->get_kline(symbols, BarPeriod::DAY, -1, -1, QuoteRight::br, 251);
```

---

## 分时 / Timeline

```cpp
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));

value result = qc->get_timeline(symbols);
// 返回: time, price, volume, avgPrice 等

// 历史分时 / Historical timeline
value hist = qc->get_history_timeline(symbols, U("2024-01-15"));
```

---

## 深度行情 / Quote Depth
<!-- 当用户提到"买卖盘"、"深度"、"depth"时 -->

```cpp
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));

// Market: Market::US / Market::HK
value result = qc->get_quote_depth(symbols, Market::US);
ucout << result << endl;
// 返回: asks/bids 各含 price, volume 的数组
```

---

## 逐笔成交 / Trade Ticks

```cpp
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));

// begin_index=-1, end_index=-1 获取最新
value result = qc->get_trade_tick(symbols, -1, -1);
ucout << result << endl;
```

---

## 期权到期日 / Option Expirations

```cpp
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));

value result = qc->get_option_expiration(symbols);
// 返回: dates 数组（字符串日期）, timestamps 数组, periodTag ("m"=月度/"w"=周度)
ucout << result << endl;
```

---

## 期权链 / Option Chain

```cpp
// 使用日期字符串 / Use date string
value result = qc->get_option_chain(U("AAPL"), U("2025-08-29"));
ucout << result << endl;

// 使用时间戳（毫秒）/ Use timestamp (ms)
time_t expiry_ts = 1756425600000;
value result2 = qc->get_option_chain(U("AAPL"), expiry_ts);
```

返回每个合约包含：`identifier`, `strike`, `right`(CALL/PUT), `bid`, `ask`, `volume`, `openInterest`, `impliedVol`, `delta`, `gamma` 等

---

## 期权报价 / Option Brief

```cpp
// 单个期权 / Single option
// 格式: "SYMBOL  YYMMDD[C/P]STRIKE*1000(8位)"
value result = qc->get_option_brief(U("AAPL 230224C000150000"));
ucout << result << endl;

// 多个期权 / Multiple options
value identifiers = value::array();
identifiers[0] = value::string(U("AAPL 250829C00150000"));
identifiers[1] = value::string(U("AAPL 250829P00150000"));
value result2 = qc->get_option_brief(identifiers);
```

---

## 期货 / Futures

```cpp
// 期货交易所 / Future exchanges
value exchanges = qc->get_future_exchange();
ucout << exchanges << endl;

// 期货合约列表 / Future contracts by exchange
value contracts = qc->get_future_contracts(U("CME"));
ucout << contracts << endl;

// 期货实时行情 / Future real-time quotes
value symbols = value::array();
symbols[0] = value::string(U("CL2509"));
value contract_codes = value::array();
contract_codes[0] = value::string(U("CLmain"));
vector<RealtimeQuote> quotes = qc->get_future_real_time_quote(contract_codes);
for (auto& q : quotes) {
    ucout << q.to_string() << endl;
}

// 期货 K 线 / Future kline
vector<Kline> klines = qc->get_future_kline(symbols, BarPeriod::DAY);

// 获取当前主力合约 / Get current main contract
value current = qc->get_future_current_contract(U("CL"));
utility::string_t code = current[U("contractCode")].as_string();
ucout << U("Current CL contract: ") << code << endl;
```

---

## 资金流向 / Capital Flow

```cpp
// symbol, market, period
// period: CapitalPeriod::DAY / WEEK / MONTH / INTRADAY
value result = qc->get_capital_flow(U("AAPL"), Market::US, CapitalPeriod::DAY);
ucout << result << endl;

// 资金分布 / Capital distribution
value dist = qc->get_capital_distribution(U("AAPL"), Market::US);
ucout << dist << endl;
```

---

## 其他行情 / Other Quote APIs

```cpp
// 股票基本信息 / Stock detail
value symbols = value::array();
symbols[0] = value::string(U("AAPL"));
value detail = qc->get_stock_detail(symbols);

// 港股经纪商席位 / HK stock broker seats
value broker = qc->get_stock_broker(U("00700"), 40);

// 交易日历 / Trading calendar
value calendar = qc->get_trading_calendar(Market::US, U("2025-01-01"), U("2025-01-31"));

// 市场扫描 / Market scanner
// 先获取可用标签
value scanner_tags = qc->get_market_scanner_tags(U("US"), value::array());

// 所有 symbol 列表 / All symbols
value all_symbols = qc->get_symbols(Market::US);
```

---

## JSON 字段访问 / Parse JSON Result

```cpp
value result = qc->get_brief(symbols);

// result 是 json::array
for (const auto& item : result.as_array()) {
    utility::string_t symbol = item.at(U("symbol")).as_string();
    double latest_price = item.at(U("latestPrice")).as_double();
    ucout << symbol << U(": ") << latest_price << endl;
}
```

---

## 注意事项 / Notes

- 所有方法均为**同步调用**，直接返回结果（无 `async`/`await`）
- `value` 返回类型为 `web::json::value`，可用 `.as_array()`, `.as_string()`, `.as_double()` 等解析
- `U("str")` 宏用于跨平台字符串（Windows 宽字符兼容）
- 行情数据需要对应市场的行情权限
- 港股期权代码格式：`TCH.HK 20241230 410.00 CALL`（与美股不同）
