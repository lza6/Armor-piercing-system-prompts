# Case：bundle.js 常驻加载 + bdms.init + XHR patch 截获 a_bogus（抖音）

> 难度：★★★★
> 还原方案：D 环境伪装（bundle.js 常驻 + XHR patch）
> 实现语言：Node.js
> 最后验证日期：2026-07-08
> 平台类型：抖音（douyin.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] 三层 SDK 联动：`sdk-glue.js` → `bdms.js` → `webmssdk.es5.js`
- [x] `bdms.init()` 异步回调注册 XHR 拦截器，匹配 `bdms.paths` 正则的 URL 追加 `a_bogus`
- [x] `signUrl(config)` 同步触发签名：内部 `xhr.send()` → bdms hooked send `z[107]` → 生成 a_bogus
- [x] JSVMP 字节码执行（`_$webrt_` 解释器），但本案例不 trace 字节码，而是让 SDK 原样运行截获输出
- [x] `runtime_bundler_34` / `sdk-glue` / `bdms` 会覆盖 `XMLHttpRequest.prototype`，XHR patch 必须在它们之后执行

### 参数特征
- [x] `a_bogus`：约 180~192 字符，Base64 变体编码
- [x] `msToken`：约 140 字符，来自 mssdk.bytedance.com 请求的响应
- [x] 签名生成顺序：先获取 msToken 加到 URL → 再计算 a_bogus

### 请求特征
- [x] mssdk.bytedance.com 请求用于获取 msToken（SDK 发起，必须真发）
- [x] 非 mssdk 的 https:// 请求不能真发（否则 signUrl 的 xhr.send 被绕过，a_bogus 为 null）
- [x] IP 限流时返回 HTTP 200 空 body + `x-vc-bdturing-parameters` 响应头（code=10000, type=verify, subtype=slide）

### 反调试特征
- [x] JSVMP 字节码（CALL 指令从栈取 undefined 函数时 crash，发生在 mon 请求 send 之前）
- [x] bdms.init() 异步回调可能引用 undefined 值导致 crash
- [x] 环境检测：Function.prototype.toString 原生性、navigator.webdriver 等

### 混淆类型
- [x] JSVMP（字节码虚拟机），但本案例采用"让 SDK 原样运行"而非"trace 字节码"策略

---

## 加密方案

- **路径**：D 环境伪装（bundle.js 常驻 + XHR patch 喂入-截出）
- **框架**：vm（Node.js 原生 vm，非 jsdom）
- **TLS 客户端**：Node.js 原生 https（OpenSSL TLS 指纹）
- **核心思路**：构建 bundle.js（含 SDK 全部源码 + 环境补丁）→ 常驻加载 → `signUrl(url)` 同步触发签名 → XHR patch 截获 a_bogus

### 算法细节

**a_bogus** 由 JSVMP 字节码计算，算法不可直接提取。本案例不 trace 字节码，而是：

1. 将抖音官方 SDK 4 个源文件（sdk-glue / bdms / webmssdk / runtime_bundler）构建为 bundle.js
2. 在 bundle.js 中注入环境补丁（window / document / navigator / XMLHttpRequest 等）
3. 常驻加载 bundle.js，执行 `bdms.init()` 注册拦截器
4. 调用 `signUrl(config)` 触发内部 `xhr.send()` → bdms hooked send 生成 a_bogus
5. XHR patch 拦截 mssdk.bytedance.com 请求真发获取 msToken，拦截其他请求避免绕过签名

**签名公式**：无法提取（JSVMP）。策略 = 让 SDK 原样运行 + XHR patch 截获输出。

---

## 方案方向

与现有字节系案例（jsvmp-xhr-interceptor-env-emulation）的区别：

| 维度 | jsvmp-xhr-interceptor-env-emulation | 本案例 |
|------|-------------------------------------|--------|
| 框架 | jsdom 全量环境伪装 | vm + 手写环境补丁（bundle.js） |
| 策略 | jsdom 模拟浏览器环境 → JSVMP 运行 | 构建产物 bundle.js 常驻 → SDK 原样运行 |
| XHR | jsdom 内置 XHR | 手写 XHR patch（只真发 mssdk 请求） |
| 常驻 | 按需调用 | 常驻模式 + uncaughtException 兜底 |
| crash 处理 | 未提及 | process.on('uncaughtException') 捕获异步 crash |

本案例更适合 API 服务场景（常驻 signer），jsvmp-xhr-interceptor 更适合一次性脚本场景。

## 标准流程

### FORENSIC_CAPTURE → TRACE_CAPTURE：定位 + SDK 提取

```
1. 定位三层 SDK：sdk-glue.js（100KB）→ bdms.js（147KB）→ webmssdk.es5.js（387KB）
2. 确认 bdms.init() 注册 XHR 拦截器，匹配 bdms.paths 正则的 URL 追加 a_bogus
3. 确认 signUrl(config) 同步触发签名：内部 xhr.send() → bdms hooked send z[107]
4. 确认 msToken 来源：mssdk.bytedance.com 请求的响应
5. 保存 SDK 源文件到 platforms/douyin/sdk/（4个文件）
6. 编写 profile.js 声明构建配置
7. 构建产物 bundle.js（final.js 唯一加载的文件）
```

### TRACE_ANALYZE：环境补丁 + 常驻加载

```javascript
// bundle.js 结构
// 1. 环境补丁（window/document/navigator/XMLHttpRequest 等）
// 2. SDK 源码（sdk-glue + bdms + webmssdk + runtime_bundler）
// 3. XHR patch（在所有脚本加载完成后执行）

// XHR patch 关键逻辑
const originalSend = XMLHttpRequest.prototype.send;
XMLHttpRequest.prototype.send = function(body) {
  const url = this._url;
  // 只真发 mssdk.bytedance.com 请求（获取 msToken）
  if (url.includes('mssdk.bytedance.com')) {
    return originalSend.call(this, body);
  }
  // 其他请求不真发（避免 signUrl 的 xhr.send 被绕过）
  // 让 bdms hooked send z[107] 正常执行生成 a_bogus
};

// 常驻模式：捕获异步 crash 不影响 signUrl 同步执行
process.on('uncaughtException', (err) => {
  // bdms.init() 异步回调 crash 不影响 signUrl
  console.error('uncaughtException:', err.message);
});
```

```javascript
// final.js 结构（API 服务入口）
const { signUrl } = require('./bundle.js');

// 常驻 signer 单例
const signer = {
  sign(url) {
    return signUrl({ url });
  }
};

// signUrl 同步触发：xhr.send → bdms hooked send → a_bogus
const aBogus = signer.sign(targetUrl);
```

### IMPLEMENT：验证

```
1. signUrl 返回 a_bogus（非 null），长度 180-192 字符
2. 用 a_bogus 请求抖音业务接口，返回 HTTP 200 + 正确数据
3. 连续 5 次签名稳定，a_bogus 每次不同（含时间因子）
4. IP 限流时识别 x-vc-bdturing-parameters 响应头，等待解封重测
```

## 🚫 禁动清单（实战踩过的"不要碰"）

| # | 禁动 | 原因 |
|---|------|------|
| 1 | 不要让所有 https:// 请求真发 | 会导致 signUrl 触发签名的 xhr.send 被绕过（走自定义 https.request 而非 bdms hooked send z[107]），a_bogus 为 null |
| 2 | 不要在 SDK 加载前执行 XHR patch | runtime_bundler_34 / sdk-glue / bdms 会覆盖 XMLHttpRequest.prototype，patch 必须在所有脚本加载完成后执行 |
| 3 | 不要忽略 bdms.init() 异步 crash | crash 发生在 mon 请求 send 之前（与请求是否真发无关），只能靠 uncaughtException 兜底 |
| 4 | 不要用 Python/curl_cffi | 项目约束：纯 Node.js 运行时 |
| 5 | 不要自定义 /parse 接口的 UA | UA 固定为 Chrome 135，确保签名与请求 UA 一致 |

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | 所有 https:// 真发 | a_bogus 为 null | XHR patch 只真发 mssdk.bytedance.com 请求，其他不真发 |
| 2 | XHR patch 被 SDK 覆盖 | 拦截失效，a_bogus 为 null | patch 在所有脚本加载完成后执行 |
| 3 | bdms.init() 异步 crash | 进程崩溃，signUrl 无法执行 | process.on('uncaughtException') 捕获，不影响同步签名 |
| 4 | msToken 拿不到 | SDK 请求不发导致无响应 | 让 mssdk 请求真发即可 |
| 5 | IP 限流 | HTTP 200 空 body + x-vc-bdturing-parameters 头 | 等待 IP 解封后重测 |
| 6 | JSVMP Error stack 为空 | 调用栈显示 Function patch 创建的函数 → _0x1233dd → fdFQs | 检查 env/core.js 中 Function.prototype.toString 和 Error 处理 |

## crash vs msToken 本质区别

| 问题 | 根因 | 发生时机 | 解决方法 |
|------|------|---------|---------|
| msToken 缺失 | SDK 请求不发导致拿不到响应 | 请求阶段 | 让 mssdk 请求真发 |
| crash | JSVMP CALL 指令从栈取 undefined 函数 | mon 请求 send 之前 | uncaughtException 兜底 |

**关键**：crash 与请求是否真发无关，是 JSVMP 执行字节码的内部问题，只能靠 uncaughtException 兜底。

## 边界判断

```
JSVMP 字节码能否在最小 sandbox 中执行？
  ├─ 能（无环境检测）→ vm 沙箱
  └─ 不能（需完整环境 + bdms.init 注册拦截器）→ 环境伪装（本案例）

JSVMP 的 a_bogus 能否纯算还原？
  ├─ 能 → 纯算还原
  └─ 不能（字节码保护）→ 环境伪装
```

## 可验证事实清单（经验资产）

1. a_bogus 长度 180~192 字符，Base64 变体编码
2. msToken 长度约 140 字符，来自 mssdk.bytedance.com 请求响应
3. 签名顺序：先获取 msToken 加到 URL → 再计算 a_bogus
4. signUrl 同步触发签名，返回 a_bogus（非 null 表示成功）
5. XHR patch 只真发 mssdk.bytedance.com 请求，其他不真发
6. bdms.init() 异步 crash 不影响 signUrl 同步执行
7. IP 限流返回 HTTP 200 空 body + x-vc-bdturing-parameters 头（code=10000）
8. UA 固定 Chrome 135，签名与请求 UA 必须一致
9. bundle.js 是构建产物，final.js 唯一加载的文件
10. /parse 接口支持用户自备 cookie（优先级：用户传入 > 内置 DEVICE_COOKIE）
11. /parse 双路径：默认用 DEVICE_COOKIE（快速），失败 fallback 访问主页刷新 cookie（保险）
12. ≥5 次签名稳定通过

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | 标准流程 |
| `references/workflow/decision-tree.md` | 题型判定边界 |
| `references/env/env-debug-loop.md` | 环境补丁调试循环 |
| `references/workflow/common-pitfalls.md` | 反模式 7（signer 无状态并发） |
| `cases/jsvmp-xhr-interceptor-env-emulation.md` | 同站 jsdom 方案对比 |
| `cases/jsvmp-dual-sign-xhr-intercept-cacheOpts-jsdom-firefox.md` | 字节系双签名方案 |
| `references/workflow/experience-rules.md` | 规则 7（signer 单例常驻） |
