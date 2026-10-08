# Tiger OpenAPI Rust SDK — Quickstart

> Rust SDK 快速入门 / Quick Start for Rust SDK
> GitHub: https://github.com/tigerfintech/openapi-rust-sdk

## 安装 / Installation

在 `Cargo.toml` 中添加依赖：

```toml
[dependencies]
tigeropen = "0.5"
tokio = { version = "1", features = ["full"] }
serde_json = "1"
```

要求 / Requirements: Rust 1.70+，tokio async runtime

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Builder pattern

```rust
use tigeropen::config::ClientConfig;

let config = ClientConfig::builder()
    .tiger_id("your_tiger_id")
    .private_key("your_rsa_private_key")  // PEM 格式私钥字符串
    .account("your_account")
    .build()?;
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```rust
let config = ClientConfig::builder()
    .properties_file("tiger_openapi_config.properties")
    .build()?;
```

配置文件格式 / Config file format:
```properties
tiger_id=your_tiger_id
private_key=your_rsa_private_key
account=your_account
```

### 方式三：环境变量 / Method 3: Environment variables

```bash
export TIGEROPEN_TIGER_ID=your_tiger_id
export TIGEROPEN_PRIVATE_KEY=your_rsa_private_key
export TIGEROPEN_ACCOUNT=your_account
```

支持的环境变量仅这五个 / Only these five env vars are read:
`TIGEROPEN_TIGER_ID`、`TIGEROPEN_PRIVATE_KEY`、`TIGEROPEN_ACCOUNT`、
`TIGEROPEN_TOKEN`、`TIGEROPEN_TOKEN_FILE`。

未显式设置必填字段时，SDK 会按 `./tiger_openapi_config.properties` →
`~/.tigeropen/tiger_openapi_config.properties` 顺序自动查找配置文件；
显式调用过 `.properties_file()` 则跳过自动查找。
If required fields are unset, the SDK auto-discovers a properties file in that order.

### 配置项说明 / Config Options

| Builder 方法 | 说明 | 必填 |
|-------------|------|------|
| `.tiger_id(id)` | 开发者 ID | ✅ |
| `.private_key(key)` | RSA 私钥 PEM 字符串 | ✅ |
| `.account(account)` | 交易账户号 | - |
| `.secret_key(key)` | 机构用户 secret key | - |
| `.properties_file(path)` | .properties 文件路径 | - |
| `.license(license)` | 牌照类型（如 `"TBNZ"`） | - |
| `.language(Language::EnUs)` | `Language::ZhCn` / `Language::EnUs` | - |
| `.timezone(tz)` | 时区字符串 | - |
| `.timeout(Duration::from_secs(30))` | 请求超时（默认 15s） | - |
| `.enable_dynamic_domain(bool)` | 动态域名 | - |
| `.token(token)` | 直接注入 token | - |
| `.token_refresh_duration(d)` | token 自动刷新周期 | - |
| `.device_id(id)` | 设备 ID | - |

---

## 客户端创建 / Create Clients

```rust
use std::sync::Arc;
use tigeropen::client::http_client::HttpClient;
use tigeropen::config::ClientConfig;
use tigeropen::push::PushClient;
use tigeropen::quote::QuoteClient;
use tigeropen::trade::TradeClient;

let config = ClientConfig::builder()
    .tiger_id("your_tiger_id")
    .private_key("your_rsa_private_key")
    .account("your_account")
    .build()?;

// 推荐：每个客户端从 config 构造（HttpClient 不实现 Clone）
// Recommended: build each client from config — HttpClient is NOT Clone
let qc = QuoteClient::from_config(config.clone());
let tc = TradeClient::from_config(config.clone());

// PushClient 需要包在 Arc 中才能连接
// PushClient must be wrapped in an Arc to connect
let pc = Arc::new(PushClient::new(config.clone(), None));
```

若要手动构造 `HttpClient`，注意两点 / When building `HttpClient` manually, note:

```rust
// 1. HttpClient::new 返回 Self，不是 Result —— 不要加 `?`
//    HttpClient::new returns Self, not Result — do NOT add `?`
// 2. 客户端按【值】接收 HttpClient，不是引用；且 HttpClient 不可 clone，
//    所以一个 HttpClient 只能交给一个客户端
//    Clients take HttpClient BY VALUE (not by reference), and it is not Clone
{
    let http_client = HttpClient::new(config.clone());
    let tc_manual = TradeClient::new(http_client, config.account.clone());
}
```

机构用户可注入 secret key / Institutional users can inject a secret key:

```rust
let tc3 = TradeClient::with_secret_key(
    HttpClient::new(config.clone()),
    config.account.clone(),
    "your_secret_key",
);
```

---

## 通用 API 调用 / Generic API Call

当 SDK 未封装某个 API 时，用 `execute`。第二个参数是 **JSON 字符串**，返回 **String**。
Use `execute`; the second arg is a **JSON string** and it returns a **String**.
（没有 `execute_raw` 方法 / there is no `execute_raw` method.）

```rust
let http_client = HttpClient::new(config.clone());
let raw = http_client.execute("market_state", r#"{"market":"US"}"#).await?;
println!("{}", raw);
```

需要动态构造入参时先序列化 / Serialize dynamic params first:

```rust
use serde_json::json;

let body = json!({"market": "US"}).to_string();
let raw2 = http_client.execute("market_state", &body).await?;
```

---

## 错误处理 / Error Handling

`TigerError::Api` 是**结构体变体**，不是元组变体 / `TigerError::Api` is a **struct variant**:

```rust
use tigeropen::error::TigerError;
use tigeropen::model::quote_requests::BriefRequest;

match qc.get_real_time_quote(BriefRequest {
    symbols: Some(vec!["AAPL".to_string()]),
    ..Default::default()
}).await {
    Ok(briefs) => {
        for b in briefs {
            println!("{} {:?}", b.symbol, b.latest_price);
        }
    }
    Err(TigerError::Api { code, message }) => eprintln!("API 错误 code={} msg={}", code, message),
    Err(TigerError::Network(e)) => eprintln!("网络错误: {}", e),
    Err(TigerError::Auth(msg)) => eprintln!("认证错误: {}", msg),
    Err(TigerError::Config(msg)) => eprintln!("配置错误: {}", msg),
    Err(TigerError::Parse(msg)) => eprintln!("解析错误: {}", msg),
}
```

`TigerError` 共五个变体：`Api { code, message }`、`Network`、`Auth`、`Config`、`Parse`。
穷尽匹配时五个都要覆盖 / All five must be covered in an exhaustive match.

---

## 完整示例 / Full Example

```rust
use tigeropen::client::http_client::HttpClient;
use tigeropen::config::ClientConfig;
use tigeropen::model::quote_requests::BriefRequest;
use tigeropen::model::trade_requests::OrdersRequest;
use tigeropen::quote::QuoteClient;
use tigeropen::trade::TradeClient;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = ClientConfig::builder()
        .tiger_id("your_tiger_id")
        .private_key("your_rsa_private_key")
        .account("your_account")
        .build()?;

    // 查询实时行情 / Query real-time quotes
    let qc = QuoteClient::from_config(config.clone());
    let briefs = qc
        .get_real_time_quote(BriefRequest {
            symbols: Some(vec!["AAPL".to_string(), "TSLA".to_string()]),
            ..Default::default()
        })
        .await?;
    for b in &briefs {
        println!("{} latest={:?}", b.symbol, b.latest_price);
    }

    // 查询订单 / Query orders
    let tc = TradeClient::new(HttpClient::new(config.clone()), config.account.clone());
    let orders = tc
        .get_orders(OrdersRequest {
            limit: Some(20),
            ..Default::default()
        })
        .await?;
    println!("orders: {}", orders.len());

    Ok(())
}
```

---

## Token 自动刷新 / Token Auto-refresh

```rust
// 查询当前 token / Query the current token
let token = qc.query_token().await?;

// 手动刷新（token_manager 可传 None）/ Manual refresh (token_manager may be None)
qc.refresh_token(None).await?;

// 后台自动刷新：返回 Arc<TokenManager>，需保留句柄以便停止
// Background auto-refresh: keep the returned handle to stop it later
let tm = qc.start_token_auto_refresh(86400, 300, None);
// tm.stop_auto_refresh();
```

`TradeClient` 上有同名方法 / The same methods exist on `TradeClient`.

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥、`account`
3. 行情数据需要对应的行情权限；期权行情需要期权行情权限

---

## FAQ

**Q: 私钥如何生成?**
参考官方文档 https://docs.itigerup.com/docs/prepare，使用 RSA-2048 生成密钥对，上传公钥到开发者后台。

**Q: 返回值格式?**
封装好的 API 返回强类型结构体（如 `Vec<Brief>`、`Vec<Order>`），直接取字段。
只有底层的 `http_client.execute()` 返回 `String`。

**Q: `HttpClient::new` 要不要加 `?`?**
不要。它返回 `Self`，加 `?` 会编译失败。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请。SDK 根据账号自动识别模拟/实盘账户并路由到对应域名，无需额外配置。

**Q: `async` 运行时要求?**
必须在 tokio 运行时中使用，通常在 `main` 函数加 `#[tokio::main]`。
