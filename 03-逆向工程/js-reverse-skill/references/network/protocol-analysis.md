# 协议层分析与对抗

本文件在算法全部还原正确但请求仍然失败时读取。目标是排查协议层检测（TLS 指纹、HTTP/2、UA、Referer、CORS、频率限制、IP 封禁），不要把协议层问题误判为 JS 补环境问题。

## 诊断流程

```
请求失败？
    │
    ├─ 检查 HTTP 状态码
    │   ├─ 403 → TLS 指纹 / Referer / UA / IP 封禁
    │   ├─ 412 → 前置条件失败（Cookie / Token 过期）
    │   ├─ 429 → 频率限制
    │   └─ 200 但数据异常 → 参数错误 / 加密不对
    │
    ├─ 同样的请求在浏览器中成功？
    │   ├─ 是 → TLS 指纹问题，切换到 TLS 指纹兼容客户端
    │   └─ 否 → 参数/Cookie 确实有问题，重新分析
    │
    ├─ curl 命令测试
    │   ├─ curl 成功 → Node.js 请求头/参数构造问题
    │   └─ curl 也失败 → TLS / 协议 / IP 问题
    │
    └─ 使用 HTTP/2？
        ├─ HTTP/2 成功 → 切换到 http2 模块
        └─ HTTP/2 也失败 → TLS 指纹问题
```
## 字段归属分类法

在排查协议层问题之前，先对请求中每个参数做**字段本质属性分类**，避免把不同类字段用同一思路处理导致误判。

### 6 类字段

| 类型 | 特征 | 定位思路 | 复现方式 |
|---|---|---|---|
| 固定字段 | 多次请求值不变 | 直接从抓包复制 | 写死常量 |
| 明文字段 | 从页面/接口响应中直接可见 | 搜 Network Response / DOM 内容 | 从响应提取 |
| 本地算法字段 | 客户端计算得出（md5/aes/rsa/自编） | 纯算还原 / vm 沙箱 | 复现算法 |
| 服务端 token | 服务端下发，客户端仅携带 | 分析前置接口 + Session 链 | 请求前置接口 |
| 风控指纹 | 环境/设备指纹，不参与签名但影响校验 | 补环境 | 指纹基线对齐 |
| 验证码交互 | 人机验证产物（滑块/点选/图片） | 参考 web-verify-patcher | 打码平台 |

### 分类流程

```
拿到请求参数
  │
  ├─ 多次请求值不变？ → 固定字段
  ├─ 在页面/接口响应中搜到？ → 明文字段
  ├─ 在 JS 中搜到加密/签名逻辑？ → 本地算法字段
  ├─ 在前置接口 Set-Cookie 或响应体中？ → 服务端 token
  ├─ 涉及 navigator/canvas/webgl/screen 等环境 API？ → 风控指纹
  └─ 需要人机交互？ → 验证码交互
```

分类后再走四层链路（source→entry→builder→writer）做函数级精准定位。

## 1. TLS 指纹检测

### 原理

服务器通过分析 TLS Client Hello 报文中的特征来识别客户端类型：
- Cipher Suites 列表和顺序
- TLS 扩展列表和顺序
- 支持的曲线（Elliptic Curves）
- 签名算法
- ALPN 协议列表

不同客户端（Chrome、Firefox、curl、Node.js）的 TLS 指纹不同。

### 检测表现

- 请求返回 `403 Forbidden`
- 返回空响应或错误页面
- 返回 `token failed`、`access denied` 等自定义错误
- 同样的参数在浏览器中正常，在 Node.js 中失败

### 解决方案

TLS 指纹问题的完整解决方案见 `network/tls-validation.md`。核心原则：

- 前置阶段自动探测并选用 TLS 指纹兼容客户端（Node curl-cffi-node → impers → Python curl_cffi / cffi_curl），不要等失败后才切换，不要求用户选择。
- 最终请求必须使用 Session 模式，复用同一 Cookie jar、UA、TLS 指纹。
- 取证 baseline 是 Firefox 时，curl_cffi 必须按 JA3 / JA4 / Akamai 指纹对齐，不能只改 UA。

### ruyiPage 辅助验证

```
ruyiPage: launch_browser(os_type="macos")
ruyiPage: get_fingerprint_info
→ 验证当前浏览器环境中的 UA、平台和指纹是否与目标站点要求一致
```

注意：ruyiPage 只用于取证和参数生成验证，最终请求不得使用浏览器自动化发送，必须使用 TLS 指纹兼容客户端。

## 2. HTTP/2 协议

### 检测表现

- HTTP/1.1 请求返回错误
- HTTP/2 请求正常返回数据
- 抓包可以看到服务器仅接受 h2 连接

### Node.js HTTP/2 请求

```javascript
const http2 = require('http2');

function http2Request(url, options = {}) {
    return new Promise((resolve, reject) => {
        const urlObj = new URL(url);
        const client = http2.connect(`${urlObj.protocol}//${urlObj.host}`);

        client.on('error', reject);

        const headers = {
            ':method': options.method || 'GET',
            ':path': urlObj.pathname + urlObj.search,
            ':scheme': urlObj.protocol.replace(':', ''),
            ':authority': urlObj.host,
            'user-agent': options.userAgent || 'Mozilla/5.0 ...',
            'accept': 'application/json',
            ...options.headers,
        };

        if (options.cookie) {
            headers['cookie'] = options.cookie;
        }

        const req = client.request(headers);

        let data = '';
        req.on('data', (chunk) => { data += chunk; });
        req.on('end', () => {
            client.close();
            resolve(data);
        });
        req.on('error', reject);

        if (options.body) {
            req.write(options.body);
        }
        req.end();
    });
}
```

注意：Node.js 原生 http2 模块的 TLS 指纹仍与浏览器不同，高强度检测场景仍需使用 curl-cffi / impers 等 TLS 指纹兼容客户端。

## 3. User-Agent 校验

### 常见场景

- 某些页面要求特定 UA
- 不同 UA 返回不同内容
- 空 UA 或非浏览器 UA 被拒绝

### 规则

- 最终请求的 UA 必须与补环境时的 `navigator.userAgent` 一致。
- UA 必须来自取证 baseline，不要使用默认或随机 UA。
- Client Hints（`sec-ch-ua`、`sec-ch-ua-platform`、`sec-ch-ua-mobile`）必须与 `navigator.userAgentData` 一致；Firefox baseline 不应伪造 Chrome Client Hints。

## 4. Referer 校验

### 常见场景

- API 接口检查 Referer 头
- 必须来自特定页面的请求
- 缺少 Referer 返回 403

### 解决方案

```javascript
headers: {
    'Referer': 'https://target.com/page',
    'Origin': 'https://target.com',
}
```

Referer 和 Origin 必须来自 HAR / 浏览器请求链，不要凭空构造。

## 5. CORS 与跨域

### 预检请求 (Preflight)

```javascript
// 某些请求会先发 OPTIONS 预检
// 确保模拟请求时也处理 CORS 相关头
headers: {
    'Origin': 'https://target.com',
    'Access-Control-Request-Method': 'POST',
    'Access-Control-Request-Headers': 'content-type',
}
```

最终请求验证时，CORS 预检请求也应在同一 Session 中完成。

## 6. 请求频率限制

### 常见策略

- IP 维度限流
- Cookie/Session 维度限流
- 请求间隔检测
- 滑动窗口计数

### 对抗方式

```javascript
function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

async function fetchWithRateLimit(urls, delayMs = 1000) {
    const results = [];
    for (const url of urls) {
        const data = await fetch(url);
        results.push(data);
        await sleep(delayMs + Math.random() * 500); // 随机化延迟
    }
    return results;
}
```

注意：本 Skill 只做低频、最小化请求，优先复现用户已提供的成功 cURL / HAR。不要做批量请求或压力测试。

## 7. IP 封禁对抗

### 代理轮换

```javascript
const HttpsProxyAgent = require('https-proxy-agent');

const proxies = [
    'http://proxy1:port',
    'http://proxy2:port',
];

function getRandomProxy() {
    return proxies[Math.floor(Math.random() * proxies.length)];
}

async function fetchWithProxy(url, options = {}) {
    const agent = new HttpsProxyAgent(getRandomProxy());
    return fetch(url, { ...options, agent });
}
```

代理 / IP / 地区必须与 fingerprint baseline 一致，不要随意切换代理导致 baseline 冲突。

## 8. 二进制序列化请求体（Protobuf / MsgPack）

### 识别特征

部分现代 API 不用 JSON，而用二进制序列化格式编码请求体。识别信号：

| 信号 | Protobuf | MsgPack |
|---|---|---|
| Content-Type | `application/x-protobuf` / `application/grpc` / `application/grpc-web` | `application/x-msgpack` / `application/msgpack` |
| 请求体特征 | 二进制，首字节是 field tag（如 `0x0a` / `0x12`） | 二进制，类似 JSON 但更紧凑 |
| 响应体特征 | 同上 | 同上 |
| JS 中构造方式 | `protobufjs` 库 / `grpc-web` / 手写 encode | `msgpack-lite` / `@msgpack/msgpack` |
| 抓包可见 | 不可读乱码 | 部分可读（字符串字面量） |

### 分析路径

```
检测到二进制请求体
  │
  ├─ 检查 Content-Type
  │   ├─ protobuf/grpc → 在 JS 中搜 protobufjs / grpc-web / .proto
  │   └─ msgpack → 在 JS 中搜 msgpack-lite / @msgpack/msgpack
  │
  ├─ 搜索序列化库
  │   search_code(keyword="protobuf") / search_code(keyword="msgpack")
  │   search_code(keyword="encode") / search_code(keyword="decode")
  │
  ├─ 定位序列化入口
  │   hook_function(path="<encode 函数>", mode='trace')
  │   → 查看输入参数（原始对象）和输出（二进制）
  │
  └─ 复现方案
      ├─ 提取 .proto 定义（如有） → 用 protobufjs 直接 encode
      ├─ 无 .proto → 从 encode 函数反推 schema
      └─ MsgPack → 直接用 msgpack-lite 库 encode 同结构对象
```

### 复现方案

**Protobuf（有 .proto 文件）**：

```javascript
const protobuf = require('protobufjs');
const root = protobuf.loadSync('protocol.proto');
const Request = root.lookupType('package.Request');
const buffer = Request.encode({ field1: 'value', field2: 123 }).finish();
// buffer 是 Uint8Array，作为请求 body 发送
```

**Protobuf（无 .proto，反射 schema）**：

```javascript
// 用 hook 截获 encode 输入和输出
// 从多次截获中推断字段编号和类型
// 构建最小 .proto 或直接用 protobufjs 的 Reader/Writer 手写 encode
```

**MsgPack**：

```javascript
const msgpack = require('@msgpack/msgpack');
const encoded = msgpack.encode({ field1: 'value', field2: 123 });
// encoded 是 Uint8Array
```

### 注意事项

- 二进制序列化的签名参数通常在序列化**之前**的对象中计算，然后一起 encode
- 定位签名函数时，hook encode 入口能看到完整的明文对象（含签名字段）
- gRPC-web 的请求体可能有 frame prefix（5 字节：1 字节压缩标志 + 4 字节长度）
- 响应也可能是二进制序列化，需要对应 decode

## 与其他文件的关系

- TLS 指纹兼容客户端选择、Session 模式、Firefox baseline 对齐：见 `network/tls-validation.md`
- 最终请求链路、Cookie / challenge 生成链路、final 入口顺序：见 `network/session-chain.md`
- 动态 HTML / JS 资源刷新：见 `network/dynamic-resource.md`
- 指纹基线一致性：见 `fingerprint/fingerprint-baseline-consistency.md`

## 输出要求

阶段报告记录：

```markdown
## 协议层分析

- 是否存在协议层检测：是 / 否
- 检测类型：TLS 指纹 / HTTP/2 / UA / Referer / CORS / 频率限制 / IP 封禁
- 诊断依据：
- 解决方案：
- 是否影响最终请求：是 / 否
```
