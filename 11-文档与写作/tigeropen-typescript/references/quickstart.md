# Tiger OpenAPI TypeScript SDK — Quickstart

> TypeScript SDK 快速入门 / Quick Start for TypeScript SDK
> npm: https://www.npmjs.com/package/@tigeropenapi/tigeropen

## 安装 / Installation

```bash
npm install @tigeropenapi/tigeropen
# 或 / or
yarn add @tigeropenapi/tigeropen
# 或 / or
pnpm add @tigeropenapi/tigeropen
```

要求 / Requirements: Node.js 16+，支持 ESM 和 CommonJS

> 包名是 `@tigeropenapi/tigeropen`。历史上也发布过非 scoped 的 `tigeropen`，
> 但该名称已停留在旧版本，请使用 scoped 包名。
> The canonical package is the scoped `@tigeropenapi/tigeropen`; the unscoped
> `tigeropen` name is stale.

---

## 配置 / Configuration

### 方式一：代码直接设置 / Method 1: Direct config

```typescript
import { createClientConfig } from '@tigeropenapi/tigeropen';

const config = createClientConfig({
  tigerId: 'your_tiger_id',
  privateKey: 'your_rsa_private_key',   // PEM 格式私钥字符串
  account: 'your_account',
});
```

### 方式二：从 properties 文件加载 / Method 2: Properties file

```typescript
const configFromFile = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});
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

| 配置项 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| `tigerId` | 开发者 ID | ✅ | - |
| `privateKey` | RSA 私钥 PEM 字符串 | ✅ | - |
| `account` | 交易账户号 | - | - |
| `secretKey` | 机构用户 secret key | - | - |
| `propertiesFilePath` | .properties 文件路径 | - | - |
| `license` | 牌照类型 | - | - |
| `language` | `zh_CN` / `en_US` | - | - |
| `timezone` | 时区字符串 | - | - |
| `timeout` | 请求超时（**秒**） | - | `15` |
| `token` | 直接注入 token | - | - |
| `tokenRefreshDuration` | token 刷新阈值 | - | - |
| `tokenCheckInterval` | token 轮询间隔 | - | - |
| `serverUrl` / `quoteServerUrl` | 自定义服务地址 | - | - |
| `enableDynamicDomain` | 动态域名 | - | - |

---

## 客户端创建 / Create Clients

所有客户端都从**包根路径**导入，不需要子路径 / Import everything from the package root:

```typescript
import {
  createClientConfig,
  HttpClient,
  QuoteClient,
  TradeClient,
  PushClient,
} from '@tigeropenapi/tigeropen';

const cfg = createClientConfig({
  propertiesFilePath: 'tiger_openapi_config.properties',
});

const httpClient = new HttpClient(cfg);

const qc = new QuoteClient(httpClient);              // 行情客户端
const tc = new TradeClient(httpClient, cfg.account); // 交易客户端
const pc = new PushClient(cfg);                      // 推送客户端
```

机构用户可传第三个参数注入 secret key / Institutional users pass a secret key:

```typescript
const tcWithSecret = new TradeClient(httpClient, cfg.account, 'your_secret_key');
```

> `package.json` 的 `exports` 字段只声明了根路径 `.`，因此
> `@tigeropenapi/tigeropen/client/http-client` 这类子路径导入会报
> `ERR_PACKAGE_PATH_NOT_EXPORTED`。但**所有公开 API 都在根路径导出**，
> 直接从包名导入即可，无需退回 `dist/esm/...` 之类的内部路径。
> The package only exports `.`, but every public API is re-exported there —
> import from the package name, never from internal `dist/` paths.

---

## ESM 和 CommonJS / ESM and CommonJS

```typescript
// ESM
import { createClientConfig, QuoteClient } from '@tigeropenapi/tigeropen';

// CommonJS
// const { createClientConfig, QuoteClient } = require('@tigeropenapi/tigeropen');
```

---

## 通用 API 调用 / Generic API Call

当 SDK 未封装某个 API 时，第二个参数是 **JSON 字符串**，返回 **string**：

```typescript
const resp = await httpClient.execute(
  'market_state',
  JSON.stringify({ market: 'US' }),
);
console.log(resp);
```

---

## 错误处理 / Error Handling

```typescript
import { TigerError, classifyErrorCode } from '@tigeropenapi/tigeropen';

try {
  // 注意：getBrief 接收请求对象，不是字符串数组
  const briefs = await qc.getBrief({ symbols: ['AAPL'] });
  console.log(briefs);
} catch (error) {
  if (error instanceof TigerError) {
    console.error('API 错误:', error.message);
  } else if (error instanceof Error) {
    console.error('其他错误:', error.message);
  }
}
```

---

## Token 自动刷新 / Token Auto-refresh

```typescript
const token = await qc.queryToken();
await qc.refreshToken();
```

`TradeClient` 上有同名方法 / The same methods exist on `TradeClient`.

---

## 前置条件 / Prerequisites

1. 老虎证券账户 + 开发者 API 权限：https://developer.itigerup.com/
2. 准备好 `tigerId`、RSA 私钥、`account`
3. 行情数据需要对应的行情权限；期权行情需要期权行情权限

---

## FAQ

**Q: 私钥如何生成?**
参考官方文档 https://docs.itigerup.com/docs/prepare，使用 RSA-2048 生成密钥对，上传公钥到开发者后台。

**Q: 返回值格式?**
封装好的 API 返回**强类型对象**（如 `Brief[]`、`Order[]`、`PrimeAsset | undefined`），
直接取字段即可。只有底层的 `httpClient.execute()` 返回 JSON 字符串。

**Q: 为什么子路径导入报错?**
包只导出根路径。从 `@tigeropenapi/tigeropen` 导入所有 API，不要使用子路径或 `dist/` 内部路径。

**Q: 模拟账户和实盘账户区别?**
模拟账户在开发者后台申请。SDK 根据账号自动识别模拟/实盘账户并路由到对应域名，无需额外配置。
