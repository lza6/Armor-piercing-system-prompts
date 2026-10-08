# 信息收集模板

> **输入方式**：
> - 用户提供 cURL/HAR/JS 文件 → 先运行 `node scripts/check_evidence.js --case-dir <project-root> --url <目标URL> --inputs <材料路径> --markdown` 验证材料真实存在，门禁通过后从包中提取信息，跳过 FORENSIC_CAPTURE ruyipage 抓包
> - 用户只提供 URL + 参数名 → FORENSIC_CAPTURE ruyipage 自动抓包获取其余信息；**URL 不是取证材料，不能作为跳过 trace 的证据**
>
> 本模板用于用户主动补充信息或 skill 抓包后确认信息时使用。

## 必填（用户提供即可启动）

- **目标 URL**（必填）：页面 URL 或 API URL
- **目标加密参数名**（可选）：如 sign / a_bogus / token；为空时 skill 自动识别可疑参数

> 用户只提供 URL 时，skill 通过 FORENSIC_CAPTURE ruyipage 抓包 + TRACE_CAPTURE RuyiTrace 日志采集获取：目标 API、请求方法、参数位置、成功请求样本、响应特征、反爬类型、补环境证据。
> 抓包遇到登录/交互/验证码时暂停，要求用户补充请求包。

## 用户提供 cURL/HAR/JS 文件时

用户提供以下任一**真实存在的文件**即可：
- cURL（Copy as cURL 文本，需先落盘为文件如 `case/notes/user-curl.txt`）
- HAR 文件
- JS 文件
- RuyiTrace NDJSON 日志（提供后可直接跳过 Step 2 日志采集）
- 调用栈截图

> ⚠️ **URL ≠ 证据**：目标页 URL、接口 URL、JS 文件 URL 都只是"目标地址"，不是取证材料。仅提供 URL 时必须走完整两步取证（ruyipage 网络取证 + RuyiTrace 日志采集），禁止以"用户提供了证据"为由跳过 trace。
>
> 所有"用户已提供材料"的判定，必须先用 `node scripts/check_evidence.js --case-dir <project-root> --url <目标URL> --inputs <材料路径,逗号分隔> --markdown` 验证文件真实存在，并以脚本输出的可跳过步骤为准。

从包中直接提取：目标 API、请求方法、参数位置、成功请求样本、响应特征、JS 文件 URL。跳过 FORENSIC_CAPTURE 抓包，但仍需下载 JS 文件识别反爬类型。

## 自动获取字段（FORENSIC_CAPTURE ruyipage 抓包后回填，用户可修正）

### 反爬类型（自动判断，用户可覆盖）

skill 根据 FORENSIC_CAPTURE 抓包结果 + JS 文件特征自动判断：
- JS 文件 <50KB + 无 while-switch + 标准 md5/aes 特征 → 纯算还原
- JS 文件含 WASM 加载 → WASM 加载
- JS 文件含 webmssdk / a_bogus → 行为型补环境
- 响应码 412 循环 → 签名型补环境
- JS 文件含 _0x 前缀 → 纯混淆（AST 反混淆后判断）

详细识别标准见 `references/workflow/decision-tree.md`「反爬类型识别」。

### TLS 指纹客户端（自动选择，用户可覆盖）

- 纯算无 TLS 检测：标准 fetch/requests
- 有 TLS 检测：Node.js curl-cffi-node / Python curl_cffi / cyCronet
- 不发真实请求：只输出本地 sign/参数

### 请求详情（skill 抓包获取）

- 目标 API URL
- 请求方法
- 目标参数名（如用户未指定，skill 列出可疑参数作为候选假设，由取证定位确认，不等待用户选择）
- 参数位置：Query / Header / Body / Cookie

### 成功请求样本

- 用户提供 cURL/HAR 时：用户提供的请求/响应包即为成功样本
- 自动抓包时：skill 通过 FORENSIC_CAPTURE 自动抓取至少一份成功请求（含完整 Headers/Cookie/Body/Response）

用户也可补充 cURL：

```bash
# 用户可粘贴 Copy as cURL 补充（可选）
```

## 可选补充字段（用户提供可加速分析）

### 已知 JS 文件
- JS 文件 URL：
- 是否已下载到本地：是 / 否
- 本地文件路径：

### DevTools 调试信息
- Network Initiator 截图或文本：
- 调用栈 Stack Trace：
- 可疑函数名：
- 可疑代码片段：

### 登录态
- 是否需要登录：是 / 否
- 登录态获取方式：用户提供 cookie / skill 自动登录 / 无需登录

## 可选确认字段（工程化交付时补充）

以下字段在自动抓包/用户提供 cURL 时均非必填，但在生产级交付时建议确认：

### 网络与代理

- **代理类型**：无代理 / HTTP / SOCKS5 / 隧道
- **代理地址**：（如适用）
- **并发限制**：最大并发请求数
- **重试策略**：重试次数 / 退避方式（固定 / 指数）
- **超时设置**：连接超时 / 读取超时（秒）
- **请求频率**：每秒/每分钟最大请求数
- **TLS 版本**：TLS 1.2 / TLS 1.3 / 自动协商
- **HTTP 版本**：HTTP/1.1 / HTTP/2 / HTTP/3

### 环境固定值

- **UA 固定值**：是否锁定固定 UA（是 / 否 + 具体值）
- **Cookie 刷新策略**：定时刷新 / 失效重取 / 手动提供
- **Viewport 固定值**：宽度 / 高度 / DPR

### 输出与调试

- **日志级别**：debug / info / warn / error
- **输出格式**：JSON / 文本 / CSV
- **调试模式**：开启（打印中间值） / 关闭

## 阻断项（以下情况必须暂停）

- 需要登录态：抓包遇到登录/验证码/MFA 时暂停，要求用户手动登录或补充请求包，不绕过登录验证
- 目标参数未列全：IDENTIFY 从证据列全候选作为假设继续，不要求用户确认；不得只盯单一参数进入补环境
- 工具不可用：按 GATE-1 自动安装（`install_all.js --yes`，执行前先宣布安装目标与规模）；自动安装失败时才由用户选择提供路径或降级
