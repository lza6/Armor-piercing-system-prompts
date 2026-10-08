# WASM 逆向进阶

> **触发条件**：基础 `env-wasm.md` 的 I/O 验证不足以解决当前 WASM 案例时读。典型场景：import 依赖复杂、Emscripten/wasm-bindgen 环境补全量大、memory 管理出问题、streaming 实例化、Worker 中加载 WASM、unreachable panic 无法定位。

本文档是 `env-wasm.md` 的进阶补充，不重复基础加载内容。

## import 依赖分析决策树

```
WebAssembly.Module.imports(module) 返回导入列表
  │
  ├─ 模块名是 env？
  │   ├─ 是 → 标准 WASM 环境依赖（memory/table/abort/...）
  │   │        按"常见 env import 函数表"逐项补全
  │   └─ 否 → 检查是否为编译器特定模块名
  │       ├─ wasi_snapshot_preview1 / wasi_unstable → WASI 接口（见下文）
  │       ├─ wbg → wasm-bindgen 生成（见 wasm-bindgen 段）
  │       ├─ env + __syscall_* → Emscripten syscall（见 Emscripten 段）
  │       └─ 其他 → 按目标 JS 中对应实现补全
  │
  ├─ kind 是 memory / table？
  │   └─ 必须由宿主提供共享 Memory/Table 实例
  │
  └─ kind 是 function？
      └─ 按函数名查找目标 JS 中的实现，复制到 importObject
```

### 常见 env import 函数表

| 函数名 | 用途 | 默认实现 |
|---|---|---|
| `memory` | 共享线性内存 | `new WebAssembly.Memory({ initial: 256 })` |
| `table` | 函数表 | `new WebAssembly.Table({ initial: 0, element: 'anyfunc' })` |
| `abort(msg, file, line, col)` | abort 调用 | 抛 Error 含位置信息 |
| `__js_*` / `__jsbridge_*` | JS 桥接函数 | 按目标 JS 实现 |
| `console_log(ptr, len)` | 日志输出 | 读 memory 写入字符串后 console.log |
| `Math_*` | 数学函数 | 对应 Math 方法 |
| `Date_now()` | 时间戳 | `Date.now()` |
| `performance_now()` | 高精度时间 | `performance.now()` |
| `random_*` | 随机数 | fixtures 控制或 `Math.random()` |

### WASI 接口

WASI（WebAssembly System Interface）模块名通常是 `wasi_snapshot_preview1` 或 `wasi_unstable`。常见函数：

| 函数 | 用途 | 最小实现 |
|---|---|---|
| `fd_write(fd, iovs, iovs_len, nwritten)` | 写入 | 返回 0，写入字节数写回 memory |
| `fd_close(fd)` | 关闭 fd | 返回 0 |
| `fd_seek(fd, offset, whence, newoffset)` | seek | 返回 0 |
| `proc_exit(code)` | 进程退出 | 抛 Error 或 no-op |
| `environ_get` / `environ_sizes_get` | 环境变量 | 返回 0 |
| `args_get` / `args_sizes_get` | 命令行参数 | 返回 0 |
| `clock_time_get` / `clock_res_get` | 时钟 | 写入 `Date.now()` |
| `random_get(buf, len)` | 随机字节 | crypto.randomFillSync |

**推荐**：如果 WASM 大量使用 WASI，直接用 Node.js 的 `wasi` 模块（Node 22+ 稳定）：

```javascript
const { WASI } = require('wasi');
const wasi = new WASI({ version: 'preview1', args: [], env: {}, preopens: {} });
const importObject = { ...wasi.wasiImport };
```

## Emscripten 环境补全完整模板

Emscripten 编译的 WASM 通常需要 `Module` 对象和大量运行时函数。完整模板：

```javascript
const Module = {
    // 基础配置
    print: (text) => console.log('stdout:', text),
    printErr: (text) => console.error('stderr:', text),
    noInitialRun: true,
    noExitRuntime: true,

    // 内存配置
    INITIAL_MEMORY: 16777216,    // 16MB
    MAXIMUM_MEMORY: 268435456,   // 256MB
    ALLOW_MEMORY_GROWTH: true,

    // 运行时回调
    onRuntimeInitialized: function() {
        console.log('WASM Runtime Ready');
    },
    onAbort: function(e) {
        console.error('WASM Aborted:', e);
    },

    // 环境变量
    ENV: {},
    PATH: '/',

    // 文件系统（如果 WASM 用到 FS）
    // 轻量方案：stub 掉 FS 调用
    // 完整方案：引入 emscripten 的 MEMFS
};

// Emscripten 常见 import 函数
const emscriptenImports = {
    env: {
        // 内存管理
        emscripten_resize_heap: function(requestedSize) {
            // 让内存增长
            return 1;
        },
        emscripten_memcpy_big: function(dest, src, num) {
            const heap = new Uint8Array(Module.HEAP8.buffer);
            heap.copyWithin(dest, src, src + num);
        },
        emscripten_get_heap_size: function() {
            return Module.HEAP8.buffer.byteLength;
        },

        // 时间
        emscripten_date_now: function() { return Date.now(); },
        emscripten_performance_now: function() { return performance.now(); },
        emscripten_get_now: function() { return Date.now(); },

        // 日志
        emscripten_log: function(priority, format, varArgs) {
            // 简化：忽略格式化，直接读取字符串
        },

        // 退出
        emscripten_exit_with_live_runtime: function() {},
        emscripten_force_exit: function(status) { throw new Error('force_exit: ' + status); },
        exit: function(status) { if (status !== 0) throw new Error('exit: ' + status); },

        // syscall stubs
        __syscall_fcntl64: function(fd, cmd, varargs) { return 0; },
        __syscall_ioctl: function(fd, op, varargs) { return 0; },
        __syscall_openat: function(dirfd, path, flags, varargs) { return -1; },
    }
};
```

### ASYNCIFY 支持

如果 Emscripten 启用了 ASYNCIFY（用于异步操作），需要额外支持：

```javascript
const Module = {
    // ...其他配置

    ASYNCIFY: {
        // ASYNCIFY 需要主动管理栈
        // 由 WASM 内部通过 imports.wasi_snapshot_preview1.asyncio_* 调用
    }
};

// ASYNCIFY 相关 import
const asyncifyImports = {
    wasi_snapshot_preview1: {
        // ...其他 WASI 函数
    },
    env: {
        // ASYNCIFY 导入的函数由 emscripten 自动生成
        // 通常不需要手动补
    }
};
```

## wasm-bindgen 环境补全完整模板

wasm-bindgen（Rust）生成的 WASM 需要 `wbg` 模块。完整模板：

```javascript
function buildWbgImports(memory) {
    const decoder = new TextDecoder('utf-8');
    const encoder = new TextEncoder();

    function getString(ptr, len) {
        return decoder.decode(new Uint8Array(memory.buffer, ptr, len));
    }

    function appendString(str) {
        const bytes = encoder.encode(str);
        const ptr = wasm.__wbindgen_export_0(bytes.length, 1);
        new Uint8Array(memory.buffer, ptr, bytes.length).set(bytes);
        return [ptr, bytes.length];
    }

    return {
        wbg: {
            // 字符串传递
            __wbindgen_string_new: function(ptr, len) {
                return getString(ptr, len);
            },
            __wbindgen_string_get: function(idx, ptrptr, lenptr) {
                const str = getObject(idx);
                const [ptr, len] = appendString(str);
                new Uint32Array(memory.buffer, ptrptr, 1)[0] = ptr;
                new Uint32Array(memory.buffer, lenptr, 1)[0] = len;
            },

            // 对象管理
            __wbindgen_object_drop_ref: function(idx) {
                dropObject(idx);
            },
            __wbindgen_object_clone_ref: function(idx) {
                return addObject(getObject(idx));
            },

            // 异常
            __wbindgen_throw: function(ptr, len) {
                throw new Error(getString(ptr, len));
            },

            // console
            __wbindgen_log: function(ptr, len) {
                console.log(getString(ptr, len));
            },

            // 性能
            __wbindgen_now: function() {
                return performance.now();
            },

            // 随机数
            __wbindgen_math_random: function() {
                return Math.random();
            },

            // 其他按目标 JS 的 wbg 适配文件补全
            // 通常在 .wasm 同目录有 *_bg.js 文件参考
        }
    };
}
```

**提示**：wasm-bindgen 生成的 JS 胶水文件（通常叫 `*_bg.js`）是最佳参考。直接读该文件，按其中的 `wbg` 对象逐项实现。

## WASM memory 管理

### malloc/free 分配器

WASM 通常导出 `_malloc` / `_free`（Emscripten）或 `__wbindgen_export_0`（wasm-bindgen）。策略：

| 场景 | 策略 |
|---|---|
| WASM 导出了 malloc/free | 用导出函数分配/释放，不要手动算偏移 |
| WASM 没有导出 malloc | 用静态偏移区（如 `__heap_base` 之后）手动管理 |
| 大量小对象分配 | 避免频繁 malloc/free，用 buffer pool |

### 内存增长

```javascript
// 允许内存增长
const memory = new WebAssembly.Memory({
    initial: 256,      // 初始 256 页 = 16MB
    maximum: 16384,    // 最大 16384 页 = 1GB
});

// 监听内存增长（会触发 ArrayBuffer detached）
memory.grow = new Proxy(memory.grow, {
    apply(target, thisArg, args) {
        const oldBuffer = memory.buffer;
        const result = Reflect.apply(target, thisArg, args);
        // 内存增长后，所有引用旧 buffer 的 Uint8Array 等都失效
        // 需要重新创建视图
        return result;
    }
});

// 实战：不要缓存 Uint8Array(memory.buffer, ...)
// 每次访问都重新创建视图：
function readMemory(ptr, len) {
    return new Uint8Array(memory.buffer, ptr, len);
}
```

### 字符串读写边界

```javascript
// 读字符串（UTF-8，以 null 结尾）
function readCString(ptr) {
    const view = new Uint8Array(memory.buffer);
    let end = ptr;
    while (view[end] !== 0) end++;
    return new TextDecoder('utf-8').decode(view.slice(ptr, end));
}

// 读字符串（UTF-8，指定长度）
function readString(ptr, len) {
    return new TextDecoder('utf-8').decode(new Uint8Array(memory.buffer, ptr, len));
}

// 写字符串（UTF-8）
function writeString(str, ptr) {
    const bytes = new TextEncoder().encode(str);
    new Uint8Array(memory.buffer, ptr, bytes.length).set(bytes);
    // 如果需要 null terminator，额外写一个 0
}

// 读 UTF-16 字符串（Windows 风格）
function readWideString(ptr) {
    const view = new Uint16Array(memory.buffer);
    let end = ptr / 2;
    while (view[end] !== 0) end++;
    return String.fromCharCode.apply(null, view.slice(ptr / 2, end));
}
```

**边界注意**：
- `ptr` 必须对齐（4 字节对齐常见）
- `len` 单位是字节，不是字符数
- 内存增长后旧 `Uint8Array` 视图失效，必须重新创建

## streaming 实例化

`WebAssembly.instantiateStreaming` 与 buffer 实例化的差异：

| 维度 | `instantiateStreaming` | `instantiate(Buffer)` |
|---|---|---|
| 输入 | Response 对象（流式） | ArrayBuffer/TypedArray |
| 性能 | 更优（边下载边编译） | 需要完整下载后编译 |
| Node.js 支持 | 不直接支持（需要 polyfill） | 完全支持 |
| Content-Type | 服务器必须返回 `application/wasm` | 不涉及 |

**Node.js 中的 streaming 替代**：

```javascript
const fs = require('fs');
const { pipeline } = require('stream');

async function loadWasmStreaming(wasmPath, importObject) {
    // Node.js 没有原生 WebAssembly.instantiateStreaming
    // 但可以用 compile + instantiate 分离
    const stream = fs.createReadStream(wasmPath);
    const chunks = [];

    for await (const chunk of stream) {
        chunks.push(chunk);
    }

    const buffer = Buffer.concat(chunks);
    const module = await WebAssembly.compile(buffer);
    return await WebAssembly.instantiate(module, importObject);
}

// 或直接用 compileStreaming polyfill
if (!WebAssembly.compileStreaming) {
    WebAssembly.compileStreaming = async function(response) {
        const buffer = await response.arrayBuffer();
        return WebAssembly.compile(buffer);
    };
}
```

**实战建议**：Node.js 环境直接用 `fs.readFileSync` + `WebAssembly.instantiate`，不需要 streaming。

## Worker 中的 WASM 加载链路

### 普通 Worker

```javascript
// Worker 上下文补全
const workerContext = {
    self: null,           // 指向自身
    postMessage: null,    // 发消息到主线程
    onmessage: null,      // 接收主线程消息
    importScripts: null,  // 加载其他脚本
    close: null,          // 关闭 Worker

    // Worker 没有 window / document
    // 但有 navigator / location / indexedDB / caches 等

    // WASM 在 Worker 中加载与主线程一致
    // 但 importObject 可能需要通过 postMessage 接收
};

// 主线程侧
const worker = new Worker('./worker.js');
worker.postMessage({ type: 'init', wasmUrl: '...', importConfig: {...} });
worker.onmessage = (e) => {
    if (e.data.type === 'sign_result') {
        console.log('签名结果:', e.data.signature);
    }
};
```

### Service Worker fetch 拦截

Service Worker 可以拦截 fetch 请求注入签名：

```javascript
// Service Worker 上下文
self.addEventListener('fetch', (event) => {
    const url = new URL(event.request.url);

    // 匹配需要签名的接口
    if (url.pathname.startsWith('/api/')) {
        event.respondWith(async function() {
            // 在 SW 中调用 WASM 生成签名
            const signature = await wasmSign(url);

            // 克隆请求并添加签名头
            const signedRequest = new Request(event.request, {
                headers: {
                    ...event.request.headers,
                    'X-Signature': signature,
                }
            });

            return fetch(signedRequest);
        }());
    }
});

// SW 特殊性：
// 1. 生命周期：install → activate → fetch/message 事件
// 2. 不能直接访问 window，但有 self / clients / caches
// 3. WASM 在 SW 中加载用 importScripts 或 fetch + WebAssembly.instantiate
```

### 分析策略

| 场景 | 分析方法 |
|---|---|
| Worker 加载 WASM 并生成签名 | hook `Worker` 构造函数，截获 Worker 脚本 URL；在 Worker 脚本中定位 WASM 加载和签名导出 |
| Worker 通过 postMessage 传签名回主线程 | hook `postMessage`，记录消息内容 |
| Service Worker 拦截 fetch 注入签名 | 在 SW 脚本中搜索 `fetch` / `respondWith` / `event.request` |
| SharedArrayBuffer + Atomics 协调 | 高级场景，需 hook Atomics.wait/notify |

详见 `references/workflow/worker-signing.md`。

## 高级调试技巧

### unreachable panic 栈追踪

`unreachable` 指令触发时，WASM 抛出 `RuntimeError: unreachable`。栈通常不包含源码位置。

排查步骤：

1. **确认环境完整性**：`WebAssembly.Module.imports()` 所有依赖是否都已提供
2. **检查内存对齐**：传入的指针是否对齐（4 字节 / 8 字节）
3. **检查参数类型**：JS 传给 WASM 的参数类型是否正确（number vs BigInt）
4. **检查内存越界**：写入/读取的偏移是否超出 memory 范围
5. **启用 debug 构建**：如果有源码，用 `-g` 编译 WASM 保留调试信息
6. **用 wasm-objdump 分析**：`wasm-objdump -d module.wasm` 查看反汇编
7. **用 wasm-decompile**：`wasm-decompile module.wasm -o out.dcall` 生成伪代码

### 输出为空排查决策树

```
WASM 调用后返回空值或空字符串
  │
  ├─ 检查返回值类型
  │   ├─ 返回指针 → 读 memory 看是否有数据
  │   │   ├─ 有数据但长度不对 → 检查 null terminator 或长度参数
  │   │   ├─ 有数据但编码错 → 检查 UTF-8 vs UTF-16
  │   │   └─ 无数据 → 检查输入参数是否正确
  │   └─ 返回 void → 签名可能写入了 memory 的某个地址，需检查调用约定
  │
  ├─ 检查输入参数
  │   ├─ 字符串未正确写入 memory → 检查 malloc + write 流程
  │   ├─ 参数顺序错误 → 对照 WASM 导出函数签名
  │   └─ 参数类型错误 → number vs BigInt vs 指针
  │
  ├─ 检查 memory 视图
  │   ├─ 内存增长后旧视图失效 → 重新创建 Uint8Array(memory.buffer, ...)
  │   └─ 偏移计算错误 → 用 console.log 打印 ptr 和 len
  │
  └─ 检查调用顺序
      ├─ 是否需要先调用 init 函数 → 检查 Emscripten 的 _main / __wasm_call_ctors
      └─ 是否需要设置全局状态 → 检查 Module 对象的初始化
```

### 性能问题

| 现象 | 可能原因 | 解决方案 |
|---|---|---|
| 单次调用慢（>100ms） | 内存不足频繁增长 | 增大 INITIAL_MEMORY |
| 多次调用累积变慢 | memory 泄漏 | 检查 free 调用，用 buffer pool |
| WASM 编译慢 | 文件大（>5MB） | 用 streaming 编译，或缓存编译结果 |
| 频繁 GC | 大量临时 Uint8Array | 复用视图，避免每次 new |

**缓存编译结果**：

```javascript
const moduleCache = new Map();

async function loadWasmCached(wasmPath, importObject) {
    if (!moduleCache.has(wasmPath)) {
        const buffer = fs.readFileSync(wasmPath);
        const module = await WebAssembly.compile(buffer);
        moduleCache.set(wasmPath, module);
    }

    const module = moduleCache.get(wasmPath);
    return await WebAssembly.instantiate(module, importObject);
}
```

## 整包 Emscripten bundle 黑盒执行

> **触发条件**：目标不是「干净 `.wasm` + 明确导出函数」，而是压缩 webpack bundle 内嵌 WASM base64 + 异步 Emscripten glue + 内部 fetch（典型：风控 / 验证码握手 SDK，如 handshake 类接口）。
> **核心原则（强规则）**：确认加密逻辑落在 WASM 后，先整包黑盒执行——Node `vm` 加载原版 glue，mock `window/document/第三方 SDK/fetch`，hook fetch 抓 body；**禁止先反编译 WASM、逐字节解析 body 或手撕字节码**。黑盒方案跑通后再按需静态提取。

### 与「干净 wasm 加载」的区别

| 维度 | 干净 `.wasm` + 导出函数 | 整包 Emscripten bundle |
|---|---|---|
| 产物形态 | 独立 `.wasm` + 明确导出 | webpack bundle 内嵌 base64 + glue JS |
| 调用方式 | `WebAssembly.instantiate` + `exports` | `vm` 跑原版 bundle，等 glue 初始化后调导出 / 触发内部 fetch |
| 环境依赖 | 少量 import | `window`/`document`/第三方 SDK/异步回调全要 mock |
| 输出抓取 | 读 memory | hook 内部 fetch 抓请求 body |
| 默认策略 | 可直接加载 | 整包黑盒，不拆不反编译 |

### 步骤

1. **先确认结论再动手**：IDENTIFY 定位加密入口 → 证据指向 WASM（`WebAssembly.instantiate` / `.wasm` / 内嵌 base64 魔数）即停，禁止继续手撕 body 字节。
2. **整包原样落盘**：把压缩后的 bundle（glue + 内嵌 wasm base64）原样存盘，不格式化、不 beautify、不反混淆。
3. **vm 加载原版 bundle**：

    ```javascript
    const vm = require('vm');
    const ctx = vm.createContext(sandbox);
    vm.runInContext(code, ctx, { filename: 'bundle.js' });
    ```

    注意同一 bundle 内多次 `runInContext` 会重建 realm，**跨 realm 的 `TypeError instanceof` 失效**——错误类型判断与真实现不同、构造器判断失败；需要跨 realm 类型判断时在 sandbox 内完成，或把 sandbox 实例导出回宿主互转。
4. **mock 环境（trace 驱动的最小集合）**：
   - `window`/`document`/`navigator`/`location`/`crypto`：只补 bundle 实际读取的（见 `env-object-model.md`），不一次性伪造大量 API。
   - `fetch`：记录 URL / body / headers，返回 mock Response（`{ ok:true, json:async()=>({}), text:async()=>'' }`），让 glue 内部请求走通。
   - 第三方验证码 SDK（如 `CaptchaSDK`）：mock 其回调生命周期——`onVerify`/`onStatusChange` 按 SDK 调用顺序触发，`R.show()` 等 UI 方法返回 null 而非抛错。
5. **等异步初始化**：Emscripten Asyncify 下 `Module.onRuntimeInitialized` / `ccall` 未必主动触发；`ccall` 不触发不要死磕，直接在 sandbox 里查导出与 `CI.currData`，或用 hook 在回调处取中间值。
6. **抓 body**：hook 内部 fetch 调用点，把 body 参数原样输出，与取证样本（`target-hits.json`）逐字段比对，结构一致即交付，再进 `REAL_VERIFY`。

### 已踩过的坑（模板化，遇到直接照做）

| 坑 | 现象 | 解法 |
|---|---|---|
| Node v24 `WebAssembly.Table` `funcref`/`anyfunc` 不兼容 | 实例化报 table 元素类型错误 | 按 `WebAssembly.Module.imports()` 读到的声明手动指定 element 类型，不依赖默认值 |
| 跨 vm realm `instanceof` 失效 | `TypeError` / 自定义错误跨 realm 判断 false | 类型判断在 sandbox 内完成，或 sandbox 实例导出回宿主互转 |
| Asyncify 下 `ccall` 不触发 | 导出了 `ccall` 但调用无效果 | 不依赖 `ccall`；hook 回调 / `CI.currData` 取中间值 |
| 第三方 SDK mock 不全 | `R.show()` 为 null 报错 | 按回调生命周期 mock（`onVerify`/`onStatusChange`），UI 方法返回 null |
| bundle 依赖 `document.currentScript` / 动态 `import()` | 加载报错或找不到 chunk | mock `document.currentScript`；把内嵌 chunk 的 base64 解出后手动注册；`import` 注入同步返回 Promise 的 mock |
| glue 内部 `fetch` 未走通 | body 一直为空 | 确认 sandbox 的 `fetch` 是 glue 闭包捕获的那个引用，必要时用 `runInContext` 改写全局 |

> 完整 harness 模板见 `templates/wasm-loader/emscripten-bundle-blackbox.js`。

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/env/env-wasm.md` | WASM 基础加载、wasm-bindgen / Emscripten 简要补全、基础调试 |
| `references/env/env-object-model.md` | WebAssembly 对象的原型链 / descriptor / native-like 保护 |
| `references/env/env-native-protection.md` | WASM 调用的 navigator / document / crypto 等 env 对象保护 |
| `references/workflow/worker-signing.md` | Worker / Service Worker 中加载 WASM 生成签名的分析路径 |
| `templates/wasm-loader/loader.js` | WASM 加载器交付模板（干净 `.wasm` + 导出函数） |
| `templates/wasm-loader/emscripten-bundle-blackbox.js` | 整包 Emscripten bundle 黑盒执行 harness（webpack 内嵌 wasm base64 + glue） |
