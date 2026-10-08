# Tiger OpenAPI C++ SDK — Real-time Push / 实时推送

> C++ SDK 实时推送 API 参考 / Push API Reference
<!-- 当用户提到"推送"、"实时"、"订阅"、"WebSocket"、"push"、"subscribe"时 -->

## 初始化 / Initialize

```cpp
#include "tigerapi/push_client.h"
#include "tigerapi/client_config.h"

using namespace std;
using namespace TIGER_API;

ClientConfig config(false, "path/to/config/");

// 工厂方法创建推送客户端 / Factory method
shared_ptr<IPushClient> pc = IPushClient::create_push_client(config);
```

---

## 完整示例 / Full Example

```cpp
#include <atomic>
#include <csignal>
#include <thread>
#include <chrono>
#include "tigerapi/push_client.h"
#include "tigerapi/client_config.h"
#include "cpprest/details/basic_types.h"

using namespace std;
using namespace TIGER_API;

atomic<bool> keep_running(true);

void signal_handler(int sig) {
    if (sig == SIGINT || sig == SIGTERM) keep_running = false;
}

int main() {
    ClientConfig config(false, "path/to/config/");
    shared_ptr<IPushClient> pc = IPushClient::create_push_client(config);

    string account = utility::conversions::to_utf8string(config.account);

    // 1. 注册回调 / Register callbacks
    pc->set_connected_callback([&]() {
        ucout << U("Connected!") << endl;
        // 在连接成功回调中执行订阅
        pc->subscribe_quote({"AAPL", "TSLA"});
        pc->subscribe_asset(account);
        pc->subscribe_order(account);
        pc->subscribe_position(account);
    });

    pc->set_disconnected_callback([]() {
        ucout << U("Disconnected.") << endl;
    });

    pc->set_inner_error_callback([](string err) {
        cerr << "Error: " << err << endl;
    });

    pc->set_quote_changed_callback([](const tigeropen::push::pb::QuoteBasicData& data) {
        ucout << U("Quote: ") << utility::conversions::to_string_t(data.symbol())
              << U(" price=") << data.latestprice()
              << U(" volume=") << data.volume() << endl;
    });

    pc->set_asset_changed_callback([](const tigeropen::push::pb::AssetData& data) {
        ucout << U("Asset: cash=") << data.cashbalance()
              << U(" net=") << data.netliquidation() << endl;
    });

    pc->set_order_changed_callback([](const tigeropen::push::pb::OrderStatusData& data) {
        ucout << U("Order: id=") << data.id()
              << U(" status=") << utility::conversions::to_string_t(data.status()) << endl;
    });

    pc->set_position_changed_callback([](const tigeropen::push::pb::PositionData& data) {
        ucout << U("Position: ") << utility::conversions::to_string_t(data.symbol())
              << U(" qty=") << data.positionqty() << endl;
    });

    // 2. 连接 / Connect
    pc->connect();

    // 3. 等待信号 / Wait for signal
    signal(SIGINT, signal_handler);
    signal(SIGTERM, signal_handler);
    while (keep_running) {
        this_thread::sleep_for(chrono::seconds(1));
    }

    // 4. 取消订阅并断开 / Unsubscribe and disconnect
    pc->unsubscribe_quote({"AAPL", "TSLA"});
    pc->unsubscribe_asset(account);
    pc->unsubscribe_order(account);
    pc->unsubscribe_position(account);
    pc->disconnect();
    return 0;
}
```

---

## 回调注册方法 / Callback Registration

| 方法 Method | 触发时机 | 数据类型 |
|------------|---------|---------|
| `set_connected_callback(cb)` | WebSocket 连接成功时 | `void()` |
| `set_disconnected_callback(cb)` | 连接断开时 | `void()` |
| `set_inner_error_callback(cb)` | 内部错误时 | `void(string)` |
| `set_error_callback(cb)` | 服务端错误响应 | `void(Response&)` |
| `set_kickout_callback(cb)` | 被踢出时 | `void(Response&)` |
| `set_quote_changed_callback(cb)` | 行情更新时 | `void(QuoteBasicData&)` |
| `set_quote_bbo_changed_callback(cb)` | BBO 更新时 | `void(QuoteBBOData&)` |
| `set_quote_depth_changed_callback(cb)` | 深度行情更新 | `void(QuoteDepthData&)` |
| `set_tick_changed_callback(cb)` | 逐笔成交（简化） | `void(TradeTick&)` |
| `set_full_tick_changed_callback(cb)` | 逐笔成交（完整） | `void(TickData&)` |
| `set_kline_changed_callback(cb)` | K 线更新时 | `void(KlineData&)` |
| `set_asset_changed_callback(cb)` | 账户资产变动时 | `void(AssetData&)` |
| `set_order_changed_callback(cb)` | 订单状态变化时 | `void(OrderStatusData&)` |
| `set_position_changed_callback(cb)` | 持仓变动时 | `void(PositionData&)` |
| `set_transaction_changed_callback(cb)` | 成交明细推送 | `void(OrderTransactionData&)` |
| `set_heartbeat_callback(cb)` | 心跳时 | `void()` |

---

## 订阅方法 / Subscribe Methods

### 行情订阅 / Quote Subscription

```cpp
// 股票行情 / Stock quotes
pc->subscribe_quote({"AAPL", "TSLA", "00700"});
pc->unsubscribe_quote({"TSLA"});

// 深度行情 / Quote depth
pc->subscribe_quote_depth({"AAPL"});
pc->unsubscribe_quote_depth({"AAPL"});

// K 线 / Kline
pc->subscribe_kline({"AAPL"});
pc->unsubscribe_kline({"AAPL"});

// 逐笔成交 / Tick
pc->subscribe_tick({"AAPL"});
pc->unsubscribe_tick({"AAPL"});

// 期权行情 / Option quotes
pc->subscribe_option_quote({"AAPL 250829C00150000"});

// 期货行情 / Future quotes
pc->subscribe_future_quote({"CL2509"});

// 整个市场 / Whole market
pc->subscribe_market("US");
pc->unsubscribe_market("US");
```

### 账户推送 / Account Push

```cpp
// account 需传 std::string（UTF-8），用 to_utf8string 转换
string account = utility::conversions::to_utf8string(config.account);

pc->subscribe_asset(account);
pc->subscribe_order(account);
pc->subscribe_position(account);
pc->subscribe_transaction(account);

pc->unsubscribe_asset(account);
pc->unsubscribe_order(account);
pc->unsubscribe_position(account);
pc->unsubscribe_transaction(account);
```

---

## 推送数据字段 / Push Data Fields

### QuoteBasicData（行情）

| 字段 | 类型 | 说明 |
|------|------|------|
| `symbol()` | string | 标的代码 |
| `latestprice()` | double | 最新价 |
| `volume()` | int64 | 成交量 |
| `bidprice()` | double | 买一价 |
| `askprice()` | double | 卖一价 |
| `change()` | double | 涨跌额 |
| `changeratio()` | double | 涨跌幅 |
| `timestamp()` | string | 时间戳 |

### AssetData（资产）

| 字段 | 类型 | 说明 |
|------|------|------|
| `account()` | string | 账户号 |
| `netliquidation()` | double | 净资产 |
| `cashbalance()` | double | 现金余额 |
| `buyingpower()` | double | 购买力 |
| `availablefunds()` | double | 可用资金 |

### OrderStatusData（订单）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id()` | string | 订单 ID |
| `symbol()` | string | 标的代码 |
| `status()` | string | 订单状态 |
| `avgfillprice()` | double | 平均成交价 |
| `filledquantity()` | int | 已成交数量 |

### PositionData（持仓）

| 字段 | 类型 | 说明 |
|------|------|------|
| `account()` | string | 账户号 |
| `symbol()` | string | 标的代码 |
| `positionqty()` | int | 持仓数量 |
| `averagecost()` | double | 平均成本 |
| `marketprice()` | double | 最新价格 |
| `marketvalue()` | double | 市值 |
| `unrealizedpnl()` | double | 浮动盈亏 |

---

## 查询当前订阅 / Query Subscriptions

```cpp
// 注册回调（先注册再查询）
pc->set_query_subscribed_symbols_changed_callback(
    [](const tigeropen::push::pb::Response& data) {
        ucout << U("Subscribed symbols: ")
              << utility::conversions::to_string_t(data.msg()) << endl;
    }
);

// 触发查询
pc->query_subscribed_symbols();
```

---

## 注意事项 / Notes

- **在 `set_connected_callback` 回调中执行订阅**，连接成功后才能订阅
- `connect()` 是非阻塞的，会启动后台工作线程处理 IO
- 账户类推送的 account 参数需为 `std::string`（UTF-8），用 `utility::conversions::to_utf8string()` 转换
- SDK 不自动重连，如需断线重连需自行在 `set_disconnected_callback` 中处理
- `std::bind` 可用于绑定类成员函数作为回调
