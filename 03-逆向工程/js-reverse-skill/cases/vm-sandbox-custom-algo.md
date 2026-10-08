# Case：vm 沙箱执行自定义算法（骨架模板）

> 难度：★★★（骨架模板）
> 还原方案：B vm 沙箱执行
> 实现语言：Node.js
> 最后验证日期：2026-07-11
> 平台类型：通用骨架（自定义 MD5 / 混淆算法 / 算法不可静态提取但 JS 可 vm 执行）

> **骨架案例**。本文是**方法论模板**，适用于：自定义 MD5/SHA 实现、混淆后算法不可静态还原、算法可提取但依赖少量环境属性（非 JSVMP）等场景。
>
> 使用方式：
> 1. 在 CASE_LOOKUP 指纹匹配时，若检测到"算法不可直接提取但 JS 可 vm 执行"特征，直接走本案例的流程
> 2. 完成具体站点逆向后，复制本文件重命名为 `vm-<具体技术特征>.md`，按真实数据填充占位符

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [ ] 算法函数存在但不可直接提取（自定义 MD5 变种 / 混淆后控制流打乱 / eval 包裹）
- [ ] 无 JSVMP 字节码虚拟机（区别于补环境场景）
- [ ] 无 200KB+ 大文件 + while-switch 解释器（区别于补环境场景）
- [ ] 算法依赖少量环境属性（如 navigator.userAgent / 时间戳），但不依赖完整浏览器环境

### 参数特征
- [ ] 签名参数长度异常（非标准 MD5 的 32 位 / 非 SHA256 的 64 位）
- [ ] 或长度标准但值与标准算法不一致（自定义变种）
- [ ] 签名输入含环境属性（UA / 时间 / 随机数）

### 请求特征
- [ ] 缺/错签名 → 服务端返回 403 或业务码异常
- [ ] 不返回 412 循环（区别于瑞数签名型）
- [ ] 不返回 200 空 body（区别于 JSVMP 行为型）

### 混淆类型
- [ ] OB 混淆但可 AST 反混淆后提取
- [ ] eval/Function 包裹（可 Hook 拦截源码）
- [ ] 无混淆但算法逻辑复杂（自定义哈希表 + 异或 + 位移）

---

## 加密方案

- **算法**：[填入：自定义 MD5 变种 / 混淆后的 AES / 自定义哈希]
- **密钥来源**：[填入：硬编码 / 动态计算 / 接口下发]
- **加密流程**：
  1. [填入：参数收集]
  2. [填入：拼接/排序]
  3. [填入：算法执行]
  4. [填入：编码输出]
- **签名公式**：[填入真实公式，若可提取部分逻辑]

---

## 方案方向

vm 沙箱执行：提取算法 JS 代码 → 在 Node.js `vm` 模块中执行 → 喂入参数截出签名。

与纯算的区别：算法不可直接用 `crypto` 复现（自定义实现），但 JS 代码本身可独立执行（不需要完整浏览器环境）。

与补环境的区别：不需要 jsdom / 补环境 / 浏览器指纹，只需 `vm.createContext` 提供最小 sandbox。

## 标准流程（详见 references/workflow/trace-flow.md）

### FORENSIC_CAPTURE → TRACE_CAPTURE：定位 + 提取

```
1. trace 取证黄金路径定位签名函数
   network_capture → get_request_initiator → 直达签名函数
2. search_code(keyword="参数名") → 定位赋值点
3. scripts(action='save') → 保存算法 JS
4. 识别算法类型：
   - 标准 MD5/SHA/AES → 降级纯算还原
   - 自定义 MD5（chrsz 变化 / 轮函数修改）→ vm 执行
   - 混淆不可静态还原 → vm 执行
5. 提取算法函数 + 依赖的全局变量/常量
```

### TRACE_ANALYZE：vm 沙箱搭建

#### 3.1 基础 sandbox（算法自包含，无浏览器环境依赖）

适用于：算法 JS 只依赖 Date/Math/parseInt 等标准全局变量。

```javascript
const vm = require('vm');

// 最小 sandbox：只提供算法依赖的属性
const sandbox = {
    // 算法依赖的环境属性（按 trace/hook 确认）
    navigator: { userAgent: '<UA>' },
    Date: { now: () => <固定时间戳> },  // 调试时用固定值
    Math: Math,
    parseInt: parseInt,
    String: String,
    Array: Array,
    JSON: JSON,
    console: { log: () => {} },  // 静默
};

// 如果算法用了 CryptoJS，需要提供
sandbox.CryptoJS = require('crypto-js');

vm.createContext(sandbox);

// 加载算法 JS
const algorithmCode = require('fs').readFileSync('./case/js/extracted/sign_algorithm.js', 'utf8');
vm.runInContext(algorithmCode, sandbox);

// 调用签名函数
function generateSign(params) {
    return vm.runInContext(`signFunction(${JSON.stringify(params)})`, sandbox);
}
```

#### 3.2 中等量 sandbox（算法 patch 原型方法，需 DOM stub）

适用于：目标 JS 会 patch `Element.prototype` / `XMLHttpRequest.prototype` 等原型方法，需要提供对应构造函数 stub。

触发信号：
- 目标 JS 含 `Element.prototype.addEventListener = ...` 或类似原型 patch
- 目标 JS 含 `XMLHttpRequest.prototype.open = ...` 拦截器逻辑
- 目标 JS 含 `document.cookie` 读写（签名值通过 cookie 传递）

```javascript
const vm = require('vm');
const fs = require('fs');

// 中等量 sandbox：基础全局 + 浏览器 DOM stub
const sandbox = {
    // 基础全局（同 3.1）
    Date, Math, parseInt, parseFloat, String, Array, Object, Number, Boolean,
    JSON, Error, TypeError, RegExp, Promise, Map, Set, Symbol, Reflect, Proxy,
    encodeURIComponent, decodeURIComponent, encodeURI, decodeURI,
    setTimeout, setInterval, clearTimeout, clearInterval,
    isNaN, isFinite, undefined, NaN, Infinity, console,
    btoa: (s) => Buffer.from(s, 'binary').toString('base64'),
    atob: (s) => Buffer.from(s, 'base64').toString('binary'),

    // 浏览器 DOM stub（按目标 JS 实际访问补齐）
    document: { /* cookie getter/setter + createElement + ... */ },
    navigator: { /* userAgent + platform + plugins + ... */ },
    location: { /* href + origin + pathname + ... */ },
    screen: { /* width + height + ... */ },
    XMLHttpRequest: function() { /* 构造函数 + prototype */ },
    Element: function() { /* 构造函数 + prototype（若目标 JS patch Element.prototype） */ },
    fetch: () => Promise.resolve({ /* stub */ }),
    Headers: class { /* stub */ },
    localStorage: { /* getItem/setItem/... */ },

    innerWidth: 1920, innerHeight: 1080,
    devicePixelRatio: 1,
    getComputedStyle: () => ({}),
    matchMedia: () => ({ matches: false, addListener(){}, removeListener(){} }),
    addEventListener: () => {}, removeEventListener: () => {},
};
sandbox.window = sandbox;
sandbox.self = sandbox;
sandbox.top = sandbox;
sandbox.parent = sandbox;
sandbox.frames = sandbox;
sandbox.global = sandbox;

vm.createContext(sandbox);
vm.runInContext(fs.readFileSync('./target.js', 'utf8'), sandbox, { timeout: 5000 });

// 读取签名输出（cookie / 全局变量 / 返回值）
const sign = sandbox.document.cookie;  // 或 sandbox.someGlobalVar
```

**关键要点**：
- `Element` / `Document` 只需构造函数 + prototype stub，**不需要完整原型链**（EventTarget → Node → Element → ...），这是 vm 沙箱与补环境的边界
- `document.cookie` 必须用 `Object.defineProperty` 实现 getter/setter，目标 JS 通过 setCookie 写入签名值
- 不需要 NativeProtect（目标 JS 通常不做 toString 检测；若做则升级补环境）

### IMPLEMENT：验证

```
1. 用浏览器样本的相同输入调用 generateSign
2. 对比输出是否一致
3. 不一致 → 检查 sandbox 缺失的依赖（hook_function trace 确认）
4. 一致 → ≥5 次请求验证稳定性
```

### IMPLEMENT 补充：常见陷阱

| 陷阱 | 现象 | 解决 |
|------|------|------|
| **try-catch 静默吞错** | vm.runInContext 运行成功但签名未生成 | Grep 目标 JS 的 `try{...}catch(...){return ...}`，字符串替换透明化后重新运行（详见 common-pitfalls.md 反模式 8） |
| **setInterval 阻止退出** | 签名生成后进程挂起 | 测试入口添加 `process.exit(0)` |
| **Element 缺失** | `ReferenceError: Element is not defined` | sandbox 添加 Element 构造函数 + prototype stub |
| **navigator 属性不全** | `TypeError: navigator[r(...)] is not a function` | 用 Proxy 拦截 navigator 所有属性访问，精确发现缺失项 |

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | sandbox 缺失依赖 | vm 执行报 `xxx is not defined` | 用 hook_function trace 确认算法读取的全局变量，逐项补到 sandbox |
| 2 | 自定义 MD5 误当标准 MD5 | 签名长度 32 位但值不对 | 同一输入对比标准 MD5，不一致则为自定义实现 |
| 3 | 算法依赖 Date.now() | 每次签名不同，无法对比 | 调试时用固定时间戳，验证通过后改回 Date.now() |
| 4 | CryptoJS 版本差异 | vm 中 CryptoJS 输出与浏览器不一致 | 确认浏览器用的 CryptoJS 版本（3.1.2 / 4.0.0），npm 安装对应版本 |
| 5 | 算法含 setTimeout 异步 | vm 同步执行拿不到结果 | 改用 Promise + vm 微任务，或重构为同步 |
| 6 | try-catch 静默吞错 | 运行成功但输出未生成 | 透明化目标 JS 内部 try-catch，暴露真实错误（详见 common-pitfalls.md 反模式 8） |
| 7 | setInterval 阻止退出 | 签名生成后进程挂起 | 测试入口 `process.exit(0)` |
| 8 | Element/Document 缺失 | `ReferenceError: Element is not defined` | sandbox 添加构造函数 + prototype stub（目标 JS patch 原型方法时需要） |

## 边界判断

```
算法提取后能否用标准 crypto 库复现？
  ├─ 能 → 纯算还原（走 trace-flow.md）
  └─ 不能
      │
      ├─ 算法 JS 能否在最小 sandbox 中执行（不需要 document/window/navigator.* 指纹）？
      │   ├─ 能 → vm 沙箱（本案例）
      │   └─ 不能（需要完整浏览器环境 / JSVMP）→ 补环境
      │
      └─ 是否是 JSVMP（200KB+ / while-switch / 字节码数组）？
          ├─ 是 → 补环境路径 D
          └─ 否 → vm 沙箱路径 B
```

### sandbox 量级判断

不是只有"最小 sandbox"一种形态。按目标 JS 对浏览器环境的依赖程度分两档：

| 量级 | 触发信号 | sandbox 内容 | 与补环境边界 |
|------|---------|-------------|-----------|
| **基础** | 目标 JS 只依赖 Date/Math/navigator.userAgent | 基础全局变量 | 远离补环境 |
| **中等** | 目标 JS patch Element.prototype / XMLHttpRequest.prototype / 读写 document.cookie | + DOM 构造函数 stub + cookie getter/setter | 接近补环境但不需要完整原型链和 NativeProtect |

**升级补环境信号**：目标 JS 做 `Function.prototype.toString` 检测 / `document.all` 检测 / 完整原型链 instanceof 检测 → 需要 NativeProtect 和完整原型链 → 升级补环境。

## 可验证事实清单（经验资产）

1. [签名参数名] 长度 [N] 字符
2. 算法类型：[自定义 MD5 / 混淆 AES / 自定义哈希]
3. 算法依赖的环境属性：[UA / 时间戳 / 随机数]
4. vm sandbox 需要提供的全局变量：[CryptoJS / Math / parseInt]
5. 签名输入：[参数排序规则 / 拼接格式]
6. ≥5 次请求签名稳定通过

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | 统一日志驱动逆向流程 |
| `references/workflow/decision-tree.md` | 题型判定边界 |
| `references/env/runtime-frameworks.md` | 升级补环境判断（何时需 jsdom/sdenv） |
| `references/env/env-debug-loop.md` | 静默吞错诊断 + setInterval 退出陷阱 |
| `references/workflow/common-pitfalls.md` | 反模式 8（try-catch 静默吞错） |
| `cases/vm-sandbox-chameleon-iwencai.md` | 中等量 sandbox 实战案例（同花顺 chameleon.js） |
| `templates/vm-sandbox/` | vm 沙箱交付模板 |
