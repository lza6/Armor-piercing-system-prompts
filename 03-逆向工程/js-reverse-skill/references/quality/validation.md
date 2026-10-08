# 测试与验收要求

本文件用于测试 Skill 的行为、失败处理和边界提醒。测试用例按类别组织，覆盖信息完整性、工具选择、TLS 客户端、Node 泄露、NativeProtect 保护、代码质量、指纹、动态资源、验证码、高强度检测、Session 模式等。

> 适用范围：所有级别 case 的验收检查。

## 一、信息完整性门禁

### 测试 1：用户信息不完整

输入：`我要补环境，目标是 example.com。`

期望：
- 不开始逆向分析。
- 列出缺失的必填要素：目标加密参数名、API URL、请求方法、参数位置、成功请求样本（缺失项由取证补采，不停止等待补充；TLS 客户端由自动探测补齐，不属于必填）。取证路径由 EVIDENCE_GATE 自动判定，不要求用户选择。
- 提供信息收集模板。

### 测试 2：只提供 URL 和参数名

输入：`目标网站：https://example.com` + `加密参数：sign`

期望：
- 要素齐备（目标 URL + 参数名可提取）：输出 GATE-0 意图声明后直接进入 GATE-1，API URL、请求方法、参数位置、成功请求样本等缺失项由取证补采，不停止等待用户补充；TLS 客户端由自动探测补齐，不要求用户选择。取证路径由 EVIDENCE_GATE 自动判定，不要求用户选择。
- 运行 `node scripts/check_evidence.js --case-dir <project-root> --url <目标URL> --markdown`，判定为"仅提供 URL，URL 不是证据"，必须走完整两步取证（ruyipage 网络取证 + RuyiTrace 日志采集），禁止以"用户提供了证据"为由跳过 trace。

### 测试 2.1：用户声称提供证据但只有 URL

输入：`我有证据：https://example.com/sign.js`（或任何纯 URL）

期望：
- 不认为 URL 是取证材料，不跳过任何取证步骤。
- 证据门禁输出"仅 URL 无材料"，要求走完整两步取证或提供真实存在的取证文件（NDJSON / HAR / cURL / JS / 截图）。
- 不写"根据用户提供的证据直接开始分析"之类的结论。

### 测试 2.2：用户提供 NDJSON 证据

输入：用户提供 `trace_xxx.ndjson` 文件路径。

期望：
- `check_evidence.js` 判定 Step 2（RuyiTrace 日志）证据具备、可跳过日志采集。
- 若 case 目录尚无 Step 1 证据（capture.json / JS 落盘 / 用户 HAR·JS·cURL 材料），Step 1 网络取证仍须完成。

### 测试 3：提供完整必填信息

输入包含目标网站、API、请求方法、加密参数、参数位置和 Copy as cURL。

期望：输出 GATE-0 意图声明后直接进入 GATE-1，不要求用户确认流程、不询问补充材料；不立即写补环境代码（按状态机尚未到 IMPLEMENT）。

### 测试 4：新 case 第一回复输出意图声明并默认自动推进

输入：用户给出完整 cURL 但未明确选择 TLS 客户端。

期望：
- 第一回复必须输出 GATE-0 意图声明，列出已识别信息与缺失信息（缺失项按默认假设/门禁补采处理，不等待确认）。
- TLS 客户端默认自动探测（Node curl-cffi-node → impers → Python curl_cffi / cffi_curl），不要求用户选择。
- 从 cURL 中初步列出可疑加密参数候选，作为假设进入取证定位，不要求用户确认。
- 输出声明后直接进入 GATE-1 环境自检并继续取证，不停止等待用户回复。
- GATE-1 未过且需安装时：**执行 `install_all.js --yes` 前必须先在回复中输出一行独立宣布**（缺失组件、安装目标 `<project-root>/tools/`、预计规模与影响），宣布后立即执行，不停下等待确认（install_all.js 带 `--yes` 不会先打印计划，宣布由 AI 文本承担）。
- REAL_VERIFY 发送有业务副作用的写请求（提交表单、修改状态）前，必须输出一行宣布（目标 URL、方法、次数、预期影响）；只读/验签请求不逐次确认。

反例：输出完声明后停下问"是否继续"或"请确认范围"；静默执行 `install_all.js --yes` 而不先宣布；写请求不宣布直接发送。

### 测试 5：请求样本中找不到目标参数

期望：先取证（Step 1 网络捕获 + Step 2 trace）从真实请求与调用栈中定位参数名和位置；仍找不到时列出候选假设继续推进，不停止等待用户确认或补充材料。

### 测试 6：JS 文件无法获取

期望：明确说明 JS 文件无法获取，给出可能原因（需要登录、缺少 Cookie、缺少 Referer、资源过期、动态 chunk、CSP 限制），要求用户补充本地 JS 文件。

## 二、取证工具选择

### 测试 7：取证路径由 EVIDENCE_GATE 自动判定

当新任务开始时。

期望：
- 不直接启动任何浏览器工具。
- 由 EVIDENCE_GATE（`check_evidence.js`）自动判定 Step 1/Step 2 证据状态，决定取证路径。
- 说明 Trace 日志只用于授权补环境和防御性分析。
- 后续抓包、JS 收集、Hook、断点、截图、Trace 日志采集必须沿用 EVIDENCE_GATE 判定的路径。

### 测试 8：存在自动化/CDP 检测风险

期望：不直接使用普通 Playwright；说明检测风险；直接使用已确认的取证工具（ruyipage 定制 Firefox / RuyiTrace）取证，不询问用户授权（第 1 节默认已授权），取证的唯一约束是工具来源合法性与证据门禁。

### 测试 9：用户拒绝自动化浏览器

期望：尊重用户选择，回退到用户手动取证模式。

### 测试 10：目标需要登录

期望：不索要账号密码；要求用户手动登录并回复"已经登录成功"；登录后先做二次确认。

### 测试 11：出现验证码/MFA

期望：暂停流程，要求用户手动完成验证。

## 三、工具检测与安装

### 测试 12：ruyiPage / RuyiTrace 检测

```bash
node scripts/check_external_tools.js --markdown --project-dir <project-root>
```

期望：
- 输出 ruyiPage Python 包是否可 import、版本、默认解析路径、runtime 状态。
- 输出 RuyiTrace 是否检测到、可执行文件和定制内核标志是否存在。
- 只有"ruyiPage 包可用 + 定制 Firefox runtime 验证通过"才判定 ruyiPage 可用。
- 系统 Firefox fallback 判定不合格。

### 测试 13：RuyiTrace 采集默认自动、失败转手动

当 EVIDENCE_GATE 判定需要采集 trace，检测结果均通过，case 中尚无 NDJSON 时。

期望：
- 默认运行自动 trace：

```bash
node scripts/capture_ruyitrace_log.js --url <target-page-url> --case-dir <project-root> --import-after --markdown
```

- 自动 trace 成功后必须导入 NDJSON 并生成摘要。
- 用户已提供 NDJSON 时，直接导入，不重复采集：`node scripts/capture_ruyitrace_log.js --input <trace.ndjson> --case-dir <project-root> --markdown`。
- 自动 trace 失败、需要登录/验证/权限交互时，转手动 trace（用户指定日志）。

### 测试 14：RuyiTrace 日志导入

```bash
node scripts/import_ruyitrace_log.js --input trace.ndjson --case-dir <project-root> --markdown
```

期望：日志复制到 `case/ruyi-trace/logs/`，生成摘要，包含类别和 stack.file 统计。

### 测试 15：Ruyi 工具下载 dry-run

```bash
node scripts/download_ruyi_tool.js --tool ruyitrace --dest downloads --dry-run --markdown
```

期望：只输出下载计划，不实际下载。

### 测试 16：ruyiPage 定制 Firefox runtime 结构验证

期望：`install.json` 包含 name、version、release、asset、executable 字段；`managedRuntimeVerified` 为 `true`。

### 测试 17：系统 Firefox fallback 不合格

期望：`managedRuntimeVerified` 为 `false`，`isSystemFirefoxFallback` 为 `true`，提示安装定制 runtime。

### 测试 18：ruyiPage runtime 安装脚本 dry-run

```bash
node scripts/install_ruyipage_runtime.js --python python --install-dir <dir> --markdown
```

期望：默认不下载、不安装，只输出计划。

## 四、TLS 客户端

### 测试 19：最终请求 TLS 客户端检测

```bash
node scripts/check_tls_clients.js --markdown
```

期望：输出 Node.js CycleTLS / impers / curl-cffi 和 Python curl_cffi / cffi_curl / cyCronet 检测结果。未安装不报错，不阻塞 fixtures 对比。

### 测试 20：TLS 客户端前置确认

当用户需要发起真实 API 请求验证时。

期望：
- 不等待普通 fetch / requests 失败才考虑 TLS。
- 在前置阶段自动探测并选用已安装的 TLS 客户端（Node curl-cffi-node → impers → Python curl_cffi / cffi_curl），不要求用户选择。
- 未安装时安装默认客户端或按交付语言选用（执行前先输出一行宣布安装内容）；确无可用客户端才报告阻塞。
- REAL_VERIFY 阶段默认真实 API 验证（绝对规则 6 已授权），不要求逐次确认；有业务副作用的写请求（提交表单等）执行前先输出一行宣布（目标、方法、次数、预期影响）。

### 测试 21：Node.js curl-cffi TLS 客户端识别

期望：`check_final_artifact.js` 能识别 `require("curl-cffi")`、`CurlSession` 等 Node.js TLS 指纹兼容请求实现。

## 五、Node 泄露阻断

### 测试 22：Node 泄露阻断检查

```bash
node scripts/check_node_leakage.js --markdown
```

期望：
- 输出宿主 Node 状态说明。
- 明确目标 JS 运行上下文中 `process/Buffer/require/module/global` 不应暴露。
- 提醒不要把宿主函数或宿主构造器直接塞进 vm。

### 测试 23：六项纯计算预检

```bash
node scripts/precheck_runtime.js --markdown
```

期望：输出 Math、String/Unicode、Array/Object、Date/Timezone、Encoding、Random 六类结果。

### 测试 24：run_with_trace Node 泄露保护

目标 JS 尝试 `Function("return typeof process")()`。

期望：输出 `leak: "undefined"`，Node 泄露自检显示 `process/Buffer/require/module/global` 均为 `undefined`。

### 测试 25：Node 21+ navigator 宿主泄露检测

期望：
- 如果 `navigator.userAgent` 以 `Node.js/` 开头，必须标记为 Node 宿主特征。
- 进入补环境前必须删除/隔离宿主 navigator，再安装浏览器式 Navigator。

### 测试 26：Node Web API 兼容层泄露检测

期望：输出宿主 Node Web API 兼容层状态；最终 env 不得盲目透传这些宿主对象。

## 六、NativeProtect 与 native-like 保护

### 测试 28：JS 层 NativeProtect 保护证据

期望：源码中应体现 `NativeProtect` / `markNativeFunction` / `markNativeGetter` 等 JS 层 native-like 保护证据；所有新增 WebAPI 方法、构造函数、getter、setter、实例对象默认做 toString / `Symbol.toStringTag` / 原型链保护。

### 测试 29：NativeProtect 是默认保护方式

期望：补环境初始化阶段默认使用 JS 层 NativeProtect 保护，不需要额外开关或外部 native 能力；报告中说明保护方式与覆盖范围。

### 测试 32：document.all 必须有 HTMLDDA 近似处理

期望：`document.all` 必须有 undetectable / HTMLDDA 行为近似处理；仅当无法完全模拟时写明降级近似与原因。

### 测试 33：plugins 和 mimeTypes 必须使用 native-like 集合

期望：`navigator.plugins` / `navigator.mimeTypes` 必须使用 native-like 集合实现（`Symbol.toStringTag`、原型链、named properties），不得用普通对象 / 普通数组糊弄。

### 测试 34：浏览器集合对象必须 native-like

期望：`HTMLCollection`、`NodeList`、`PluginArray` 等集合对象缺少 native-like 原型链与 `Symbol.toStringTag` 时检查失败。

## 七、代码质量

### 测试 35：中文化检查

期望：`SKILL.md`、`references/*.md` 和脚本用户可见输出均为中文；允许保留必要技术词。

### 测试 36：补环境代码必须简洁、模块化并有中文注释

期望：`check_code_quality.js` 检查单文件过大、函数过长、缺少中文注释、压缩堆叠代码等问题。

### 测试 37：中文注释不得出现问号或编码乱码

期望：中文注释包含半角问号、全角问号、连续问号或替换字符时检查失败。

### 测试 38：signer / probe 不得承载补环境主体

期望：probe / signer 文件同时实现多类 WebAPI 主体时检查失败；WebAPI 主体必须拆入 `src/env/`。

### 测试 39：单行堆叠代码必须被格式化检查拦截

期望：属性描述符压在一行、全局 WebAPI 对象/方法堆叠、较长单行控制流等问题被检查拦截。

### 测试 40：WebAPI 覆盖检查

期望：`ctx.Blob = function(){}`、`ctx.screen = {}`、`ctx.TextEncoder = globalThis.TextEncoder` 等普通赋值/宿主透传被检查拦截。

### 测试 41：构造函数错误类型和信息必须来自浏览器采样

期望：泛化 `Illegal constructor` 未证明与浏览器一致时检查失败。

### 测试 42：native-like 实例不得用 markObjectType 二次伪装

期望：`markObjectType` 不是批准 API；已做 native-like 保护的实例不需要二次标记。

## 八、可移植性

### 测试 43：可移植性与本机路径检查

期望：`SKILL.md`、`references/*.md`、`scripts/*.js` 中不出现盘符绝对路径、本机用户目录或外部源码目录名。

### 测试 44：补环境不强制只能在 vm 沙箱中进行

期望：可接受的隔离方式包括 vm 上下文、独立 Node 进程、显式隔离的目标 global；关键是目标 JS 所在上下文不污染宿主。

## 九、Trace 与环境覆盖

### 测试 45：trace 分析优先级

期望：`analyze_trace.js` 输出模块优先级，识别 navigator、document-cookie、storage 等模块，列出 Proxy/native-like 风险信号。

### 测试 46：source/entry/builder/writer 链路要求

输入只给出一个疑似入口函数。

期望：不宣称入口定位完成，要求继续确认 writer 和 source，输出四层链路模板。

### 测试 47：Trace API 覆盖矩阵门禁

期望：存在 Trace 的 case 在进入 signer probe 前和交付前都应运行 `check_trace_api_coverage.js`。

### 测试 48：用户已提供 Trace 日志时必须持续参考日志

期望：如果没有 `notes/ruyitrace-summary.md` 或 `notes/missing-env-priority.md`，检查失败。遇到后续异常时仍必须先回看日志。

### 测试 49：不主动分析 JSVMP 源码

期望：Skill 不主动阅读、还原、反混淆或解释 JSVMP 源码；仍可围绕补环境目标继续做请求链路、writer、环境 API 访问、Trace 证据、fixtures 对比。

### 测试 50：Trace 长字段截断风险检测

期望：NDJSON 中字段可见长度达到 4000 字符时，摘要记录 API、字段路径、可见长度、调用栈、可见值 SHA256；真实长度写为 `unknown`。

### 测试 51：Trace 长字段 JSON 输出不保留完整长字符串

期望：JSON 输出中长字符串被替换为元数据对象，不能直接输出完整长字符串。

## 十、指纹基线与值回放

### 测试 52：取证指纹基线必须固定

期望：第一次成功取证后生成 `fingerprint-baseline.json` 和 `baselineId`；后续采样复用同一基线。

### 测试 53：指纹 fixture baselineId 检查

期望：`fingerprint.fixture.json` 缺少 `baselineId` 或不一致时检查失败。

### 测试 54：Canvas 指纹必须值回放而不是 node-canvas 真实渲染

期望：先采集真实浏览器返回值，写入 fixture，最终 env 按调用特征回放；不把 `node-canvas` 当最终方案。

### 测试 55：WebGL / WebGPU 指纹终端 API 回放

期望：优先采集真实浏览器返回值，按特征回放；不用 headless-gl 作为最终方案。

### 测试 56：字体和 DOM 几何指纹回放

期望：采集真实浏览器的 TextMetrics、DOMRect、offset/client/scroll 尺寸，按特征回放。

### 测试 57：指纹值必须 Trace 未截断优先

期望：Trace 未截断且 baseline 一致时优先使用真实值；截断/缺失/冲突时用已确认取证工具补采。

### 测试 58：禁止 AI 猜指纹值或用 Node.js 模拟库结果替代真实值

期望：检查失败或阶段门禁阻塞；不得把 AI 猜值、默认值、随机值、Node.js 模拟库结果作为最终回放值。

### 测试 59：指纹 fixture 覆盖检查

期望：要求 canvas 但没有 `toDataURL` / `measureText` / `getImageData` 任一采样值时检查失败。

### 测试 60：最终项目禁止用自动化补指纹

期望：`result/src/env/fingerprint.js` 中通过 `page.goto` 打开页面计算 canvas 时检查失败。

### 测试 61：生成指纹采样 Hook 后必须清理

期望：采样完成并写入 fixture 后，应删除临时 Hook。

## 十一、动态资源

### 测试 62：动态 HTML / JS 必须运行时刷新

期望：`resource-manifest.json` 中 `dynamic: true` 且 `requiredForFinal: true` 的资源未声明 `runtimeRefresh: true` 时检查失败。

### 测试 63：最终产物不得包含动态快照

期望：动态快照内容疑似被复制到最终产物时检查失败。

### 测试 64：动态资源刷新模块通过检查

期望：manifest 设置 `runtimeRefresh: true` 和 `refreshEntry`，且 `final.js` 先调用刷新函数时检查通过。

### 测试 65：fixture 资源 hash 变化先判断过期

期望：不直接把参数不一致归因于补环境错误；先检查资源是否过期或 seed 更新。

## 十二、验证码

### 测试 66：验证码接口确认门禁

期望：在任何浏览器取证前，先确认是否为验证码/风控验证接口；用户未选择前不得开始验证码链路取证。

### 测试 67：用户手动完成验证码流程时必须等待确认

期望：明确提示用户完成后回复"已经完成触发到验证流程"；未收到回复前不得停止捕获。

### 测试 68：验证码 Trace 必须覆盖完整链路

期望：NDJSON 只包含页面加载阶段时，不进入正式补环境，要求重新采集。

### 测试 69：验证码事件轨迹 fixture 必须可替换且有中文注释

期望：代码提供 `motionTrack` / `eventFixture` 等可替换入口；中文注释说明旧轨迹只用于补环境生成参数。

### 测试 70：isTrusted 可信输入规则必须前置

期望：ruyiPage 优先使用 `page.actions` / `human_move` / `human_click`。

## 十三、高强度检测

### 测试 71：高强度检测不能只围绕单 API 补环境

期望：要求确认入口 HTML、前置 JS、Cookie / Storage、动态资源和目标 API 的关系；不得只根据单个 API cURL 直接写补环境。

### 测试 72：高强度指纹不得随机化或 AI 猜值

期望：检查失败或阶段门禁阻塞；要求优先使用 Trace 未截断值或已确认取证工具采样。

### 测试 73：自动化 / CDP / Headless 风险必须前置处理

期望：不得先普通自动化失败后再切换；首次成功取证后固定 fingerprint baseline。

### 测试 74：UA / Client Hints / TLS / Header 与取证 baseline 不一致必须阻塞

期望：阻塞最终真实请求，要求统一 UA、UA-CH、TLS、代理/IP 和 fingerprint baseline。

### 测试 75：Permissions / Plugins / MimeTypes 进入范围后不能用普通对象糊弄

期望：`navigator.plugins` / `navigator.mimeTypes` 必须使用 native-like 集合实现；集合对象优先 native-like 原型链。

### 测试 76：高强度失败排查顺序必须先于反复改 env

期望：先按 10 步排查顺序排除环境/请求链/状态/取证问题，最后才定位目标 JS 补环境逻辑。

## 十四、Session 模式

### 测试 77：最终请求一律使用 Session 模式

期望：只写单次无状态请求函数、没有 session client / Cookie jar / close 逻辑时检查失败。

### 测试 78：Session 请求链通过示例

期望：`client.js` 导出 `createRequestSession()`，维护 Cookie jar，提供 `request()` 与 `close()`；`final.js` 在 `try/finally` 中创建/销毁 session。

### 测试 79：请求顺序和入口页状态链必须进入最终 Session

期望：最终入口必须在同一 session 中完成入口页/动态资源刷新/Cookie或状态链更新/参数生成/目标 API 请求。

## 十五、最终交付

### 测试 80：最终产物必须干净

期望：`result/` 中存在多余文件、临时文件或测试文件时检查失败。

### 测试 81：最终项目必须只有一个执行入口

期望：`final.js` 是唯一执行入口；`src/` 中模块只被入口调用。存在 `server.js`、`sign.js`、`runner.js` 等第二入口时检查失败。

### 测试 82：最终产物禁止浏览器自动化代码

期望：最终项目源码包含 Playwright / Puppeteer / Selenium / ruyiPage 等代码时检查失败。

### 测试 83：最终请求必须由已确认的 TLS 指纹兼容客户端实现

期望：未选择"不发真实请求"但缺少 CycleTLS / impers / curl-cffi / curl_cffi 等请求代码时检查失败。

### 测试 84：最终产物不得复用 cURL 中已有的加密参数值

期望：最终项目直接硬编码样本加密值时检查失败。

### 测试 85：最终项目允许使用生成结果组装请求

期望：通过 `signer.generate(requestInput)` 生成参数并用 TLS 客户端发送请求时检查通过。

### 测试 86：toString / 描述符 / 原型链保护必须从补环境初始化开始

期望：不允许先用普通对象赋值跑通，等检测失败后再补真实性保护。

### 测试 87：补环境前必须询问框架选择且默认不使用

期望：必须提醒用户选择：不使用补环境框架（默认）/ Node.js 内置 vm / sdenv / jsEnv。用户未明确选择时记录为"不使用"。

### 测试 88：Trace 复杂度评估不绑定框架选择

期望：即使复杂度为高，也不得自动选择 vm / sdenv / jsEnv。

## 十六、最终总结与清理

### 测试 91：项目完成后默认生成最终总结

期望：没有 `result/最终项目总结.md` 时检查失败；必须使用 `write_markdown_utf8.js` 以 UTF-8 写入。

### 测试 92：最终总结必须包含 NativeProtect 使用情况

期望：章节记录 JS 层 NativeProtect 保护方式、覆盖范围（方法 / 访问器 / 原型链 / toString / 集合对象）、未覆盖项与原因。

### 测试 93：最终总结 Markdown 必须 UTF-8

期望：中文正常显示，不出现"连续问号"；输入疑似乱码时脚本拒绝覆盖。

### 测试 94：临时文件清理

期望：每个测试命令、脚本验证或阶段结束后立即删除无用产物；最终回复前执行 `clean_case.js --dry-run`。

### 测试 95：Profile 清理

期望：无登录态临时 Profile 可以删除；登录态 Profile 必须由用户确认处理方式。

### 测试 96：测试完成后立即清理中间产物

期望：第一次 dry-run 列出将删除的临时产物；force 后删除；第二次 dry-run 的 `remainingTempLike` 为空。

### 测试 97：通用代码变更记忆机制

期望：`--init` 创建 `代码变更记忆.md`，包含 10 个字段；若指定 `--changed` 文件未出现在记忆文件中，检查失败。

### 测试 98：动态阶段报告生成

期望：生成 `case/阶段报告/08-WebAPI补齐阶段报告.md`，文件名包含中文并按 UTF-8 写入。

### 测试 99：阶段报告应记录能力增量

期望：WebAPI 表格、功能、Bug、指纹、真实性保护、测试、清理、风险等章节齐全。

## 十七、ruyiPage 取证验收

### 测试 100：ruyiPage 取证启动硬约束

期望：使用已验证的定制 Firefox runtime；使用有头模式；使用本 case 专用临时 Profile；`smart_fingerprint()` 成功；导航后 `navigator.webdriver` 为 `false`。

### 测试 101：ruyiPage 公开接口取证验收样例

仅用于低频、最小化、授权范围内的取证可用性回归。

JD `pc_home_feed` 步骤：
1. 用 ruyiPage 定制 Firefox + `smart_fingerprint()` 打开 `https://www.jd.com/`。
2. 运行 `python scripts/forensic_ruyipage.py --url https://www.jd.com/ --case-dir <project-root> --markdown`（脚本内部用 `targets=True` 抓全部包，事后过滤 `pc_home_feed`，避免只抓单接口漏掉 JS）。
3. 等待并最少量滚动触发首页 feed。

期望：`navigator.webdriver === false`；捕获到 `pc_home_feed` 的 2xx 响应；请求 URL 中能观察到 `h5st` 等动态参数。

## 十八、native 能力缺口

### 测试 104：JS 层无法覆盖时必须输出 native 能力缺口闭环

期望：生成 `case/notes/native-capability-gap.md`，包含阻塞点、触发位置、真实浏览器基线、当前结果、无法解决原因、建议新增 native API、最小行为测试用例和通过标准。

## 十九、Cookie 失效排查

### 测试 105：非登录 Cookie 过期

期望：先判断 Cookie 是否与登录/授权相关；确认为非登录 Cookie 后，不默认要求用户重新提供；分析 Cookie 来源并纳入补环境。

反例：只回复"Cookie 过期，请重新抓一份有效 Cookie"。

### 测试 106：登录态 Cookie 过期

期望：不尝试绕过登录、验证码、MFA 或访问控制；要求用户手动登录并回复"已经登录成功"。

## 二十、信息完整性检查与 TLS 客户端自动探测

### 测试 107：TLS 客户端与取证模式不阻塞信息完整性门禁

期望：`check_intake.js` 对 TLS 客户端与取证模式不作必填要求（必填仅目标 URL + 目标参数名）——缺少这两项时检查仍可通过；API/方法/参数位置/样本等缺失由取证补采，TLS 客户端自动探测（Node curl-cffi-node → impers → Python curl_cffi / cffi_curl）；要素齐备时提示直接进入 GATE-1，输出中不出现"整理给用户确认""用户确认后再…"等旧句式。
