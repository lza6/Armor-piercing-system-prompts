# Case：vm 沙箱 JSVMP + 纯算双轨签名（小红书 X-s / X-s-common）

> 难度：★★★★
> 还原方案：A 纯算还原（X-S-Common）+ B vm 沙箱执行（X-s）
> 实现语言：Node.js
> 最后验证日期：2026-07-12
> 平台类型：小红书（xiaohongshu.com / edith.xiaohongshu.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] 文件名 `as-v2-ds.js`（本地 SDK，~60KB）+ `as-v2-fp.js`（指纹脚本，~390KB）
- [x] JSVMP 字节码解释器（while-switch 循环 + 状态机）
- [x] SDK 导出函数名形如 `_1619d69735e1d480a72d7e01c4a40b7f`（32 位 hex MD5 命名）
- [x] 自定义 Base64 字母表：`ZmserbBoHQtNP+wOcza/LpngG8yJq42KWYj0DSfdikx3VT16IlUAFM97hECvuRX5`（非标准）
- [x] 修改版 CRC32：与标准 CRC32 差异为返回值异或 `0xedb88320`，结果为有符号 32 位整数
- [x] 业务层签名入口函数名 `seccore_signv2(e, a)`（在 vendor-dynamic.*.js 中）
- [x] `as-v2-fp.js` 中有 `_0x19467f` 函数（JSVMP bytecode handler），对 `_sabo_*` 对象返回 undefined/null

### 参数特征
- [x] header `X-s`：`XYS_` 前缀 + 自定义 Base64（UTF8 编码的 JSON）
- [x] header `X-s-common`：纯自定义 Base64（UTF8 编码的 JSON，含 s0/s1/x0..x12 共 15 字段）
- [x] header `X-t`：13 位毫秒时间戳字符串
- [x] X-s 内 x3 字段：`mns0101_` / `mns0201_` / `mns0301_` 前缀 + 自定义 Base64 字节流
- [x] X-s-common 内 x12 字段格式：`<localStorageDsllt>;<windowDsl>`（如 `null;1780544705228`）
- [x] X-s-common 内 x5 字段 = cookie `a1` 的值

### 请求特征
- [x] 业务 API 域名 `edith.xiaohongshu.com`
- [x] 笔记/评论 API 路径含 `xsec_token` query 参数（从笔记页 URL 获取）
- [x] 无签名返回 461/412；签名正确但无登录态返回 200 + `code:-101`
- [x] 评论接口需 `web_session` cookie 才能返回评论数据
- [x] trace 浏览器无登录态时评论请求返回 `verifytype:301`（旋转验证码挑战）

### 反调试特征
- [x] `as-v2-fp.js` 加载时 `Cannot read properties of undefined (reading 'apply')` 崩溃
- [x] `_0x19467f` 对 `_sabo_*` 内部对象返回 undefined/null，调用方 `.apply()` 崩溃
- [x] `_AUuXfEG27Xa3x` 内部 JSVMP 完整性检查失败返回 `err:d93135:`

---

## 加密方案

- **路径**：A 纯算还原（X-S-Common）+ B vm 沙箱执行（X-s）双轨
- **框架**：vm（Node.js 原生 vm 模块）
- **TLS 客户端**：Node.js 原生 https
- **核心思路**：
  - X-S-Common：纯算实现（MD5 + 自定义 Base64 + UTF8 + 修改版 CRC32）
  - X-s：vm.createContext 加载 `as-v2-ds.js`，调用 `_1619d69735e1d480a72d7e01c4a40b7f(c,u,p)` 获取签名字节数组，外层用纯算包装（MD5 + 自定义 Base64）
  - **不加载 `as-v2-fp.js`**：fp.js 负责附加浏览器指纹，但服务端不校验指纹部分

### 算法细节

**X-s = `"XYS_" + customBase64(utf8(JSON.stringify({x0,x1,x2,x3,x4})))`**

其中 `x3 = mnsv2(c, u, p)`：
- `c = url + payload`（payload 为请求体，GET 请求无）
- `u = md5(c)`，`p = md5(url)`
- `mnsv2` 返回 `"mns0101_" + customBase64(_1619(c,u,p))`
- `_1619` 是 JSVMP SDK 导出的字节码加密函数，返回 number[]

**X-s-common = `customBase64(utf8(JSON.stringify({s0,s1,x0,...,x12})))`**

字段定义：
- s0:5, s1:'', x0:'1', x1:'4.3.7'(版本), x2:'Windows'(平台), x3:'xhs-pc-web', x4:'6.31.5'(app版本)
- x5: cookie a1 的值
- x6:'', x7:'' (当前版本空)
- x8: localStorage b1 的值（可 null）
- x9: `modifiedCrc32(x6 + x7 + String(x8))`
- x10: sigCount（签名计数）
- x11: 'normal'
- x12: `localStorageDsllt + ';' + windowDsl`（如 `null;1780544705228`）

**自定义 Base64**：非标准字母表 `ZmserbBoHQtNP+wOcza/LpngG8yJq42KWYj0DSfdikx3VT16IlUAFM97hECvuRX5`

**修改版 CRC32**：标准 CRC32 结果异或 `0xedb88320`，返回有符号 32 位整数

---

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | **误用 `_1619` 当 mnsv2** | x3 长度 173 字节，前缀字节 `71 a3 02`，与真实 147 字节 `83 62 43` 不符 | 真实 mnsv2 由 as-v2-fp.js 挂载（附加浏览器指纹），但**服务端不校验指纹部分**，直接用 `_1619` 输出即可通过 |
| 2 | **as-v2-fp.js 加载崩溃** | `Cannot read properties of undefined (reading 'apply')` at col 345243 | `_0x19467f` 对 `_sabo_*` 对象返回 undefined/null，调用方 `.apply()` 崩溃；**绕过方案**：不加载 fp.js，只用 ds.js |
| 3 | **mnsv2 未挂载到 window** | 加载 fp.js 成功后 `window.mnsv2` 仍 undefined | trace 显示 `mnsv2` 仅在 `Set.has("mnsv2")` 出现，无 window 赋值；可能由 JSVMP 内部机制存储；**最终不需要**（见坑 1） |
| 4 | **评论接口返回 code:-101** | 签名 5/5 通过（200 OK）但无评论数据 | `-101` = "无登录信息"，评论接口需 `web_session` cookie；补充登录态 cookie 后 5/5 成功返回评论 |
| 5 | **误判 x3 长度差异为签名失败** | 以为 173 vs 147 字节差异导致评论获取失败 | 实测：签名正确性 5/5 通过；-101 是业务层缺登录态，非签名问题 |
| 6 | **trace 浏览器也被风控** | 以为浏览器能无登录返回评论 | trace 显示浏览器评论请求返回 `verifytype:301`（旋转验证码），实际也需要登录态 |
| 7 | **X-s-common x12 字段格式** | x12 = `null;getdss_value` 还是 `Date.now();getdss_value` | 真实值 `"1783822177641;1780544705228"` = `Date.now();getdss_value`（服务端嵌入时间戳） |
| 8 | **MNS 版本前缀选择** | SDK 内 `_66062487cf103622475a2f9b17d8293e` = 'mns0301'，但评论接口实际用 'mns0101' | trace 解码评论 X-s 确认前缀 `mns0101_`；不同接口/场景可能用不同版本 |

---

## vm 沙箱环境 stub 清单

本案例只需**最小 sandbox**（比 chameleon 案例少）：

### 必须提供
| 对象 | 关键属性/方法 | 用途 |
|------|-------------|------|
| `window`/`globalThis`/`self`/`top`/`parent`/`frames` | 全部指向 sandbox 自身 | 全局上下文自引用 |
| `console`/`Date`/`Math`/`JSON`/`Array`/`String`/`Number`/`Boolean`/`RegExp`/`Symbol`/`Reflect`/`Proxy`/`Promise`/`Map`/`Set`/`WeakMap`/`WeakSet` | 标准 JS 内置 | SDK 内部逻辑 |
| `Uint8Array`/`Uint16Array`/`Uint32Array`/`Int8Array`/`Int16Array`/`Int32Array`/`Float32Array`/`Float64Array`/`ArrayBuffer`/`DataView` | TypedArray | JSVMP 字节操作 |
| `atob`/`btoa` | Buffer 实现 | Base64 解编码 |
| `document` | cookie、referrer、createElement、documentElement 等 | SDK 环境检测（不实际使用） |
| `navigator` | userAgent、platform、language 等 | SDK 环境检测 |
| `location` | href、host、pathname 等 | URL 解析 |
| `localStorage`/`sessionStorage` | getItem/setItem 等 | 存储访问 stub |
| `screen`/`history`/`performance` | 基本属性 | 环境检测 |
| `MutationObserver` | 构造函数 stub | SDK 可能引用 |
| `setTimeout`/`setInterval`/`clearTimeout`/`clearInterval` | Node.js 原生 | SDK 定时器 |
| `parseInt`/`parseFloat`/`isNaN`/`isFinite`/`encodeURIComponent`/`decodeURIComponent`/`encodeURI`/`decodeURI` | 全局函数 | SDK 内部使用 |

### 不需要
- **不需要加载 as-v2-fp.js**（指纹脚本，服务端不校验指纹）
- 不需要 Element 构造函数（ds.js 不 patch 原型）
- 不需要 XMLHttpRequest stub（ds.js 不发请求）
- 不需要 NativeProtect（ds.js 不做 toString 检测）
- 不需要 Canvas/WebGL/Audio 指纹回放

---

## 可验证事实清单（经验资产）

1. X-s header 前缀 `XYS_`，后接自定义 Base64
2. X-s-common header 无前缀，直接是自定义 Base64
3. X-t header = 13 位毫秒时间戳字符串
4. 自定义 Base64 字母表：`ZmserbBoHQtNP+wOcza/LpngG8yJq42KWYj0DSfdikx3VT16IlUAFM97hECvuRX5`
5. 修改版 CRC32：标准结果异或 `0xedb88320`，返回有符号 32 位整数
6. X-s 内 x3 字段前缀：`mns0101_`（评论接口）/ `mns0201_`（其他）/ `mns0301_`（SDK 默认）
7. X-s-common x5 字段 = cookie `a1` 的值
8. X-s-common x12 字段 = `localStorageDsllt;windowDsl`（如 `null;1780544705228`）
9. SDK 导出函数 `_1619d69735e1d480a72d7e01c4a40b7f(c,u,p)` 返回 number[]（签名字节数组）
10. SDK 导出 `_66062487cf103622475a2f9b17d8293e` = `'mns0301'`（内部默认版本）
11. 业务层入口函数 `seccore_signv2(e, a)` 在 vendor-dynamic.*.js
12. **服务端不校验 x3 指纹部分**：173 字节（无指纹）与 147 字节（有指纹）均被接受
13. **评论接口需 web_session cookie**：无登录态返回 `code:-101`
14. as-v2-fp.js 加载会崩溃（`_0x19467f` 返回 undefined），但**不需要加载**
15. ≥5 次请求签名稳定通过（user/me 5/5，comment/page 5/5）
16. 笔记/评论 API 需 `xsec_token` query 参数（从笔记页 URL 获取）
17. 业务 API 域名 `edith.xiaohongshu.com`，前端域名 `www.xiaohongshu.com`

---

## 关键决策点

### 决策 1：是否需要加载 as-v2-fp.js？
**结论**：不需要。

- fp.js 负责浏览器指纹附加（Canvas/WebGL/Audio 等）
- 加载 fp.js 需解决 `_0x19467f` 崩溃（wrap 返回 stub function）
- 加载成功后 mnsv2 仍未挂载到 window（JSVMP 内部存储）
- **服务端不校验指纹部分**：直接用 ds.js 的 `_1619` 输出即可通过
- 节省 ~390KB 脚本加载 + 环境补丁

### 决策 2：X-s-common 用纯算还是 vm 沙箱？
**结论**：纯算。

- X-s-common 算法完全可读（MD5 + Base64 + CRC32 变体）
- 无 JSVMP 字节码参与
- 纯算实现更稳定、更高效

### 决策 3：x3 字段长度差异是否影响？
**结论**：不影响。

- 真实 x3 = 147 字节（含浏览器指纹）
- 本实现 x3 = 173 字节（纯 `_1619` 输出，无指纹）
- 服务端 5/5 接受，证明只校验格式不校验内容

---

## 与其他 case 的对比

| 维度 | 本案例（小红书） | chameleon（同花顺） | 瑞数 6.0 |
|------|---------------|------------------|---------|
| 签名类型 | 双轨（纯算 + vm） | 单一（vm） | 单一（vm + sdenv） |
| JSVMP | 是（as-v2-ds.js） | 否（混淆但可读） | 是 |
| 环境补丁量 | 最小（15 项） | 中等（10+ 项） | 大（sdenv） |
| 指纹校验 | 否（服务端不校验） | 否 | 是（412） |
| 加载 fp.js | 否（不需要） | N/A | N/A |
| 难点 | 误判 mnsv2 必需 | try-catch 静默吞错 | sdenv 完整性 |

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | RuyiTrace NDJSON 日志驱动逆向 |
| `references/workflow/phase-flow.md` | CLEANUP 经验沉淀 |
| `cases/vm-sandbox-custom-algo.md` | vm 沙箱骨架模板（本案例为其具体填充） |
| `cases/jsvmp-xhr-interceptor-env-emulation.md` | JSVMP + 环境伪装参考 |
