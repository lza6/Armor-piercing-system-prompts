# WebSocket / SSE 消息签名

> **触发条件**：目标接口用 WebSocket 或 Server-Sent Events 通信，签名在消息帧里时读。直播/行情/聊天/即时通讯类站点常见。

## 场景识别

| 信号 | 含义 |
|---|---|
| `new WebSocket(url)` / `ws://` / `wss://` | WebSocket 通信 |
| `new EventSource(url)` | SSE 通信 |
| `socket.send(...)` / `socket.onmessage` | WebSocket 消息收发 |
| 消息体含 sign/token/timestamp 字段 | 消息签名 |
| `protobuf` / `msgpack` 编码的消息体 | 二进制消息（见 protocol-analysis.md 第 8 节） |
| 心跳消息（定时 send 固定结构） | 保活签名 |

## WebSocket 签名分析路径

### 1. 定位 WebSocket 连接

```
search_code(keyword="new WebSocket") → 找到 WS URL
search_code(keyword="wss://") → 找到 WS 地址
  │
  ├─ URL 是固定字符串 → 直接分析
  ├─ URL 含动态参数（token/sign） → 查找 URL 构造逻辑
  └─ URL 从接口返回 → 先请求该接口获取 WS URL
```

### 2. 定位消息签名逻辑

```
search_code(keyword="socket.send") → 找到发送消息的位置
search_code(keyword="JSON.stringify") → 查看消息体构造
hook_function(path="WebSocket.send", mode='trace') → 截获所有发送消息
  │
  ├─ 消息是 JSON 字符串 → 解析后找 sign/token 字段
  ├─ 消息是二进制（ArrayBuffer/Blob）→ 可能是 protobuf，见 protocol-analysis.md 第 8 节
  └─ 消息含时间戳 → 确认签名是否含时效性
```

### 3. 签名复现

WebSocket 签名复现与 HTTP 签名一致，区别只在传输层：

| 维度 | HTTP 签名 | WebSocket 签名 |
|---|---|---|
| 签名位置 | URL query / Header / Body | 消息帧 JSON 字段 / 二进制帧 |
| 签名输入 | URL + 参数 + 时间戳 | 消息内容 + 时间戳 + 会话 ID |
| 传输方式 | 请求-响应 | 全双工持久连接 |
| 验证方式 | ≥5 次请求 | ≥5 次消息发送 |

```javascript
// WebSocket 签名复现示例
const WebSocket = require('ws');

function createSignedMessage(payload, secret) {
    const ts = Date.now();
    const sign = md5(JSON.stringify(payload) + secret + ts);
    return JSON.stringify({ ...payload, ts, sign });
}

const ws = new WebSocket('wss://example.com/ws');
ws.on('open', () => {
    // 发送签名消息
    ws.send(createSignedMessage({ action: 'subscribe', channel: 'live' }, SECRET));
});
ws.on('message', (data) => {
    console.log('收到:', data.toString());
});
```

### 4. 心跳保活签名

部分站点的 WebSocket 心跳消息也需要签名：

```javascript
// 心跳签名
function createHeartbeat(secret) {
    const ts = Date.now();
    return JSON.stringify({ type: 'ping', ts, sign: md5('ping' + ts + secret) });
}

// 定时发送
setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
        ws.send(createHeartbeat(secret));
    }
}, 30000);
```

## SSE 签名分析路径

SSE 是单向通信（服务端→客户端），签名通常在连接 URL 中：

```
search_code(keyword="EventSource") → 找到 SSE URL
  │
  ├─ URL 含 sign/token → 与 HTTP 签名分析一致
  ├─ URL 需要先请求接口获取 → 先分析那个接口
  └─ 连接后服务端推送加密数据 → 见场景 3（响应数据加密）
```

## 工具支持

### ruyiPage

| 工具 | WebSocket 分析用途 |
|---|---|
| `search_code(keyword="WebSocket")` | 定位 WS 构造 |
| `hook_function(path="WebSocket.send")` | 截获发送消息 |
| `hook_function(path="WebSocket.onmessage")` | 截获接收消息 |
| `network_capture` | 抓 WS 握手请求（Upgrade 头） |
| `evaluate_js` | 在页面中读取 WS 实例状态 |

### Node.js 库

| 库 | 用途 |
|---|---|
| `ws` | WebSocket 客户端 |
| `eventsource` | SSE 客户端 |
| `protobufjs` | protobuf 消息编解码 |

## 决策树

```
检测到 WebSocket / SSE 信号
  │
  ├─ 签名在连接 URL 中？
  │   ├─ 是 → 按标准 HTTP 签名分析（FORENSIC_CAPTURE → TRACE_CAPTURE → TRACE_ANALYZE → IMPLEMENT）
  │   └─ 否 → 签名在消息帧中
  │       │
  │       ├─ 消息是 JSON → 提取 sign 字段生成逻辑
  │       ├─ 消息是二进制 → 可能是 protobuf（见 protocol-analysis.md 第 8 节）
  │       └─ 心跳需要签名 → 单独分析心跳签名格式
  │
  └─ 需要保持长连接？
      ├─ 是 → 用 ws 库建立持久连接
      └─ 否 → 每次新建连接（注意握手开销）
```

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/hooks/hook-templates.md` | WebSocket Hook 模板 |
| `references/network/protocol-analysis.md` | 第 8 节 Protobuf/MsgPack 二进制消息 |
| `references/crypto/crypto-entry.md` | 四层链路（WS 签名同样适用） |
| `references/workflow/worker-signing.md` | Worker 中可能用 WebSocket |
