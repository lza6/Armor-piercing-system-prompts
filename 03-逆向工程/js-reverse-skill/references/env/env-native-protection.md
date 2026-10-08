# 真实性保护、Proxy 风险与 native-like 行为

本文件描述 JS 层补环境保护策略。如需 C++ 级保护（如 document.all HTMLDDA），建议使用 sdenv。

每次进入 Node.js 补环境阶段、准备编写 env 模块、或修改任何 WebAPI 对象模型时读取本文件。真实性保护不是等目标 JS 检测到再补，而是补环境默认基线；除非用户明确要求不做保护，否则必须从补环境初始化开始执行。

## 补环境初始化硬性基线（不等待检测）

进入补环境阶段后，先执行以下基线，再运行目标 JS：

1. 从第一版 env 骨架开始就使用 NativeProtect 或等价 toString 保护；不要先用普通函数 / 普通赋值跑通，等检测到 `toString`、descriptor、原型链问题后再补保护。
2. 所有新增 WebAPI 默认使用 `Object.defineProperty` / `defineProperties`，并显式设置 `writable`、`enumerable`、`configurable`。
3. 所有方法、构造函数、getter、setter、实例对象都默认做 native-like / toString / `Symbol.toStringTag` / 原型链保护。

该基线是规范性要求，不以"目标是否已经检测到"为触发条件。

## Proxy 只用于探测

JS `Proxy` 很适合发现缺失环境，但不适合作为最终交付。

目标 JS 可能通过以下方式发现异常：

```js
Object.keys(obj)
Reflect.ownKeys(obj)
Object.getOwnPropertyDescriptor(obj, 'xxx')
Object.getPrototypeOf(obj)
obj instanceof SomeConstructor
Object.prototype.toString.call(obj)
Function.prototype.toString.call(fn)
'xxx' in obj
obj.constructor.name
```

策略：

| 阶段 | 做法 |
|---|---|
| 初次运行 | 可以使用全量 Proxy 探测访问路径 |
| 中期调试 | 已知对象改为真实结构，只对未知分支继续 Proxy |
| 最终交付 | 尽量无 Proxy，用真实对象、描述符、原型链和 native-like 函数 |

即使 trace 暂时没有出现 `ownKeys`、`getOwnPropertyDescriptor`、`getPrototypeOf`、`toString`、`instanceof` 等信号，新增关键 WebAPI 也应按真实对象、描述符、原型链和 native-like 函数实现；当这些信号出现时，应立即把残留 Proxy 迁移到真实对象。

## 补环境阶段默认真实性清单

从补环境阶段开始，不能只补到"不报错"，也不能只在检测命中后才补保护。凡新增、修改或被 RuyiTrace / Node trace / 目标检测命中的 WebAPI，都必须逐项确认：

1. **原型链**：补构造函数、`prototype.constructor`、`Object.create(Constructor.prototype)`、必要的 `Object.setPrototypeOf` 多级链路，并验证 `instanceof`。
2. **属性描述符**：所有关键属性都用 `Object.defineProperty` / `defineProperties`；明确 `writable`、`enumerable`、`configurable`，不要用普通赋值替代。
3. **访问器**：真实浏览器中是 getter / setter 的属性，必须补成 accessor descriptor；不要为了省事改成 data descriptor。
4. **函数 toString 保护**：普通方法、构造函数、原型方法使用 `NativeProtect.setNativeFunc` 保护。
5. **访问器 toString 保护**：getter / setter 本身也是函数；使用 `NativeProtect.setNativeFunc(getter, "get xxx")` / `setNativeFunc(setter, "set xxx")` 保护。
6. **实例对象 toString 保护**：对 `navigator`、`document`、`localStorage`、`screen`、`location` 等实例，使用 `Symbol.toStringTag`、`NativeProtect.setObjFunc(obj, "Navigator")` 保护 `Object.prototype.toString` 行为。
7. **集合对象**：`HTMLCollection`、`NodeList`、`PluginArray`、`MimeTypeArray`、`DOMTokenList` 等集合对象必须用真实原型链 + 描述符实现，不得以普通数组 / 普通对象作为主路径。
8. **plugins / mimeTypes**：`navigator.plugins` 和 `navigator.mimeTypes` 必须按真实浏览器结构回放，手写真实数据，不能手写 `[]` 或普通对象作为主路径。
9. **特殊对象**：`document.all` 这类 HTMLDDA / 不可检测对象在 JS 层只能标记为近似，不得声称完全一致；如需精确 HTMLDDA 行为，建议使用 sdenv。
10. **指纹终端 API**：Canvas / WebGL / WebGPU / Audio / 字体 / DOM 几何等指纹 API 必须优先回放真实浏览器采样值，同时保持 API 所在对象的原型链、描述符和 native-like `toString`；不得因为 Node.js 无法真实渲染就把最终流程改成自动化。

建议交付前运行：

```bash
node scripts/check_fingerprint_fixture.js --case-dir <project-root> --require canvas,webgl --markdown
```

## 指纹值回放真实性

即使 `toDataURL`、`getParameter`、`getBoundingClientRect` 等方法只是返回 fixture 采样值，也要像浏览器 native API：

- 方法挂在正确 prototype 上，例如 `HTMLCanvasElement.prototype.toDataURL`。
- 函数名、`length`、`toString()` 尽量 native-like。
- 实例对象满足 `Object.prototype.toString.call(canvas)` / `instanceof HTMLCanvasElement` 等目标检测。
- 缺少采样值时抛出明确错误，不能悄悄返回空字符串、空数组或随机值。
- 不使用 `node-canvas` / `headless-gl` 作为最终指纹一致性方案；这些库可用于离线探索，但不能替代真实浏览器采样值。

## toString / native-like 保护优先级

推荐优先级：

1. JS 层真实对象 + 描述符 + 原型链。
2. `NativeProtect` toString / descriptor 保护；必须同时覆盖普通函数、访问器 getter / setter 和实例对象 `Object.prototype.toString`。
3. 如需更强保护可使用 sdenv（用户自备，自带 C++ Addon）。

## JS 能力缺口闭环

如果目标行为经过真实浏览器采样后确认：JS 层 NativeProtect 无法可靠表达该行为，则不要继续硬凑补环境代码。如需 C++ 级保护（如 document.all HTMLDDA），建议使用 sdenv（用户自备，自带 C++ Addon）。

## JS 层保护基线

进入补环境阶段，env helper 必须使用 JS 层 NativeProtect 保护 toString / descriptor / 原型链。实现规则：

1. 创建函数、构造函数、getter、setter、集合对象、`navigator.plugins` / `mimeTypes`、`document.all` 等对象时，使用 `NativeProtect.setNativeFunc` / `setObjFunc` 保护 toString，并用 `Object.defineProperty` 显式设置描述符与原型链。
2. 交付前复查 JS 层保护证据：源码应出现 `NativeProtect` 初始化、`setNativeFunc` / `setObjFunc` 调用、`Object.defineProperty` 配置描述符与原型链；如果只有普通函数 / 普通赋值，应视为失败。

## 构造函数报错的 native 保护策略

构造函数错误采样（采样字段 `error.name` / `error.constructor.name` / `error.message` / `String(error)` / stack 首行、`case/fixtures/constructor-errors.fixture.json` 路径、"不要统一写 `Illegal constructor`"等取证要求、采样模板）详见 `references/env/env-object-model.md` 的"构造函数行为采样"段。本文件只关注 native 保护策略：

1. 构造函数的非法调用 / 非法构造行为必须由 JS 层抛出与采样一致的错误（错误构造器、`error.name`、`error.message`、stack 首行均需匹配）。
2. 错误信息要精确到浏览器版本和调用方式；例如 Chrome 中"需要 new 调用"和"非法构造"通常不是同一条 message，应分别实现对应抛错逻辑。
3. 不得统一写成 `throw new TypeError('Illegal constructor')`。

## `document.all`

`document.all` 的关键行为：

```js
typeof document.all              // 'undefined'
document.all == undefined        // true
document.all === undefined       // false
Boolean(document.all)             // false
'all' in document                 // true
```

JS 层只能近似：

```js
Object.defineProperty(document, 'all', {
  value: undefined,
  enumerable: false,
  configurable: true,
});
```

近似方案不能满足 `document.all !== undefined` 和 `Boolean(document.all) === false`，必须在报告中说明。如需精确 HTMLDDA 行为，建议使用 sdenv（自带 C++ Addon）。

## native-like 函数

使用 `NativeProtect`：

```js
const nativeProtect = NativeProtect.getInstance();

function querySelector(selector) {
  return null;
}

nativeProtect.setNativeFunc(querySelector, 'querySelector');

Object.defineProperty(Element.prototype, 'querySelector', {
  value: querySelector,
  writable: true,
  enumerable: true,
  configurable: true,
});

querySelector.toString();
// function querySelector() { [native code] }
```

getter / setter：

```js
function getUserAgent() {
  return userAgent;
}

nativeProtect.setNativeFunc(getUserAgent, 'get userAgent');

Object.defineProperty(Navigator.prototype, 'userAgent', {
  get: getUserAgent,
  enumerable: true,
  configurable: true,
});
```

## 多通道 toString 与 DataCloneError 保护

高强度检测不会只调用 `fn.toString()`。凡进入补环境范围的函数、构造函数、getter、setter 都要按以下通道验证：

- `fn.toString()`
- `Function.prototype.toString.call(fn)`
- 目标 JS 先保存 `const FTS = Function.prototype.toString` 后再 `FTS.call(fn)`
- `String(fn)`
- `fn + ""`
- `fn.toString.toString()`
- `structuredClone(fn)` 抛出的 `DataCloneError` message / stack
- `MessagePort.prototype.postMessage(fn)` 抛出的 `DataCloneError` message / stack

`NativeProtect` 必须使用带 `structuredClone` / `MessagePort.prototype.postMessage` DataCloneError 改写的版本，覆盖上述多通道 toString 检测；旧版只 patch `Function.prototype.toString` 的 NativeProtect 不再作为主推荐。

## `NativeProtect`

在目标 JS 所在运行上下文内使用 `NativeProtect` 保护。必须在加载目标 JS 之前执行。

完整实现见 `assets/env-patch-snippets/native-protect.js`（可被 `templates/vm-sandbox/install-env.js` 直接 require）。覆盖通道：

- `fn.toString()` / `Function.prototype.toString.call(fn)` / `String(fn)` / `fn + ""`
- `fn.toString.toString()`
- `Object.prototype.toString.call(obj)`
- `structuredClone(fn)` 抛出的 `DataCloneError` message / stack
- `MessagePort.prototype.postMessage(fn)` 抛出的 `DataCloneError` message / stack

```js
// 加载方式（在目标运行上下文内）
const NativeProtect = require('<skill_path>/assets/env-patch-snippets/native-protect.js');
const nativeProtect = NativeProtect.getInstance();
```

使用示例：

```js
const nativeProtect = NativeProtect.getInstance();

function getItem(key) {
  return storageMap.get(String(key)) ?? null;
}

nativeProtect.setNativeFunc(getItem, 'getItem');

Object.defineProperty(Storage.prototype, 'getItem', {
  value: getItem,
  writable: true,
  enumerable: true,
  configurable: true,
});
```

注意：

- 只在目标 JS 所在运行上下文内 patch，不要污染宿主 Node.js 全局环境；该上下文可以是 `vm`、独立 Node 进程或显式隔离的全局对象。
- `NativeProtect` 必须使用带 `structuredClone` / `MessagePort.prototype.postMessage` DataCloneError 改写的版本；如果目标会通过 `structuredClone(fn)` 或 `postMessage(fn)` 检测函数源码，旧版 NativeProtect 会暴露非 native 函数。
- 如果目标 JS 提前保存了原始 `Function.prototype.toString`，后 patch 可能失效。
- 如果使用 `vm`，要确保 patch 发生在目标 JS 所在 context 内；如果不用 `vm`，要确保入口文件以独立进程 / 隔离 global 初始化并在结束后不污染其他任务。

## 属性描述符、访问器与原型链

常见检查：

```js
Object.getOwnPropertyDescriptor(navigator, 'userAgent')
Object.getOwnPropertyDescriptor(window, 'navigator')
Object.getPrototypeOf(document)
navigator.constructor.name
```

处理原则：

- 所有关键属性都用 `Object.defineProperty`，禁止用普通赋值替代关键 WebAPI。
- 对不可写、不可枚举、只读 getter 要显式设置；真实浏览器是 accessor 的属性不得降级为 data descriptor。
- getter / setter 的 `Function.prototype.toString.call(descriptor.get)` / `descriptor.set` 也要 native-like。
- `constructor` 通常不可枚举，并且构造函数本身也要做 toString 保护。
- `Symbol.toStringTag` 与 `NativeProtect.setObjFunc(obj, "Xxx")` 用于 `[object Xxx]` 保护。
- 发现 `instanceof`、`Object.getPrototypeOf`、`constructor.name` 检测时，补构造函数和完整原型链。

## 网络代理检测

如果用户所说"代理检测"是网络代理 / IP 检测，而不是 JS `Proxy`：

- 默认回退到用户真实浏览器手动取证。
- 不默认启用网络代理。
- 如果必须使用代理，必须由用户提供授权代理并确认使用范围。
- 代理被检测时，不尝试绕过风控；应暂停并让用户选择：
  1. 换回用户真实浏览器手动取证。
  2. 提供 HAR / cURL / JS 文件离线分析。
  3. 在授权环境中更换合规网络环境。

## 深度行为审计（移植自 xbs）

toString / descriptor / 原型链保护只是 native-like 的"形似"层。下列三个移植自 xbs 的深度审计文档负责"神似"层——验证补环境对象在 realm 隔离、网络对象语义、TLS Session 复用等维度上是否与真实浏览器一致：

| 审计层 | 文档 | 触发条件 | 关注点 |
|---|---|---|---|
| Trace-runtime 一致性闭环 | `references/quality/trace-runtime-conformance.md` | 高强度检测命中 realm / descriptor / brand / prototype 深度 diff | runtime contract + audit-only/no-send + realm/descriptor/brand/prototype 深度 diff |
| XHR/fetch 语义审计 | `references/network/xhr-fetch-semantics-audit.md` | 目标 JS 调用 `XMLHttpRequest` / `fetch` / `sendBeacon` 且行为被检测 | XHR/fetch/sendBeacon 行为比较 + 真实请求前 no-send diff（no-send / Realm / actor / reload） |
| XHR/fetch Session bridge | `references/network/xhr-fetch-session-bridge.md` | 最终交付需复用同 TLS Session 跨请求保活 | 同 TLS Session 复用 + curl_cffi / curl-cffi-node 桥接 |

`NativeProtect` 的多通道 toString 保护覆盖了上述审计的"函数源码泄露"维度；但 realm 隔离、网络对象语义、Session 复用需要额外的运行时桥接实现，不在 `NativeProtect` 范围内。

## 相关案例

| 案例文件 | 关联点 |
|---------|--------|
| `cases/jsvmp-xhr-interceptor-env-emulation.md` | Function.prototype.toString 三层防御 + markNative |
| `cases/jsvmp-dual-sign-xhr-intercept-cacheOpts-jsdom-firefox.md` | Firefox 格式 native code 伪装（含换行缩进） |
| `cases/jsvmp-ruishu6-cookie-412-sdenv.md` | document.all native HTMLDDA（C++ MarkAsUndetectable） |
