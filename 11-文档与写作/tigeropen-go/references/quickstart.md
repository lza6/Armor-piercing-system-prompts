# Tiger OpenAPI Go SDK — Quickstart

> Go SDK 快速入门 / Quick Start for Go SDK
> GitHub: https://github.com/tigerfintech/openapi-go-sdk

## 安装 / Installation

```bash
go get github.com/tigerfintech/openapi-go-sdk@latest
```

要求 / Requirements: Go 1.20+

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Code assignment

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/config"
)

cfg, err := config.NewClientConfig(
    config.WithTigerID("your_tiger_id"),
    config.WithPrivateKey("your_rsa_private_key"),  // PEM 格式私钥字符串
    config.WithAccount("your_account"),
)
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```go
cfg, err := config.NewClientConfig(
    config.WithPropertiesFile("tiger_openapi_config.properties"),
)
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

### 配置项说明 / Config Options

| 选项 Option | 函数 Function | 说明 | 必填 |
|------------|--------------|------|------|
| Tiger ID | `WithTigerID` | 开发者 ID | ✅ |
| 私钥 | `WithPrivateKey` | RSA 私钥 PEM 字符串 | ✅ |
| 账户 | `WithAccount` | 交易账户号 | ✅ |
| 配置文件 | `WithPropertiesFile` | .properties 文件路径 | - |
| 语言 | `WithLanguage` | `zh_CN` / `en_US` | - |
| 超时 | `WithTimeout` | 请求超时时长（默认 15s） | - |

---

## 客户端创建 / Create Clients

```go
import (
    "github.com/tigerfintech/openapi-go-sdk/client"
    "github.com/tigerfintech/openapi-go-sdk/quote"
    "github.com/tigerfintech/openapi-go-sdk/trade"
    "github.com/tigerfintech/openapi-go-sdk/push"
)

httpClient := client.NewHttpClient(cfg)

qc := quote.NewQuoteClient(httpClient)               // 行情客户端
tc := trade.NewTradeClient(httpClient, cfg.Account)  // 交易客户端
pc := push.NewPushClient(cfg)                        // 推送客户端
```

也可直接从 config 创建，省去手动构造 `HttpClient` / You can also build clients straight from config:

```go
qc := quote.NewQuoteClientFromConfig(cfg)
tc := trade.NewTradeClientFromConfig(cfg)
```

---

## 通用 API 调用 / Generic API Call

当 SDK 尚未封装某个 API 时，使用 `httpClient.ExecuteRaw` 直接调用。
注意第二个参数是 **JSON 字符串**（不是 map），返回值也是 **字符串**。
Note the second argument is a **JSON string** (not a map), and it returns a **string**.

```go
result, err := httpClient.ExecuteRaw("market_state", `{"market":"US"}`)
if err != nil {
    log.Fatal(err)
}
fmt.Println(result)
```

如需从结构体或 map 构造入参，先自行 `json.Marshal` / Marshal your params first if you build them dynamically:

```go
body, _ := json.Marshal(map[string]any{"market": "US"})
result, err := httpClient.ExecuteRaw("market_state", string(body))
```

---

## 错误处理 / Error Handling

SDK 方法返回**强类型结构体**，不需要手动反序列化。
SDK methods return **typed structs** — no manual unmarshalling needed.

```go
import "github.com/tigerfintech/openapi-go-sdk/model"

briefs, err := qc.GetRealTimeQuote(model.BriefRequest{Symbols: []string{"AAPL"}})
if err != nil {
    // API 业务错误或网络错误 / API or network error
    log.Printf("error: %v", err)
    return
}
for _, b := range briefs {
    fmt.Printf("%s latest=%v\n", b.Symbol, b.LatestPrice)
}
```

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
封装好的 API 返回强类型结构体（如 `[]model.Brief`、`[]model.Order`、`*model.PrimeAsset`），直接取字段即可。只有底层的 `httpClient.ExecuteRaw` 返回 JSON 字符串。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请。SDK 根据账号自动识别模拟/实盘账户并路由到对应域名，无需额外配置。
