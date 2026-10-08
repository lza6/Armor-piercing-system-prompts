# Case：Gitee nox_jst_v1 + tox_token 签名（百度 WAF 三件套 JSVMP）

> 难度：★★★★
> 还原方案：D 环境伪装（nox + tox 均用 vm 沙箱补环境，不反编译字节码）
> 实现语言：Node.js
> 最后验证日期：2026-07-16
> 平台类型：Gitee web 端（gitee.com，/signup → /check 注册校验接口）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] 三件套协调器：`gangplank_20251103.js`（bantiOrigin: `https://wafbotsr.baidu.com`，sak: `a03fed3c32`）
- [x] cookie 签名 SDK：`nox_20250820.js`（约 315KB，JSVMP 字节码引擎）
- [x] 请求参数签名 SDK：`tox_20250820.js`（约 391KB，JSVMP 字节码引擎，30+ VM 实例）
- [x] JSVMP 字符串编码：charCode 数组（如 `[110,97,118,105,103,97,116,111,114]` = "navigator"）
- [x] VM 间通信：`bindingIdentifier(name, callback)` 导出 + `_$vmp_get_*` 回调获取
- [x] 暴露 API：`window.resetNoxJstV1()` / `window.Tox.getToken()`

### 参数特征
- [x] `nox_jst_v1`（cookie）：格式 `2.0_<4位hex>_<base64>`，长度约 245 字节
- [x] `tox_token`（URL query）：格式 `v2_<8位hex>_mejd42mp_<base64变体>`，长度固定 86 字节
- [x] `mejd42mp` 是 tox_token 固定标识符（同站稳定）
- [x] tox_token 每次调用 `Tox.getToken()` 生成不同值（含时间戳/随机数）

### 请求特征
- [x] WAF 注入：`/signup` 页面响应含 `window.__toxCfg` + `<script src="/sd5prgymvjlf4cklsqkz91do2mhorb/static/wb/2.0/tox_20250820.js">`
- [x] 表单提交劫持：`HTMLFormElement.prototype.submit` 被改写，自动注入 `tox_token`
- [x] `/check?tox_token=...` 接口：POST + body `do=phone&val=<手机号>&entrance=register`，返回 `1` 表示手机号可用
- [x] WAF 只校验 nox_jst_v1 + tox_token，不需要 gitee-session-n / X-CSRF-Token（业务层独立校验）

### 反调试特征
- [x] tox JS 内部 try-catch 容错：访问 `__bpf__` 等 undefined 属性会被捕获，不影响 token 生成
- [x] VM 28 通过 `_$vmp_get_t` 回调获取 window 对象，访问 `window.eval` 获取 `Eval` 构造器

---

## 加密方案

- **路径**：D 环境伪装（Skill 路径 D，补环境方案，不反编译字节码）
- **框架**：Node.js `vm.createContext` + `vm.runInContext`（构造函数补环境 + Object.create 实例）
- **TLS 客户端**：Node.js 原生 https（5/5 真实 API 验证通过）
- **核心思路**：
  - nox_jst_v1：补环境加载 nox JS → 调用 `window.resetNoxJstV1()` 手动触发 → 通过 `document.cookie` setter 收集
  - tox_token：补环境加载 tox JS → 等待约 2 秒异步初始化 → 调用 `window.Tox.getToken()`

### 算法细节

**nox_jst_v1 = `window.resetNoxJstV1()` → cookie setter 收集**
- nox JS 不会自动写 cookie，必须手动调用 `resetNoxJstV1()`
- 输出格式：`2.0_<4位hex>_<base64>`（约 245 字节）

**tox_token = `window.Tox.getToken()`**
- 加载后等待约 2 秒异步初始化完成
- 输出格式：`v2_<8位hex>_mejd42mp_<base64变体>`（固定 86 字节）
- `mejd42mp` 是固定标识符

---

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | **VM 28 的 `_$vmp_get_t` 回调获取 window 对象** | tox JS 通过 `$.bindingIdentifier("c", cb)` 把 window 作为 `G` 变量导出，VM 28 通过 `_$vmp_get_t:function(){return G}` 获取 | 修改回调注入日志，确认 G 的值是 window 对象及其 keys |
| 2 | **window.eval 缺失导致 VM 28 崩溃** | VM 28 获取 window 后访问 `window["eval"]` 获取 eval 函数，再用 `eval["Eval"]` 获取 Eval 构造器；补环境 window 缺少 eval 属性 → `Cannot read properties of undefined (reading 'Eval')` | 在 context 中显式提供 `eval: eval`（关键修复） |
| 3 | **tox 异步初始化未完成** | 加载后立即调用 `Tox.getToken()` 返回 500 | 添加 setTimeout 等待约 2 秒 |
| 4 | **nox_jst_v1 不自动生成** | 加载 nox JS 后 cookie jar 为空 | 手动调用 `window.resetNoxJstV1()` 触发 |
| 5 | **构造函数补环境是关键** | 直接用对象字面量做 window/navigator，`instanceof` 检查失败 | 用 `function Window() {}` + `Object.create(Window.prototype)` 创建实例，让 prototype chain 正确 |
| 6 | **`__bpf__` 属性访问 undefined** | tox VM 访问 window 的 `__bpf__` 等属性报错 | VM 内部 try-catch 捕获，不影响 token 生成（可忽略） |
| 7 | **JSVMP 调试方法** | 不知道哪个 VM 的哪条指令出错 | 修改 `k[45]` 指令处理器注入 undefined 检测日志，精确定位 bid=28 Z=18 op=45 |
| 8 | **GET /signup 返回 405** | 验证脚本想先 GET /signup 拿 session+CSRF | WAF 只校验 nox_jst_v1 + tox_token，不需要 session/CSRF，直接 POST /check 即可 |
| 9 | **GET /check 返回 HTML 而非 JSON** | 用 GET 请求 /check 返回注册页 HTML（含 __toxCfg 注入） | 真正业务接口是 POST /check + body `do=phone&val=...&entrance=register`，返回 `1` |

---

## 可验证事实清单（经验资产）

1. 百度 WAF 三件套：Banti SDK（协调器）+ nox（cookie 签名）+ tox（请求参数签名）
2. `nox_jst_v1` 是 cookie 参数，格式 `2.0_<4位hex>_<base64>`，长度约 245 字节
3. `tox_token` 是 URL query 参数，格式 `v2_<8位hex>_mejd42mp_<base64变体>`，长度固定 86 字节
4. `mejd42mp` 是 tox_token 固定标识符
5. nox JS 必须手动调用 `window.resetNoxJstV1()` 才会生成 cookie
6. tox JS 加载后需等待约 2 秒异步初始化，再调用 `window.Tox.getToken()`
7. tox JS 暴露 `window.Tox` 对象，含 `getToken()` / `isCaptchaResponse(resp, ...)` / `getCaptchaToken(...)` 方法
8. 补环境必须提供 `eval: eval` 到 context（tox VM 28 通过 `window.eval` 获取 Eval 构造器）
9. 构造函数补环境：`function Window() {}` + `Object.create(Window.prototype)` 让 `instanceof` 通过
10. WAF 只校验 nox_jst_v1 + tox_token，不需要 gitee-session-n / X-CSRF-Token
11. `/check` 业务接口是 POST + body `do=phone&val=<手机号>&entrance=register`，返回 `1` 表示手机号可用
12. tox JS 指令处理器：`k[3]`=设置G寄存器, `k[5]`=读常量, `k[44]`=读scope变量, `k[45]`=属性访问, `k[46/47]`=方法调用
13. VM 间通信：`bindingIdentifier(name, callback)` 导出 + `_$vmp_get_*` 回调获取
14. JSVMP 调试方法：修改指令处理器（如 k[45]）注入日志，精确定位出错 VM 和指令
15. 5 次端到端 API 验证全部返回 `1`（POST /check + 项目生成的 nox_jst_v1 + tox_token）

---

## 适用 / 不适用场景

- 适用：Gitee web 端 WAF 签名（`/signup` → `/check` 注册流程，需 nox_jst_v1 + tox_token）
- 适用：百度 WAF 同款三件套（Banti + nox + tox）的其他站点（补环境方案可复用）
- 不适用：Gitee App 端（不同签名方案）、需要 tox JSVMP 字节码反编译的场景（本项目用补环境方案）

---

## 工具链使用经验

### ruyipage（取证）
- `forensic_ruyipage.py` 通用脚本抓包：捕获 `/signup` 页面响应中的 WAF 注入标记（`__toxCfg` + tox/nox script 标签）
- `target-hits.json` 含目标命中详情，含完整响应体

### RuyiTrace（调试）
- NDJSON 格式日志，10 个 trace 文件
- JSVMP 分析日志量过大，优先用指令处理器注入日志方式调试

### JSVMP 指令级调试（核心方法）
- 修改 `k[45]` 指令处理器：`k[45]=function(n,i){var e=i.G[n[1]].get(),r=i.X.get();if(e===undefined||e===null){console.error("[K45 ERR bid=... r=... e=...]")};i.X=new t(e[r])}`
- 修改 VM `_$vmp_get_*` 回调：记录返回值的类型和 keys
- 修改 `run` 方法指令读取点：按 bid 过滤追踪特定 VM 的指令执行序列

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/phase-flow.md` | CLEANUP 经验沉淀 |
| `cases/jsvmp-dual-sign-purealgo-vm-xiaohongshu.md` | JSVMP 还原对照（小红书用纯算+vm，本项目用补环境） |
| `cases/kuaishou-hxfalcon-kww-reverse.md` | JSVMP + 多参数签名对照（快手用 eval 全局运行，本项目用 vm 沙箱） |
