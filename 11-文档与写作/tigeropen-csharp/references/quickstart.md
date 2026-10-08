# Tiger OpenAPI C# SDK — Quickstart / 快速入门

> C# SDK 快速入门 / Quick Start for C# SDK
> GitHub: https://github.com/tigerfintech/openapi-cs-sdk

## 安装 / Installation

### NuGet

```bash
dotnet add package TigerBrokers.OpenAPI
```

或在 `.csproj` 中添加：

```xml
<PackageReference Include="TigerBrokers.OpenAPI" Version="*" />
```

---

## 配置 / Configuration

### 配置文件 / Config File

在项目目录创建 `tiger_openapi_config.properties`：

```properties
tiger_id=your_tiger_id
private_key_pk8=MIIEvQIBADANBgkqhkiG9w0BAQEFAASC...
account=your_account_number
license=TBNZ
```

> ⚠️ **配置文件字段名必须是 `private_key_pk8`（PKCS#8 DER base64，无 PEM 头尾行）**，而非 `private_key`。
> The config key must be `private_key_pk8` (PKCS#8 DER base64 without PEM header/footer lines), NOT `private_key`.
>
> 如果持有 PKCS#1 格式（PEM 文件），先转换：
> ```bash
> openssl pkcs8 -topk8 -inform PEM -outform DER -nocrypt -in pk1.pem | base64 | tr -d '\n'
> ```

### 代码加载配置 / Load Config in Code

```csharp
using TigerOpenAPI.Config;
using TigerOpenAPI.Common.Enum;

// 从配置文件目录加载（ConfigFilePath 是目录路径，不是文件全路径）
TigerConfig config = new TigerConfig()
{
    ConfigFilePath = "path/to/config/dir/",
    Language = Language.en_US,
    TimeZone = CustomTimeZone.HK_ZONE
};
```

`TigerConfig` 主要属性：

| 属性 Property | 说明 | 默认值 |
|--------------|------|-------|
| `ConfigFilePath` | 配置文件所在目录路径 | — |
| `TigerId` | Tiger ID（可代替配置文件） | — |
| `PrivateKey` | RSA 私钥（可代替配置文件） | — |
| `DefaultAccount` | 默认账户号（交易请求自动填入） | 从配置文件读取 |
| `Language` | `Language.zh_CN` / `Language.en_US` | `Language.en_US` |
| `TimeZone` | `CustomTimeZone.HK_ZONE` 等 | — |
| `FailRetryCounts` | HTTP 重试次数（0–5，Polly 指数退避） | 2 |
| `AutoGrabPermission` | 自动申请行情权限 | `true` |
| `UseFullTick` | 使用完整逐笔数据 | `false` |
| `IsSslSocket` | 推送连接使用 SSL | `true` |

---

## 创建客户端 / Create Clients

```csharp
using TigerOpenAPI.Quote;
using TigerOpenAPI.Trade;
using TigerOpenAPI.Push;

// 行情客户端 / Quote client
QuoteClient quoteClient = new QuoteClient(config);

// 交易客户端 / Trade client
TradeClient tradeClient = new TradeClient(config);

// 推送客户端（单例，私有构造函数，只能通过 GetInstance 获取）
// Push client — singleton with a private ctor; use GetInstance()
// MyCallback 需自行实现 IApiComposeCallback（27 个成员），详见 push.md
PushClient pushClient = PushClient.GetInstance()
    .Config(config)
    .ApiComposeCallback(new MyCallback());
```

> 三个客户端的构造参数都是 `TigerConfig`，**不是** `HttpClient`。
> `PushClient` 不能 `new`（构造函数是 private），必须用 `PushClient.GetInstance()`。
> All three clients take a `TigerConfig`; `PushClient` must come from `GetInstance()`.

---

## API 调用模式 / API Call Pattern

C# SDK 使用统一的 `TigerRequest<TResponse>` 泛型模式调用所有 API：

```csharp
using TigerOpenAPI.Model;
using TigerOpenAPI.Quote;
using TigerOpenAPI.Quote.Model;
using TigerOpenAPI.Quote.Response;

// 1. 构造请求 / Build request
var request = new TigerRequest<QuoteRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,  // API 名称常量
    ModelValue = new QuoteSymbolModel()
    {
        Symbols = new List<string> { "AAPL", "TSLA" }
    }
};

// 2. 同步执行 / Sync execute
QuoteRealTimeQuoteResponse response = quoteClient.Execute(request);

// 3. 异步执行 / Async execute
QuoteRealTimeQuoteResponse asyncResponse = await quoteClient.ExecuteAsync(request);
```

---

## 错误处理 / Error Handling

**`Execute` / `ExecuteAsync` 不会向调用方抛出异常**：SDK 内部捕获所有异常，
返回一个 `Code = 3`（`TigerApiCode.CLIENT_API_ERROR`）、`Message` 形如
`"sdk send request exception(...)"` 的错误响应。因此应检查 `IsSuccess()` / `Code`，
而不是 `try/catch`。
`Execute`/`ExecuteAsync` swallow exceptions and return an error response
(`Code = 3`), so check `IsSuccess()` instead of catching.

```csharp
var quoteRequest = new TigerRequest<QuoteRealTimeQuoteResponse>()
{
    ApiMethodName = QuoteApiService.QUOTE_REAL_TIME,
    ModelValue = new QuoteSymbolModel()
    {
        Symbols = new List<string> { "AAPL" }
    }
};
var quoteResponse = await quoteClient.ExecuteAsync(quoteRequest);

if (quoteResponse != null && quoteResponse.IsSuccess())
{
    foreach (var item in quoteResponse.Data)
    {
        Console.WriteLine($"{item.Symbol}: {item.LatestPrice}");
    }
}
else
{
    Console.WriteLine($"API Error [{quoteResponse?.Code}]: {quoteResponse?.Message}");
}
```

SDK 中唯一的异常类型是 `TigerOpenAPI.Common.TigerApiException`
（属性 `ErrCode`、`ErrMsg`、`TigerApiCode`）。它主要由 `Validate()` 前置校验抛出，
例如缺少 `Account`。**没有** `ApiException` 类，也没有 `TigerOpenAPI.Common.Exceptions` 命名空间。
The only exception type is `TigerOpenAPI.Common.TigerApiException`; there is no
`ApiException` and no `Common.Exceptions` namespace.

```csharp
try
{
    var badRequest = new TigerRequest<PositionsResponse>()
    {
        ApiMethodName = TradeApiService.POSITIONS,
        ModelValue = new PositionsModel()
    };
    var r = await tradeClient.ExecuteAsync(badRequest);
}
catch (TigerApiException ex)
{
    Console.WriteLine($"参数校验失败 [{ex.ErrCode}]: {ex.ErrMsg}");
}
```

---

## 直接调用 API / Raw API Call

未封装的接口用通用响应类型：`TigerDictResponse`（`Data` 为
`Dictionary<string, object>`）、`TigerListResponse`、`TigerStringResponse`、
`TigerListStringResponse`。**没有** `RawResponse` 类型。
Use the generic response types; there is no `RawResponse`.

```csharp
var rawRequest = new TigerRequest<TigerDictResponse>()
{
    ApiMethodName = QuoteApiService.MARKET_STATE,
    ModelValue = new QuoteMarketModel() { Market = Market.US }
};
var rawResponse = quoteClient.Execute(rawRequest);
```

---

## 模拟账户 vs 实盘 / Paper vs Live

`TradeClient` 根据账户号自动判断模拟/实盘并路由到对应地址
（`AccountUtil.IsVirtualAccount`）。

```csharp
// 查询账户列表以确认账户类型 / Query accounts to verify type
// 注意：ACCOUNTS 接口用基类 ApiModel，没有 AccountModel 类型
var accountsRequest = new TigerRequest<AccountsResponse>()
{
    ApiMethodName = TradeApiService.ACCOUNTS,
    ModelValue = new ApiModel()
};
var accounts = await tradeClient.ExecuteAsync(accountsRequest);
```

---

## 自动注入 / Automatic Injection

`TradeClient.Validate()` 在每次请求前自动补全两个字段（已显式赋值的不会被覆盖）：

1. **`Account`** — 除 `TradeApiService.ACCOUNTS` 外，所有交易接口在 `ModelValue.Account`
   为空时自动填入 `TigerConfig.DefaultAccount`；仍为空则抛 `TigerApiException`
2. **`SecretKey`** — 当 `ModelValue` 是 `TradeModel` 子类且其 `SecretKey` 为空、
   且 `TigerConfig.SecretKey` 非空时自动填入（机构用户）

因此示例中通常无需显式传 `Account`。
`Validate()` auto-fills `Account` (from `DefaultAccount`) and `SecretKey` for
`TradeModel` subclasses; explicit values are never overwritten.

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tiger_id`、RSA 私钥（2048 位）、账户号
3. 行情数据需要对应市场的行情权限
4. .NET 10.0 (net10.0) / C# 14 (SDK default)
