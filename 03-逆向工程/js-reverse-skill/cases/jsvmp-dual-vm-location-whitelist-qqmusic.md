# Case：JSVMP 双 VM + location.host 白名单静默降级（QQ 音乐 musics.fcg）

> 难度：★★★
> 还原方案：B vm 沙箱执行 + D 环境伪装
> 实现语言：Node.js
> 最后验证日期：2026-08-09
> 平台类型：QQ 音乐（y.qq.com / u6.y.qq.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- **JS 特征**：
  - webpack `vendor.chunk.<hash>.js`，单文件 660KB+
  - 文件内存在**两段**独立 JSVMP IIFE，形如 `(function(){...})("Xh7YHJ...")`，实参是超长 base64 字节码
  - VM 解释器为 `while + switch(case 0..82)` 结构
  - 两段 VM 尾部均有「挂载后立即 delete」模式：
    `var ie = ne._getSecuritySign; delete ne._getSecuritySign;`
    `var ae = oe.__cgiEncrypt, se = oe.__cgiDecrypt; delete oe.__cgiEncrypt, delete oe.__cgiDecrypt;`
  - 可直接全文搜索的锚点字符串：`_getSecuritySign`、`__cgiEncrypt`、`__cgiDecrypt`
- **参数特征**：
  - `sign` = `zzc` + 8 hex + 24 [a-z0-9] + 8 hex，总长 44
  - 固定伴随 `encoding=ag-1` query 参数
  - 请求 body 为纯 ASCII base64 文本，`Content-Type: text/plain`
- **请求特征**：
  - URL 在发送前由 `cgi-bin/musicu.fcg` 就地改写为 `cgi-bin/musics.fcg`
  - 无预热请求，无 Cookie 依赖（游客态可用）
  - 响应 `content-type` 标为 `text/plain; charset=utf-8`，**实际是二进制密文**
- **反调试特征**：
  - 无 debugger 定时器
  - 主要防护是**静默降级**而非报错：环境不满足时返回格式合法的废签名

## 加密方案

- 路径：**B vm 沙箱执行 + D 环境伪装**
- 框架：Node 内置 `vm`（不使用 jsdom）
- TLS 客户端：Node 原生 `fetch`（本站无 TLS 指纹校验，无需 CycleTLS / curl-cffi）
- 核心思路：从 vendor.chunk 切出两段 JSVMP IIFE，在 `vm.createContext` 里用自引用 sandbox 装载，接出 `_getSecuritySign` / `__cgiEncrypt` / `__cgiDecrypt` 三个原语，再按 `fe()` 管线复刻请求。

**完整管线**

```js
body   = JSON.stringify({ comm, req_1: { module, method, param } })
sign   = getSecuritySign(body)          // 同步，纯函数
cipher = await cgiEncrypt(body)         // async！返回 base64 文本
url    = `${API}?_=${Date.now()}&encoding=ag-1&sign=${sign}`
// POST, Content-Type: text/plain, body = cipher
plain  = cgiDecrypt(arrayBuffer)        // 同步，只吃 ArrayBuffer
json   = JSON.parse(plain)
```

## 踩坑记录

1. **坑：`ReferenceError: re is not defined`** —— 切出的 VM 片段引用了 babel `_typeof` 助手 `re`，而它定义在 vendor.chunk 顶部、不在切片范围内。
   → **正确做法**：装载前拼一段 helper：
   `var re=function(e){return e&&"undefined"!=typeof Symbol&&e.constructor===Symbol?"symbol":typeof e};`

2. **坑：`Cannot set properties of undefined (setting '_getSecuritySign')`** —— VM 内部先自探测全局对象（`typeof window`/`self`），Node 里用 `var ne = {}` 承接会导致 VM 内部实际写入的对象与你拿到的不是同一个。
   → **正确做法**：`vm.createContext(sandbox)` 且令 `sandbox.window = sandbox.self = sandbox.globalThis = sandbox` 自引用，再用 `var ne = window;` 作为 prologue。

3. **坑（本 case 最隐蔽）：签名静默降级** —— 环境不满足时 `getSecuritySign` **不报错**，返回格式完全合法的 `zzc381c7d8...`，服务端一律 `code:2000`。极易误判成「参数写错了」而反复调 body。
   → **正确做法**：必须提供 `navigator` 对象（`{}` 即可）**且** `location.host` 含 `qq.com`。落地时加一道显式守卫，host 不匹配直接抛错。

4. **坑：用「篡改 sign / 不带 sign 是否报错」来验证签名有效性** —— 本站明文 body 直连 `musics.fcg` 时，正确 sign、错误 sign、无 sign 三种情况**全部返回 2000**，该对照实验完全无效。
   → **正确做法**：`musics.fcg` 已强制 `encoding=ag-1`，2000 在签名校验之前就被触发。判定签名有效性只能看「完整加密链路能否拿到 `code:0`」。想单独验证 body 结构，改打不需要签名的 `musicu.fcg`（返回 `code:0` 即结构正确），以此隔离「body 错」与「签名错」。

5. **坑：`cgiDecrypt` 传 `Uint8Array` 不报错但结果错误** —— 传 Uint8Array 时 VM 原样透传输入，返回值开头与密文 hex 完全一致（如 `\u0001\u001d` 对应 `011d`），看着像「解密失败」实则根本没执行。
   → **正确做法**：只传 `ArrayBuffer`。在包装层加 `instanceof ArrayBuffer` 断言，把静默错误变成显式报错。

6. **坑：按 `content-type` 决定是否解密** —— 源码逻辑判断 `application/octet-stream` 才解密，但服务端实际返回 `text/plain; charset=utf-8`，内容却是二进制密文。照抄源码逻辑会跳过解密。
   → **正确做法**：加密请求的响应无条件走解密，不看 content-type。

7. **坑：`cgiEncrypt` 是 async** —— 忘记 await 会把 `[object Promise]` 当 body 发出去。
   → **正确做法**：`await cgiEncrypt(body)`；注意 `cgiDecrypt` 反而是同步的，两者不对称。

8. **坑：ruyipage 取证脚本环境不一致** —— 本 case 取证期曾遇到 ruyipage `1.2.20` 与新版 API 不兼容、以及系统 Python 与内置 Python 混用、智能指纹默认校验 US 等环境问题。**已随工具链升级解决**：`70a41b0` 起脚本适配 ruyipage `1.2.61+`（使用 `page.capture.start(targets=True, collect_bodies=True)` 抓全部包），`--require-country` 缺省改为不校验出口国家（适配代理出口 IP），统一由 `check_external_tools.js` 检测工具链并提示补齐。当前不再需要手动传 `--require-country CN` 或用 `py -3` 切换 Python。
   → **正确做法**：取证不通时不要死磕。本 case 目标接口无登录态，直接用 Node 打真实接口、以服务端 `code:0` 反证还原正确性，比抓包比对更硬。

## 可验证事实清单（经验资产）

1. `sign` 格式：`zzc` + 8 hex + 24 [a-z0-9] + 8 hex，总长 44
2. `sign` 是**纯确定性函数**，只依赖 body 字符串；跨进程、跨轮次同 body 同结果，不含时间戳或随机数
3. 请求密文 body **每次不同**（带随机 IV），与 sign 的确定性形成对照——两者不是同一套密钥体系
4. 签名 VM 只读 `location.host` 一个 location 字段，判据是 `indexOf("qq.com") !== -1`：`aaqq.combb` 能过，`music.163.com` 不能过
5. 签名 VM 读取 `navigator.userAgent`，但该值**不参与摘要**：Chrome/Win 与 Safari/Mac 签出同一个值
6. `navigator` 可以是 `{}` 空对象；为 `undefined` 走降级分支，为 `null` 抛 `Cannot read properties of null (reading 'userAgent')`
7. 降级签名是固定值 `zzc381c7d8i5d9eub3gzfcjsntukn1dlderv47e8b8cca`（对应 topid=4 测试 body），可作为「环境没配对」的快速判据
8. `document` / `screen` / `history` / `performance` / `atob` / `btoa` / `crypto` 与签名无关，删除后签名值不变
9. `cgiEncrypt` 是 async 返回 Promise；`cgiDecrypt` 是同步函数——两者不对称
10. `cgiDecrypt` 只接受 `ArrayBuffer`；传 `Uint8Array` 会原样透传输入而不报错
11. 加密请求的响应 `content-type` 是 `text/plain; charset=utf-8`，但内容为二进制密文（首字节常见 `0x01 0x1d`）
12. 同一密钥体系下，多次响应密文的**前 24 字节相同**（`011def723afe0d305e7d4eba331828ee2c115e7db94c0dba`），说明响应侧密钥固定、非每次协商
13. `musicu.fcg` 无需签名即可访问，可用于隔离验证 body 结构是否正确
14. `musics.fcg` 已强制 `encoding=ag-1`，明文 body 一律 `code:2000`
15. 游客态（`comm.uin = "0"`，无 Cookie）可正常拉取榜单与歌曲详情；搜索模块 `music.search.SearchCgiService` 返回 `sum:0` 空列表，存在额外门槛

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/decision-tree.md` | JSVMP 题型判定 → 选 B vm 沙箱而非纯算还原 |
| `references/workflow/phase-flow.md` | FORENSIC_CAPTURE → TRACE_CAPTURE → TRACE_ANALYZE 取证失败时的降级路径 |
| `references/quality/validation.md` | 交叉验证 ≥5 次、以业务 code 判定成功 |
| `references/quality/high-strength-detection.md` | 静默降级型环境检测（非报错型反调试） |
| `references/quality/final-summary.md` | 8 章总结模板 |
