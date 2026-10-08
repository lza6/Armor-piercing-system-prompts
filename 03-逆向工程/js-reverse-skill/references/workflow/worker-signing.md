# Worker / Service Worker 签名场景

> **触发条件**：目标 JS 在 Worker / Service Worker / Shared Worker 中执行签名逻辑时读。典型特征：主线程 `new Worker(...)` 后通过 `postMessage` 传签名结果，或 Service Worker 拦截 fetch 注入签名头。

## 场景识别

| 信号 | 含义 |
|---|---|
| 主线程代码有 `new Worker(...)` | 签名可能在 Worker 中 |
| 主线程 `worker.postMessage(...)` + `worker.onmessage` 收签名 | Worker 生成签名模式 |
| 注册 Service Worker：`navigator.serviceWorker.register(...)` | SW 可能拦截请求注入签名 |
| `self.addEventListener('fetch', ...)` 出现在某脚本 | SW fetch 拦截确认 |
| SharedArrayBuffer + Atomics | Worker 间共享内存协调签名 |
| `importScripts(...)` 加载签名库 | Worker 通过 importScripts 加载 SDK |

## Worker 签名分析路径

### 1. 定位 Worker 脚本

```
search_code(keyword="new Worker") → 找到 Worker URL
  │
  ├─ URL 是字符串字面量 → 直接下载
  ├─ URL 是动态拼接 → 查找拼接逻辑（如 `new Worker('/static/' + version + '/worker.js')`)
  └─ URL 是 Blob → 检查 `URL.createObjectURL(blob)`，从 blob 构造中提取代码
```

### 2. Worker 上下文定位签名函数

Worker 内部的签名定位与主线程一致（FORENSIC_CAPTURE → TRACE_CAPTURE 黄金路径）：

```
scripts(action='save', url='<worker.js>') → 保存 Worker 脚本
search_code(keyword="<参数名>") → 在 Worker 脚本中搜签名赋值
search_code(keyword="postMessage") → 找到签名结果传回主线程的位置
hook_function(function_path="<签名函数>", mode='trace') → 确认签名生成
```

### 3. Worker 环境补全特殊性

Worker 上下文与主线程不同，补环境时注意：

| 对象 | Worker 中 | 主线程中 | 差异处理 |
|---|---|---|---|
| `self` | 指向 WorkerGlobalScope | 指向 window | 补 `self` = worker context |
| `window` | **不存在** | 存在 | 不补 window，或补 undefined |
| `document` | **不存在**（DedicatedWorker） | 存在 | 不补 document |
| `navigator` | 存在（WorkerNavigator） | 存在（Navigator） | 字段集不同，无 plugins/mimeTypes |
| `location` | 存在（WorkerLocation） | 存在（Location） | 只读，无 hash/search 修改 |
| `importScripts` | 存在 | 不存在 | 补 importScripts 函数 |
| `postMessage` | 发到主线程 | 发到其他 window/worker | 补 postMessage |
| `onmessage` | 收主线程消息 | 收其他来源消息 | 补 onmessage |
| `close` / `terminate` | 关闭自身 | 主线程用 worker.terminate() | 补 close |
| `XMLHttpRequest` / `fetch` | 存在 | 存在 | 一致 |
| `indexedDB` / `caches` | 存在 | 存在 | 一致 |
| `WebSocket` / `EventSource` | 存在 | 存在 | 一致 |
| DOM API | **不存在** | 存在 | 不补 createElement 等 |

### 4. Worker 签名复现方案

| 方案 | 适用 | 实现 |
|---|---|---|
| 直接执行 Worker 脚本 | Worker 脚本自包含，不依赖 DOM | `vm.runInContext(workerCode, workerSandbox)` |
| Worker 模拟 | 需要完整 Worker 行为 | 模拟 WorkerGlobalScope + postMessage + onmessage |
| 提取签名函数 | 算法可从 Worker 脚本中提取 | 直接 require 签名函数，跳过 Worker 机制 |

**Worker 模拟最小实现**：

```javascript
const vm = require('vm');

function createWorkerContext(workerCode, mainThreadCallbacks) {
    const messageQueue = [];
    const workerContext = {
        // Worker 全局
        self: null,
        postMessage: function(data) {
            mainThreadCallbacks.onmessage(data);
        },
        onmessage: null,
        addEventListener: function(type, handler) {
            if (type === 'message') this.onmessage = handler;
        },
        importScripts: function() {
            // 加载并执行额外脚本
            for (const url of arguments) {
                // 按需实现：从本地或网络加载
            }
        },
        close: function() {},

        // Worker 中的 navigator（精简版）
        navigator: {
            userAgent: '<UA>',
            platform: '<platform>',
            language: 'zh-CN',
            hardwareConcurrency: 8,
            // WorkerNavigator 没有 plugins / mimeTypes
        },

        // Worker 中的 location（只读）
        location: new URL('<worker_script_url>'),

        // fetch / XHR（如需）
        fetch: globalThis.fetch,
        XMLHttpRequest: globalThis.XMLHttpRequest,

        // console
        console: console,

        // WASM（如 Worker 加载 WASM）
        WebAssembly: globalThis.WebAssembly,
    };

    workerContext.self = workerContext;
    workerContext.globalThis = workerContext;

    // 执行 Worker 脚本
    vm.createContext(workerContext);
    vm.runInContext(workerCode, workerContext);

    return {
        context: workerContext,
        postMessage: function(data) {
            if (workerContext.onmessage) {
                workerContext.onmessage({ data });
            }
        }
    };
}

// 使用
const worker = createWorkerContext(workerCode, {
    onmessage: (data) => {
        if (data.type === 'sign_result') {
            console.log('签名:', data.signature);
        }
    }
});

// 触发签名
worker.postMessage({ type: 'sign', payload: '<input>' });
```

## Service Worker 签名分析路径

### 1. 定位 SW 脚本

```
search_code(keyword="serviceWorker.register") → 找到 SW 脚本 URL
scripts(action='save', url='<sw.js>') → 保存 SW 脚本
```

### 2. SW fetch 拦截分析

```
search_code(keyword="fetch") → 在 SW 脚本中搜 fetch 监听
search_code(keyword="respondWith") → 确认 fetch 拦截
search_code(keyword="event.request") → 查看请求修改逻辑
search_code(keyword="Headers") → 查找签名头注入
```

典型 SW fetch 拦截代码：

```javascript
self.addEventListener('fetch', (event) => {
    const url = new URL(event.request.url);
    if (url.pathname.startsWith('/api/')) {
        event.respondWith(async () => {
            const sign = await generateSign(url);
            const newRequest = new Request(event.request, {
                headers: { ...event.request.headers, 'X-Sign': sign }
            });
            return fetch(newRequest);
        }());
    }
});
```

### 3. SW 签名复现方案

SW 的签名逻辑通常可以直接提取（不需要模拟 SW 生命周期）：

1. 从 SW 脚本中提取 `generateSign` 函数
2. 按标准 TRACE_CAPTURE → TRACE_ANALYZE → IMPLEMENT 流程还原签名
3. 在 Node.js 中直接调用签名函数

**不需要**模拟完整的 SW 生命周期（install/activate/fetch 事件）。SW 只是签名的执行环境，签名算法本身与执行环境无关。

### 4. SW 上下文特殊性（如需完整模拟）

| 对象 | SW 中 | 主线程 |
|---|---|---|
| `self` | ServiceWorkerGlobalScope | window |
| `clients` | Clients 对象（管理所有页面） | 不存在 |
| `caches` | CacheStorage（SW 专属用法） | 存在但用法不同 |
| `registration` | ServiceWorkerRegistration | 不存在 |
| `window` / `document` | **不存在** | 存在 |
| `fetch` | 存在 | 存在 |
| `importScripts` | 存在 | 不存在 |
| `skipWaiting` / `clients.claim` | SW 生命周期控制 | 不存在 |

## 工具支持

### ruyiPage

| 工具 | Worker 分析用途 |
|---|---|
| `search_code(keyword="new Worker")` | 定位 Worker 构造 |
| `scripts(action='save')` | 保存 Worker/SW 脚本 |
| `hook_function(path="Worker")` | 拦截 Worker 创建，截获脚本 URL |
| `evaluate_js` | 在页面中读取 `navigator.serviceWorker.controller.scriptURL` |
| `network_capture` | 抓 Worker / SW 脚本加载请求 |

### Hook 模板

参考 `references/hooks/hook-templates.md`：
- **postMessage Hook**（模板 11）— 截获 Worker ↔ 主线程消息
- **fetch Hook**（模板 3）— 截获 SW 中的 fetch 拦截
- **Worker Hook** — 拦截 `new Worker()` 构造，记录 Worker URL

## 决策树

```
检测到 Worker / SW 信号
  │
  ├─ 是普通 Worker？
  │   ├─ 是 → 定位 Worker 脚本 → 提取签名函数
  │   │       │
  │   │       ├─ 函数可独立提取 → 直接复用
  │   │       └─ 函数依赖 Worker 环境 → Worker 模拟
  │   │
  │   └─ 否 → 是 Service Worker？
  │       ├─ 是 → 定位 SW 脚本 → 检查 fetch 拦截
  │       │       │
  │       │       ├─ 签名在 fetch 事件中生成 → 提取签名函数直接复用
  │       │       └─ 签名由 SW 内部 WASM 生成 → 参考 env-wasm-advanced.md
  │       │
  │       └─ 是 Shared Worker？
  │           └─ 分析与普通 Worker 类似，注意多端口通信
  │
  └─ 无 Worker 信号 → 按主线程标准流程分析
```

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/hooks/hook-templates.md` | postMessage Hook（模板 11）、fetch Hook（模板 3） |
| `references/env/env-wasm-advanced.md` | Worker 中加载 WASM 的分析路径 |
| `references/env/env-object-model.md` | Worker 中 navigator/location 的对象模型 |
| `references/workflow/phase-flow.md` | FORENSIC_CAPTURE → TRACE_CAPTURE → TRACE_ANALYZE → IMPLEMENT 标准流程（Worker 内同样适用） |
| `references/workflow/decision-tree.md` | 题型判定 |
