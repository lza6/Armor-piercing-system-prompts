# 浏览器 Hook 模板

本文件提供授权调试时用于定位加密参数的最小 Hook 模板。原则：只观察、不篡改、不批量请求；命中后用调用栈回到源码确认。

## 使用规则

- 只在用户授权的目标页面中使用。
- Hook 前先记录目标 API、参数名和页面 URL。
- Hook 代码只输出调用栈和关键值，不修改请求内容。
- 命中后尽快移除 Hook，避免影响目标页面逻辑。
- 不用 Hook 绕过登录、验证码、MFA 或风控。
- 调用栈确认并写入 notes 后立即清理或归档 Hook 代码。

## 注入方式

### 通过 ruyiPage 注入（推荐）

```
ruyiPage: add_init_script(script=HookScript)
→ 在页面脚本加载前注入，确保 Hook 在目标代码之前生效

ruyiPage: evaluate_js(expression=HookScript)
→ 在当前页面上下文执行，适合页面已加载后的动态注入

ruyiPage: inject_hook_preset(preset="xhr|fetch|crypto|websocket|debugger_bypass|cookie|runtime_probe")
→ 一键注入预设 Hook，覆盖常见逆向场景

ruyiPage: hook_function(function_path="目标函数", hook_code="...", position="before|after|replace")
→ 对指定函数注入自定义 Hook
```

### ruyiPage 注入最佳实践

1. **使用 `add_init_script`**：确保 Hook 在目标代码之前生效
2. **优先使用 `inject_hook_preset`**：一键注入 xhr/fetch/crypto/websocket/debugger_bypass/cookie/runtime_probe 预设 Hook
3. **使用 `hook_function`**：对特定函数注入 before/after/replace Hook
4. **使用 `console.log` 输出**：通过 `get_console_logs` 收集结果
5. **使用 `console.trace`**：在关键点输出调用栈
6. **使用 Proxy 代替直接覆写**：更隐蔽，不改变 `typeof` 结果
7. **首屏挑战页需在页面脚本执行前装好 Hook**：RS/Akamai 首包挑战在 hook 装好前就跑完了，必须让 hook 先装再加载目标页面
8. **装完 hook 想让它先于页面 JS 跑**：带 hook 重新加载页面，同时清理各类 hook 日志变量

## Hook 模板库

### 1. Cookie Setter Hook

**用途**：监控所有 Cookie 写入操作，定位动态 Cookie 生成逻辑

```javascript
(() => {
  const targetParam = "sign";
  const proto = Document.prototype;
  const desc = Object.getOwnPropertyDescriptor(proto, "cookie");
  if (!desc || !desc.set || !desc.get) return;
  // 关键：在原型链 owner 上替换，而不是 document 实例
  Object.defineProperty(proto, "cookie", {
    configurable: true,
    enumerable: desc.enumerable,
    get() {
      const value = desc.get.call(this);
      if (String(value).includes(targetParam)) {
        console.trace("document.cookie 读取命中");
      }
      return value;
    },
    set(value) {
      if (String(value).includes(targetParam)) {
        console.group("[cookie 写入命中]");
        console.log(value);
        console.trace("cookie 写入调用栈");
        console.groupEnd();
        debugger;
      }
      return desc.set.call(this, value);
    }
  });
})();
```

### 2. XHR Hook

**用途**：拦截所有 XMLHttpRequest 请求，捕获完整请求参数

```javascript
(() => {
  const targetParam = "sign";
  const apiKeyword = "/api/";
  const rawOpen = XMLHttpRequest.prototype.open;
  const rawSend = XMLHttpRequest.prototype.send;
  const rawSetHeader = XMLHttpRequest.prototype.setRequestHeader;

  XMLHttpRequest.prototype.open = function patchedOpen(method, url) {
    this.__reverse_method = method;
    this.__reverse_url = String(url || "");
    return rawOpen.apply(this, arguments);
  };

  XMLHttpRequest.prototype.setRequestHeader = function patchedSetRequestHeader(name, value) {
    if (String(name).includes(targetParam) || String(value).includes(targetParam)) {
      console.group("[XHR Header 命中]", this.__reverse_url);
      console.log(name, value);
      console.trace("setRequestHeader 调用栈");
      console.groupEnd();
      debugger;
    }
    return rawSetHeader.apply(this, arguments);
  };

  XMLHttpRequest.prototype.send = function patchedSend(body) {
    const url = this.__reverse_url || "";
    if (url.includes(apiKeyword) || url.includes(targetParam) || String(body || "").includes(targetParam)) {
      console.group("[XHR send 命中]", this.__reverse_method, url);
      console.log("body =", body);
      console.trace("send 调用栈");
      console.groupEnd();
      debugger;
    }
    return rawSend.apply(this, arguments);
  };
})();
```

### 3. Fetch Hook

**用途**：拦截 fetch API 请求

```javascript
(() => {
  const targetParam = "sign";
  const apiKeyword = "/api/";
  const rawFetch = window.fetch;
  window.fetch = function patchedFetch(input, init) {
    const url = typeof input === "string" ? input : (input && input.url) || "";
    const body = init && init.body;
    if (url.includes(apiKeyword) || String(url).includes(targetParam) || String(body || "").includes(targetParam)) {
      console.group("[fetch 命中]", url);
      console.log("input =", input);
      console.log("init =", init);
      console.trace("fetch 调用栈");
      console.groupEnd();
      debugger;
    }
    return rawFetch.apply(this, arguments);
  };
})();
```

### 4. $.ajax Hook（jQuery 场景）

**用途**：拦截 jQuery AJAX 请求，捕获加密参数

```javascript
(() => {
  if (typeof $ === 'undefined' || typeof $.ajax === 'undefined') return;

  const _ajax = $.ajax;
  $.ajax = function(options) {
    if (typeof options === 'object') {
      console.log('[Hook] $.ajax:', {
        url: options.url,
        method: options.type || options.method,
        data: options.data,
        headers: options.headers
      });

      if (options.url && options.url.indexOf('目标接口') !== -1) {
        console.log('[Hook] ★ 目标Ajax捕获');
        if (options.data && options.data.m) {
          console.log('[Hook] 加密参数 m:', options.data.m);
        }
      }
    }
    return _ajax.apply(this, arguments);
  };

  // 同时 Hook ajaxSetup
  if ($.ajaxSetup) {
    const _setup = $.ajaxSetup;
    $.ajaxSetup = function(options) {
      console.log('[Hook] $.ajaxSetup:', options);
      return _setup.apply(this, arguments);
    };
  }
})();
```

### 5. URLSearchParams / FormData Hook

**用途**：捕获参数写入，常用于定位签名拼接

```javascript
(() => {
  const targetParam = "sign";
  for (const Ctor of [URLSearchParams, FormData].filter(Boolean)) {
    for (const method of ["append", "set"]) {
      const raw = Ctor.prototype[method];
      if (typeof raw !== "function") continue;
      Ctor.prototype[method] = function patchedParamWrite(name, value) {
        if (String(name) === targetParam || String(value).includes(targetParam)) {
          console.group(`[${Ctor.name}.${method} 命中]`, name);
          console.log("value =", value);
          console.trace("参数写入调用栈");
          console.groupEnd();
          debugger;
        }
        return raw.apply(this, arguments);
      };
    }
  }
})();
```

### 6. eval / Function Hook

**用途**：捕获动态生成和执行的代码

```javascript
(() => {
  // eval Hook
  const _eval = window.eval;
  window.eval = function(code) {
    console.log('[Hook] eval 调用, 代码长度:', (typeof code === 'string') ? code.length : 'N/A');
    if (typeof code === 'string' && code.length < 5000) {
      console.log('[Hook] eval 代码:', code.substring(0, 500));
    }
    return _eval.apply(this, arguments);
  };

  // Function 构造器 Hook
  const _Function = Function;
  const handler = {
    construct(target, args) {
      const body = args[args.length - 1];
      console.log('[Hook] new Function, body 长度:', body ? body.length : 0);
      if (body && body.indexOf('目标关键词') !== -1) {
        console.log('[Hook] ★ 目标 Function 捕获:', body.substring(0, 500));
      }
      return new target(...args);
    },
    apply(target, thisArg, args) {
      const body = args[args.length - 1];
      console.log('[Hook] Function(), body 长度:', body ? body.length : 0);
      return target.apply(thisArg, args);
    }
  };
  window.Function = new Proxy(_Function, handler);
})();
```

### 7. JSON.parse / JSON.stringify Hook

**用途**：捕获 JSON 解析 / 序列化操作，常用于响应解密或签名前

```javascript
(() => {
  const _parse = JSON.parse;
  JSON.parse = function(text) {
    const result = _parse.apply(this, arguments);
    console.log('[Hook] JSON.parse:', typeof text === 'string' ? text.substring(0, 200) : text);
    return result;
  };

  const _stringify = JSON.stringify;
  JSON.stringify = function(obj) {
    const result = _stringify.apply(this, arguments);
    console.log('[Hook] JSON.stringify:', result ? result.substring(0, 200) : result);
    return result;
  };
})();
```

### 8. atob / btoa Hook

**用途**：捕获 Base64 编解码操作

```javascript
(() => {
  const _atob = window.atob;
  const _btoa = window.btoa;

  window.atob = function(str) {
    const result = _atob(str);
    console.log('[Hook] atob:', str.substring(0, 100), '→', result.substring(0, 100));
    return result;
  };

  window.btoa = function(str) {
    const result = _btoa(str);
    console.log('[Hook] btoa:', str.substring(0, 100), '→', result.substring(0, 100));
    return result;
  };
})();
```

### 9. localStorage / sessionStorage Hook

**用途**：监控 Storage 读写

```javascript
(() => {
  const keys = ["token", "sign", "device", "fingerprint"];
  for (const storageName of ["localStorage", "sessionStorage"]) {
    const storage = window[storageName];
    if (!storage) continue;
    for (const method of ["getItem", "setItem", "removeItem"]) {
      const raw = storage[method];
      storage[method] = function patchedStorage(key, value) {
        if (keys.some(k => String(key).includes(k))) {
          console.group(`[${storageName}.${method} 命中]`, key);
          console.log("value =", value);
          console.trace("Storage 调用栈");
          console.groupEnd();
        }
        return raw.apply(this, arguments);
      };
    }
  }
})();
```

### 10. 时间、随机数 Hook

**用途**：监控时间戳和随机数生成，常用于签名一致性排查

```javascript
(() => {
  for (const [owner, name] of [[Date, "now"], [Math, "random"]]) {
    const raw = owner[name];
    owner[name] = function patchedTimeRandom() {
      const ret = raw.apply(this, arguments);
      console.log(`[${name}]`, ret);
      console.trace(`${name} 调用栈`);
      return ret;
    };
  }

  const rawGetRandomValues = crypto && crypto.getRandomValues;
  if (rawGetRandomValues) {
    crypto.getRandomValues = function patchedGetRandomValues(arr) {
      const ret = rawGetRandomValues.apply(this, arguments);
      console.log("[crypto.getRandomValues]", Array.from(arr));
      console.trace("crypto 随机数调用栈");
      return ret;
    };
  }
})();
```

### 11. WebSocket / postMessage Hook

**用途**：拦截 WebSocket 消息和 postMessage 通信

```javascript
(() => {
  const rawWS = window.WebSocket;
  if (rawWS) {
    window.WebSocket = new Proxy(rawWS, {
      construct(target, args) {
        console.log("[WebSocket 创建]", args);
        console.trace("WebSocket 调用栈");
        const ws = Reflect.construct(target, args);
        const _send = ws.send.bind(ws);
        ws.send = function(data) {
          console.log('[Hook] WS 发送:', data);
          return _send(data);
        };
        ws.addEventListener('message', function(event) {
          console.log('[Hook] WS 接收:', event.data);
        });
        return ws;
      }
    });
  }

  const rawPostMessage = window.postMessage;
  if (rawPostMessage) {
    window.postMessage = function patchedPostMessage(message, targetOrigin, transfer) {
      console.log("[postMessage]", message, targetOrigin);
      console.trace("postMessage 调用栈");
      return rawPostMessage.apply(this, arguments);
    };
  }
})();
```

### 12. Canvas 指纹 Hook

**用途**：拦截 Canvas 指纹采集

```javascript
(() => {
  const _toDataURL = HTMLCanvasElement.prototype.toDataURL;
  HTMLCanvasElement.prototype.toDataURL = function() {
    console.log('[Hook] Canvas toDataURL 调用');
    console.trace('[Hook] Canvas 调用栈');
    return _toDataURL.apply(this, arguments);
  };

  const _toBlob = HTMLCanvasElement.prototype.toBlob;
  HTMLCanvasElement.prototype.toBlob = function() {
    console.log('[Hook] Canvas toBlob 调用');
    return _toBlob.apply(this, arguments);
  };

  const _getContext = HTMLCanvasElement.prototype.getContext;
  HTMLCanvasElement.prototype.getContext = function(type) {
    console.log('[Hook] Canvas getContext:', type);
    return _getContext.apply(this, arguments);
  };

  const _getParameter = WebGLRenderingContext.prototype.getParameter;
  WebGLRenderingContext.prototype.getParameter = function(param) {
    const value = _getParameter.apply(this, arguments);
    console.log('[Hook] WebGL getParameter:', param, '→', value);
    return value;
  };
})();
```

### 13. Navigator 属性 / setTimeout / setInterval Hook

**用途**：伪装浏览器指纹、监控定时器调用、识别反调试

```javascript
(() => {
  // Navigator 属性（伪装指纹；仅在授权调试时使用）
  Object.defineProperty(navigator, 'webdriver', {
    get: () => false,
    configurable: true
  });
  Object.defineProperty(navigator, 'plugins', {
    get: () => [1, 2, 3, 4, 5],
    configurable: true
  });
  Object.defineProperty(navigator, 'languages', {
    get: () => ['zh-CN', 'zh', 'en'],
    configurable: true
  });

  // setTimeout / setInterval（过滤 debugger 反调试）
  const _setTimeout = window.setTimeout;
  const _setInterval = window.setInterval;

  window.setTimeout = function(fn, delay) {
    const fnStr = typeof fn === 'function' ? fn.toString().substring(0, 200) : String(fn).substring(0, 200);
    if (fnStr.indexOf('debugger') !== -1) {
      console.log('[Hook] setTimeout 拦截 debugger，已跳过');
      return -1;
    }
    return _setTimeout.apply(this, arguments);
  };

  window.setInterval = function(fn, delay) {
    const fnStr = typeof fn === 'function' ? fn.toString().substring(0, 200) : String(fn).substring(0, 200);
    if (fnStr.indexOf('debugger') !== -1) {
      console.log('[Hook] setInterval 拦截 debugger，已跳过');
      return -1;
    }
    return _setInterval.apply(this, arguments);
  };
})();
```

## 组合 Hook 模板（一键注入）

```javascript
(() => {
  console.log('[Hook] ========== 通用逆向Hook已注入 ==========');

  // 1. Cookie 监控（原型链级）
  const cookieDesc = Object.getOwnPropertyDescriptor(Document.prototype, 'cookie');
  if (cookieDesc) {
    Object.defineProperty(Document.prototype, 'cookie', {
      configurable: true,
      enumerable: cookieDesc.enumerable,
      get() { return cookieDesc.get.call(this); },
      set(v) { console.log('[Cookie] Set:', v); cookieDesc.set.call(this, v); }
    });
  }

  // 2. XHR 监控
  const _xhrOpen = XMLHttpRequest.prototype.open;
  const _xhrSend = XMLHttpRequest.prototype.send;
  XMLHttpRequest.prototype.open = function(m, u) { this._m = m; this._u = u; return _xhrOpen.apply(this, arguments); };
  XMLHttpRequest.prototype.send = function(b) { console.log('[XHR]', this._m, this._u, b); return _xhrSend.apply(this, arguments); };

  // 3. Fetch 监控
  const _fetch = window.fetch;
  window.fetch = function() { console.log('[Fetch]', ...arguments); return _fetch.apply(this, arguments); };

  // 4. debugger 拦截
  const _si = window.setInterval;
  window.setInterval = function(fn, d) {
    if (typeof fn === 'function' && fn.toString().indexOf('debugger') > -1) return -1;
    return _si.apply(this, arguments);
  };

  console.log('[Hook] ========== Hook注入完成 ==========');
})();
```

## cookie 预设：原型链级 document.cookie Hook

### 为什么不能直接 `Object.defineProperty(document, 'cookie', ...)`

`document.cookie` 的 getter/setter 定义在**原型链**上（具体是 `Document.prototype` 或 `HTMLDocument.prototype`），不在 `document` 实例上。直接在实例上 defineProperty 会被浏览器忽略或抛错——这是很多自写 Hook 脚本"看起来装上了但完全抓不到 cookie 写入"的根本原因。

### 正确做法

```javascript
// 沿原型链找到定义 cookie 描述符的 owner
function findCookieDescriptor() {
    var proto = Object.getPrototypeOf(document);
    while (proto) {
        var d = Object.getOwnPropertyDescriptor(proto, 'cookie');
        if (d) return { descriptor: d, owner: proto };
        proto = Object.getPrototypeOf(proto);
    }
    return null;
}

var found = findCookieDescriptor();
if (!found) return;

var origSet = found.descriptor.set;
var origGet = found.descriptor.get;

// 在正确的 owner 上替换（关键：owner 是 Document.prototype 或 HTMLDocument.prototype）
Object.defineProperty(found.owner, 'cookie', {
    set: function (value) {
        window.__mcp_cookie_log.push({
            op: 'set', value: String(value),
            stack: new Error().stack, ts: Date.now()
        });
        return origSet.call(this, value);
    },
    get: function () {
        var v = origGet.call(this);
        window.__mcp_cookie_log.push({
            op: 'get', value: String(v),
            stack: new Error().stack, ts: Date.now()
        });
        return v;
    },
    configurable: true, enumerable: true
});
```

### 适用场景

- 所有涉及 JS 写入 cookie 的场景（eval 首包、指纹 cookie、JS 计算 token 后 `document.cookie = ...`）
- **不适用**：HTTP Set-Cookie 写入的场景（这种用网络层抓包）

## runtime_probe 预设：低开销广谱运行时探针

### 与 jsvmp_hook 的区别

| 维度 | `jsvmp_hook`（`hook_jsvmp_interpreter`） | `runtime_probe` |
|------|---------------------------------------|----------------|
| 实现方式 | 在 navigator/screen 等全局对象上装 **Proxy** | **不装 Proxy**，只 override 具体热点 API |
| 开销 | 高（每次属性读取都进 Proxy trap） | 低（只在调用热点 API 时记录） |
| 覆盖面 | 全局对象所有属性 + apply/call/bind/Reflect.* | 固定一组 API |
| 安全性 | 个别页面可能被 Proxy 破坏 | 非常安全 |
| 典型用途 | JSVMP 深度分析 | "这个页面都在做什么"快速摸底 |

### runtime_probe 覆盖的 API 清单

| 类别 | 覆盖项 | 日志 type |
|------|-------|----------|
| XHR | `XMLHttpRequest.prototype.open/send` | `xhr_open` / `xhr_send` |
| fetch | `window.fetch` | `fetch` |
| Canvas 指纹 | `HTMLCanvasElement.prototype.toDataURL` | `canvas_toDataURL` |
| Canvas 上下文 | `HTMLCanvasElement.prototype.getContext` | `canvas_getContext` |
| WebGL | `WebGLRenderingContext.prototype.getParameter` | `webgl_getParameter` |
| navigator | userAgent / platform / language / languages / webdriver / hardwareConcurrency / deviceMemory / vendor / appVersion / plugins / mimeTypes 的 getter | `nav_read` |
| 事件 | `EventTarget.prototype.addEventListener`（只记 mouse/key/devicemotion 等 bot 检测类） | `addEventListener` |

### 典型诊断模式

- **反爬是否做 canvas 指纹检测？** → 看 `by_type.canvas_toDataURL` 是否 > 0
- **反爬是否读 navigator.webdriver？** → `get_runtime_probe_log(type_filter="nav_read")` 里找 `prop: "webdriver"`
- **反爬是否检测鼠标移动？** → 看 `by_type.addEventListener` 里 mousemove/mousedown 的计数

## 证据输出要求

命中 Hook 后至少记录：

- 命中的 API 或参数名。
- 调用栈截图或文本。
- 所在 JS 文件、行列号、chunk 名。
- 写入前后请求对象。
- 是否可复现。
- 是否需要 sourcemap 或动态 chunk。

调用栈确认并写入 `case/notes/` 后，立即从 `case/hooks/` 清理或归档临时 Hook 脚本。

## 相关案例

| 案例文件 | 关联点 |
|---------|--------|
| `cases/jsvmp-xhr-interceptor-env-emulation.md` | XHR Hook 模板 + 加载时序（SDK 前） |
| `cases/universal-vmp-source-instrumentation.md` | 源码级插桩 + 兜底 Hook 模板 |
