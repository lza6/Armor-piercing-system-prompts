# Case：快手 __NS_hxfalcon + kww 签名（Jose 模块 + kwpsec JSVMP）

> 难度：★★★★
> 还原方案：A 纯算还原（__NS_hxfalcon / kww SSR fallback）+ D 环境伪装（kww 浏览器端，黑盒调用 SDK）
> 实现语言：Node.js / Python
> 最后验证日期：2026-07-12
> 平台类型：快手 web 端（kuaishou.com，feed/hot 等接口）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] 请求参数 `__NS_hxfalcon`（URL query），header `kww`
- [x] webpack 打包，含 Jose 模块（IIFE，嵌在 ~678KB 的 `index-*.js` 中，含 `$encode` / `$getCatVersion` 方法）
- [x] `kwpsec` SDK = JSVMP 字节码引擎（Brook VM + Mihawk VM），生成 `kww`
- [x] 存在 `/s/w/c` 加密接口（AES-CBC-Pkcs7）

### 参数特征
- [x] `__NS_hxfalcon`：格式 `HUDR_xxx$HE_xxx`（HUDR 设备指纹 + HE 90 位哈希签名）
- [x] `kww` header：浏览器端来自 `kwpsec.getData()`；SSR fallback 格式 `AES-128-CBC-Pkcs7(ts|random8, key) + "###ssrd"`
- [x] `caver` 参数：`Jose.call('$getCatVersion')` → `2`

### 请求特征
- [x] 目标接口 `/rest/v/feed/hot` 等
- [x] `kww = localStorage.getItem('kwfv1') = kwpsec.getData()`
- [x] 无有效签名返回风控/验证码；HTTP 200 + risk-control 头 = 签名通过但被 IP 风控

### 反调试特征
- [x] Jose 内部 realm 隔离：`vm.createContext` 沙箱会因 realm 隔离失败（`Property 'jmpOnw_ms' ... is not a function`）
- [x] Node.js 全局 `navigator` 只读，`Object.assign(globalThis, ...)` 失败

---

## 加密方案

- **路径**：A 纯算还原（__NS_hxfalcon + kww SSR fallback）+ D 环境伪装（kww 浏览器端需完整浏览器，黑盒调用 SDK）
- **框架**：Node.js `eval`（全局环境运行 Jose，避开 realm 隔离）/ vm（不可行，见踩坑 2）
- **TLS 客户端**：Node.js 原生 https（验证阶段被 IP 风控阻挡，非签名问题）
- **核心思路**：
  - __NS_hxfalcon：提取 Jose 模块后在 Node 全局 `eval` 运行，调用 `Jose.call('$encode', [{url, query, form, requestBody}, {suc, err}])`
  - kww：优先用 SSR fallback `ot$1()`（AES-CBC）替代 JSVMP 浏览器端生成
  - /s/w/c：`AES-CBC-Pkcs7`，key=iv=`webweaponconfigs`

### 算法细节

**__NS_hxfalcon = `Jose.call('$encode', [{url, query, form, requestBody}, {suc, err}])`**
- Jose 由 webpack 模块加载器注册，含 HUDR（设备指纹）+ HE（哈希签名）两部分
- HE 段可纯算法还原（流式异或 + BLAKE2s）；HUDR 段为固定字段加密结果；整条可由本地生成

**kww（SSR fallback） = `AES-128-CBC-Pkcs7(ts|random8, "K8wm5PvY9nX7qJc2") + "###ssrd"`**
- 浏览器端 `kww = kwpsec.getData()` 是 JSVMP 字节码引擎，纯算还原难度极大，优先用 SSR fallback

**/s/w/c 接口加密** = `AES-CBC-Pkcs7`，key=iv=`webweaponconfigs`

**caver** = `Jose.call('$getCatVersion')` → `2`

---

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | **Jose 模块提取** | Jose 是 webpack IIFE，嵌在 678KB 压缩 JS 中，难以定位 | 用括号匹配算法（跳过字符串/注释）提取 `@418047`→`@460035`，共 41988 字节 |
| 2 | **Jose 在 vm 沙箱运行失败** | `Property 'jmpOnw_ms' of object #<Function> is not a function`；realm 隔离导致 `jmpOnw_*` 注册到外层 Object 但内部用自己的 Object | 放弃 vm，改用 `eval(joseCode)` 在 Node 全局环境运行，realm.global.Object 即全局 Object |
| 3 | **Node.js 补环境 navigator 只读** | `Object.assign(globalThis, {...})` 失败 | 用 `Object.defineProperty(globalThis, k, {value, writable:true, configurable:true})` 逐个设置 |
| 4 | **kww JSVMP 难还原** | `kwpsec` = Brook VM + Mihawk VM 字节码引擎，纯算还原难度极大 | 定位 SSR fallback `ot$1()`（AES-CBC）替代，格式 `AES-128-CBC-Pkcs7(ts|random8, key) + "###ssrd"` |
| 5 | **IP 风控误判为签名错误** | API 返回 400002 验证码，误以为签名错 | 排查：Node 请求 / 浏览器 Jose+fetch / 浏览器正常请求均 400002 → IP 被风控；HTTP 200 + risk-control 头 = 签名通过但被风控 |

---

## 可验证事实清单（经验资产）

1. `__NS_hxfalcon` 是 URL query 参数，`kww` 是 header
2. Jose 模块含 `$encode` / `$getCatVersion` 方法
3. `caver = Jose.call('$getCatVersion')` → `2`
4. `kww = localStorage.getItem('kwfv1') = kwpsec.getData()`
5. kww SSR fallback = `AES-128-CBC-Pkcs7(ts|random8, "K8wm5PvY9nX7qJc2") + "###ssrd"`
6. `/s/w/c` 接口加密 = AES-CBC-Pkcs7，key=iv=`webweaponconfigs`
7. Jose 内 realm 隔离：`vm.createContext` 沙箱不可行，必须 `eval` 全局运行
8. Node.js `navigator` 只读，补环境须用 `Object.defineProperty`
9. webpack 模块提取：括号匹配（跳过字符串/注释）最可靠
10. HTTP 200 + risk-control 头 = 签名通过但被 IP 风控（非签名问题）
11. kww 浏览器端 = JSVMP 字节码引擎（Brook VM + Mihawk VM），需完整浏览器环境
12. 验证阶段被 IP 风控阻挡（400002），签名逻辑本身已还原，非签名错误

---

## 适用 / 不适用场景

- 适用：快手 web 端 API 签名（`/rest/v/feed/hot` 等，需 __NS_hxfalcon + kww）
- 不适用：快手 App 端（不同签名方案）、快手直播接口（可能不同）、需 kwpsec 真实指纹的场景（需黑盒调用 SDK）

---

## 工具链使用经验

### ruyipage（Python Firefox 自动化）
- `add_preload_script`：在 document-start 阶段注入 hook
- `page.run_js`：执行 JS，return 必须在函数体内（用 IIFE 包裹）
- `page.capture.start(targets=True, collect_bodies=True)`：捕获网络请求（抓全部包，事后过滤，避免漏 JS）
- `page.capture.steps`：遍历捕获的数据包

### RuyiTrace（Firefox 内核日志）
- NDJSON 格式，可能包含 null 字符需处理
- JSVMP 分析日志量过大时，优先用 ruyipage 的 hook 方式

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/phase-flow.md` | CLEANUP 经验沉淀 |
| `references/env/env-native-protection.md` | realm / toString 检测绕过 |
| `cases/jsvmp-dual-sign-purealgo-vm-xiaohongshu.md` | JSVMP 还原对照 |
