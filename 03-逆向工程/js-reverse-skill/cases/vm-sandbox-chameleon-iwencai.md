# Case：vm 沙箱执行 chameleon.js 生成 hexin-v（同花顺问财）

> 难度：★★★
> 还原方案：B vm 沙箱执行
> 实现语言：Node.js
> 最后验证日期：2026-07-11
> 平台类型：同花顺问财（iwencai.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] 文件名含 `chameleon`（如 `chameleon.1.9.min.1783727.js`）
- [x] 字符串数组混淆（base64 编码的字符串数组 + 旋转 + 索引偏移函数）
- [x] 控制流平坦化（CFF，switch-case 循环 + 状态机）
- [x] 顶层 `var TOKEN_SERVER_TIME=<timestamp>`（服务端时间注入）
- [x] 末尾 IIFE 用 `try{...}catch(n){return n}` 包裹初始化（静默吞错反调试）
- [x] 非 JSVMP（无 200KB+ while-switch 字节码解释器，混淆但可读）

### 参数特征
- [x] header `hexin-v`，长度 60 字符
- [x] 字符集：Base64 变种（含 `A-Za-z0-9_-`）
- [x] 每次值不同（含时间因子 serverTimeNow / timeNow）
- [x] **参数 = cookie**：hexin-v 的值等于 cookie "v" 的值

### 请求特征
- [x] 缺/错 hexin-v 不返回 403/412，返回 HTTP 200 + 业务数据（当前未强制校验）
- [x] 页面正常加载，无 412 循环（区别于瑞数签名型）
- [x] 无 webmssdk/byted_acrawler（区别于字节行为型）

### 反调试特征
- [x] try-catch 静默吞错（末尾 IIFE 包裹）
- [x] setInterval 定时刷新（12e5ms = 20分钟，阻止 Node 进程退出）

---

## 加密方案

- **路径**：B vm 沙箱执行
- **框架**：vm（Node.js 原生 vm 模块，非 jsdom）
- **TLS 客户端**：Node.js 原生 https
- **核心思路**：vm.createContext 提供中等量浏览器环境 stub（document/navigator/window/Element/XMLHttpRequest），执行 chameleon.js 全文，读取 cookie "v" 作为 hexin-v

### 算法细节

**hexin-v = cookie "v"**，由 chameleon.js 生成：

1. **指纹采集**（18 字段 typed array `A([4,4,4,4,1,1,1,3,2,2,2,2,2,2,2,4,2,1])`）：
   - `u[0]`：随机数（d.random()）
   - `u[1]`：serverTimeNow（TOKEN_SERVER_TIME）
   - `u[2]`：timeNow（Date.now()）
   - `u[3]`：navigator.userAgent 的 hash
   - `u[4]`：操作系统类型（Win32/iPhone/Android 等）
   - `u[5]`：浏览器类型（Chrome/Firefox/Safari 等，25 种检测）
   - `u[6]`：plugin 数量
   - `u[7]-u[10]`：鼠标事件计数（mousemove/click/keydown 等）
   - `u[13]`：浏览器特征位（16 项检测，含 Shockwave/PDF/语言/时区等）
   - `u[15]`：全局标志位
   - `u[16]`：更新计数
   - `u[17]`：版本号（3）

2. **编码流程**：
   - 自定义 checksum 函数 `e(n)`：`u = (u*5 + u + n[i]) & 0xFF` 循环
   - XOR 编码：`c = 131` 初始值，`c = ~131 * c` 每字节
   - Base64 变种编码（自定义字母表）

3. **初始化序列**：`n.init() → v.init() → M.init() → q.init()`
   - `q.init()` 的 `case"0"` 调用 `A()` 生成 cookie "v"
   - `A()` 调用 `M.update()` 生成签名值，写入 cookie

---

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | **try-catch 静默吞错** | vm.runInContext 运行成功但 cookie "v" 未生成，无任何错误输出 | 字符串替换把末尾 `try{w[n(722)](e)}catch(n){return n}` 透明化为 `w[n(722)](e)`，让真实错误抛出 |
| 2 | **Element 全局对象缺失** | 透明化后报 `ReferenceError: Element is not defined`，chameleon.js patch `Element.prototype` 事件方法 | sandbox 中添加 Element/Document 构造函数 + prototype stub |
| 3 | **setInterval 阻止进程退出** | 生成 hexin-v 后进程挂起不退出，chameleon.js 设置 `setInterval(refresh, 12e5)` | 测试入口添加 `process.exit(0)`，生产代码按需调用 |
| 4 | **navigator 属性不全** | 报 `TypeError: navigator[r(...)] is not a function` | 用 Proxy 拦截 navigator 所有属性访问，精确发现缺失项后补齐（mimeTypes/vendorSub/product/productSub/appName/appCodeName/javaEnabled/taintEnabled 等） |
| 5 | **业务成功判定字段错误** | 验证脚本判 `parsed.status === 0` 全部失败，但实际请求成功返回数据 | iwencai 用 `status_code` 不是 `status`，需按实际响应字段调整 |
| 6 | **参数 = cookie 发现路径** | hexin-v 是 header，但生成逻辑在 cookie "v" | Grep 搜索 `getCookie("v")` 定位到生成逻辑 `e=n.getCookie("v")||v[r(503)](z)` |

---

## vm 沙箱环境 stub 清单

vm 沙箱场景下需要的中等量浏览器环境（非完整补环境，但比骨架模板的"最小 sandbox"多）：

### 必须提供
| 对象 | 关键属性/方法 | chameleon.js 用途 |
|------|-------------|-----------------|
| `document` | cookie(getter/setter)、readyState、documentElement、body、head、createElement、getElementsByTagName、addEventListener | cookie 读写、DOM 操作、事件绑定 |
| `navigator` | userAgent、platform、plugins、mimeTypes、language、languages、vendor、vendorSub、product、productSub、appName、appCodeName、javaEnabled、taintEnabled、webdriver、maxTouchPoints、hardwareConcurrency、cookieEnabled、doNotTrack、onLine | 指纹采集、浏览器类型检测 |
| `location` | href、protocol、host、hostname、pathname、origin、search、hash | URL 解析、域名提取 |
| `screen` | width、height、availWidth、availHeight、colorDepth、pixelDepth | 屏幕指纹 |
| `window` | 上述所有 + setTimeout、setInterval、parseInt、Date、Math、JSON、btoa、atob、encodeURIComponent、getComputedStyle、matchMedia、innerWidth、innerHeight、devicePixelRatio | 全局上下文 |
| `XMLHttpRequest` | 构造函数 + prototype(open/send/setRequestHeader/addEventListener) | 被 chameleon.js patch 注入 hexin-v header |
| `Element` | 构造函数 + prototype(addEventListener/removeEventListener/setAttribute) | **chameleon.js patch Element.prototype 事件方法** |
| `fetch` | 函数 stub | 被 chameleon.js patch |
| `Headers` | 构造函数 + set/get/has/append/delete | fetch 请求头 |
| `localStorage` | getItem/setItem/removeItem/clear/length/key | 本地存储 |

### 可选（chameleon.js 可能访问但不影响 cookie 生成）
| 对象 | 说明 |
|------|------|
| `Document` | 构造函数 stub，prototype 可指向 Element.prototype |
| `matchMedia` | 返回 `{matches: false, addListener(){}, removeListener(){}}` |
| `getComputedStyle` | 返回空对象 |

### 不需要（vm 沙箱边界）
- 不需要完整原型链（EventTarget → Node → Element → HTMLElement → ...）
- 不需要 NativeProtect（chameleon.js 不做 toString 检测）
- 不需要 jsdom
- 不需要 Canvas/WebGL/Audio 指纹回放

---

## 与骨架模板的区别

`vm-sandbox-custom-algo.md` 骨架模板说"不需要 jsdom / 补环境 / 浏览器指纹，只需 vm.createContext 提供最小 sandbox"，但本案例实际需要**中等量浏览器环境 stub**：

| 维度 | 骨架模板（理想） | 本案例（实际） |
|------|---------------|-------------|
| sandbox 内容 | navigator/Date/Math/parseInt | + document/Element/location/screen/XMLHttpRequest/fetch/Headers/localStorage |
| 环境补丁量 | 0-3 项 | 10+ 项 |
| 错误来源 | sandbox 缺失依赖 | + try-catch 静默吞错 + Element 缺失 + setInterval 退出 |

**边界判断**：混淆 JS 若 patch 原型方法（Element.prototype/XMLHttpRequest.prototype），需要提供对应构造函数 stub；这是 vm 沙箱与补环境的中间地带，仍属 vm 沙箱（不需要完整原型链和 NativeProtect）。

---

## 可验证事实清单（经验资产）

1. hexin-v header 长度 60 字符
2. hexin-v 字符集为 Base64 变种（含 `A-Za-z0-9_-`）
3. hexin-v 的值等于 cookie "v"（通过 `getCookie("v")` 定位）
4. chameleon.js 文件名格式：`chameleon.<version>.min.<timestamp>.js`
5. chameleon.js 顶层有 `var TOKEN_SERVER_TIME=<timestamp>`（服务端时间）
6. chameleon.js 末尾 IIFE 用 try-catch 包裹初始化（静默吞错反调试）
7. chameleon.js 设置 `setInterval(refresh, 12e5)`（20分钟刷新）
8. vm 沙箱必须提供 Element 全局对象（patch Element.prototype 事件方法）
9. 签名输入含 18 个字段（serverTimeNow/timeNow/navigator hash/浏览器特征/plugin 数/鼠标事件等）
10. 编码算法：自定义 checksum → XOR with 131 multiplier → Base64 变种
11. 初始化序列：`n.init() → v.init() → M.init() → q.init()`
12. ≥5 次请求签名稳定通过（5/5 成功，返回 status_code:0 + 策略数据）
13. iwencai 业务响应用 `status_code` 字段（不是 `status`）

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | 统一日志驱动逆向流程 |
| `references/workflow/common-pitfalls.md` | 反模式 8（try-catch 静默吞错） |
| `references/env/env-debug-loop.md` | 静默吞错诊断方法 + setInterval 退出陷阱 |
| `references/env/env-object-model.md` | vm 沙箱场景的 Element stub 参考方向 |
| `cases/vm-sandbox-custom-algo.md` | vm 沙箱骨架模板（本案例为其具体填充） |
