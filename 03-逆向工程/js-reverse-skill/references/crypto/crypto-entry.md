# 加密入口定位模型（四层链路）

> **触发条件**：FORENSIC_CAPTURE 需要定位加密参数生成链路时读

## 四层链路模型

定位结果必须尽量拆成四层，不要只记录一个疑似函数名。

| 层级 | 含义 | 常见证据 | 输出要求 |
|---|---|---|---|
| source 数据源 | 参与签名的输入材料 | URL、Query、Body、Cookie、localStorage、时间、随机数、指纹 | 列出字段和值来源 |
| entry 加密入口 | 直接返回 sign/token 或中间签名的函数 | 调用栈、搜索参数名、断点命中、sourcemap | 函数名、模块号、文件、行列号 |
| builder 请求构造 | 把入口结果拼到请求对象的位置 | axios/fetch 封装、SDK request 方法、拦截器 | 参数位置与写入前后的对象快照 |
| writer 请求写入 | 最终写入网络请求的位置 | fetch、XMLHttpRequest.send、setRequestHeader、URLSearchParams、cookie | 最终 API、方法、Header/Query/Body/Cookie |

**只有记录到 `writer`，才能确认"找到的函数"确实影响目标请求。**
**只有记录到 `source`，才能解释为什么 Node.js 输出和浏览器样本可能不一致。**

## 响应方向四层链路（解密/解析对称模型）

请求方向四层（source→entry→builder→writer）只覆盖"怎么把参数写进请求"。目标接口的响应体若**不是明文 JSON**（`code` 字段非 0/非 1、`data` 是二进制或乱码），必须同步还原**响应方向**四层，不能只盯着请求签名：

| 层级 | 含义 | 常见证据 |
|---|---|---|
| response 原始响应 | 网络层拿到的 body（可能 gzip/br 压缩、二进制） | trace 的 xhrNative 响应记录、capture target_hits 的 `response_body` |
| reader 读取/解码入口 | 拦截器/请求封装里把响应转成中间形式的函数 | `JSON.parse`、`charCodeAt` 循环、`atob`、`pako.inflate`、axios 拦截器 |
| decoder 解密/解压 | 真正的算法（凯撒位移 / AES / 异或 / 解压） | 调用栈、密钥/位移量参数来源 |
| parser 业务解析 | 把解密结果转成业务对象 | `JSON.parse` 之后的字段映射 |

**响应方向定位顺序与请求方向对称**：先确认"响应体是什么形态"→ 追 reader 调用链（charCodeAt/atob/inflate）→ 定位 decoder → 确认 parser。

**两条关键原则**：

1. **先看 code 分支，再判"错误/风控"**：响应有 `code`/`status` 字段且非 0 时，先在前端搜 `if(...code...){...}` 的响应拦截器分支——`code` 可能是加密容器标识（如 `code-100` 作为凯撒位移量）或加密模式标识，而非业务错误码。没看分支前不得假设"风控/拒绝"。
2. **先判 data 编码特征，再套算法**：`data` 是二进制/乱码时，先 base64 解码看 magic number（`1f8b`=gzip、`789c`=zlib），或用 `长度 mod16==0` 判 AES；`mod16≠0` 基本排除 AES-ECB，转查压缩/位移/异或。别看到源码某处有 AES 密钥串就默认 data 是 AES——密钥可能作用于别的字段。

## 三类常见架构

1. **直接写入型**：业务代码直接调用入口函数，然后把结果拼接到 Query/Header/Body。
2. **拦截器型**：axios/fetch/XHR 封装中统一补签名；业务调用栈上看不到参数名，需要看拦截器或 request wrapper。
3. **SDK 型**：第三方或站点 SDK 初始化后接管请求，入口函数可能藏在 runtime chunk、动态 chunk、WASM 或 Worker 中。

## 推荐定位顺序

1. 从成功请求样本确认 API，并先列出 Query / Header / Body / Cookie 中所有可疑加密参数作为候选假设；由取证与调用栈定位确认本次要分析的参数，不等待用户选择。
2. 在 DevTools Network 查看 Initiator，先记录请求发起文件和调用栈。
3. 对 `fetch`、`XMLHttpRequest.open/send/setRequestHeader` 设置断点或 Hook，捕获 `writer`。
4. 搜索参数名、Header 名、API path、接口封装方法名，寻找 `builder`。
5. 沿调用栈向上找返回值来源，确认 `entry`。
6. 回溯入口入参和读取的环境，整理 `source`。
7. 若源码压缩严重，结合 sourcemap、webpack module id、动态 chunk 名、运行时模块缓存定位。
8. 若存在 Worker、WASM、iframe 或 postMessage，单独建链。

## 断点与 Hook 组合

优先使用 Hook 快速缩小范围，再用断点确认源码位置：

- `fetch`：看 URL、init、Header、Body。
- `XMLHttpRequest.open/send/setRequestHeader`：看请求方法、URL、Header、Body。
- `URLSearchParams.append/set`：看 Query 参数写入。
- `FormData.append`：看表单 Body 参数写入。
- `document.cookie` setter：看 Cookie 写入。
- `localStorage/sessionStorage.getItem`：看 source 是否来自存储。
- `Date.now`、`performance.now`、`Math.random`、`crypto.getRandomValues`：看时间随机依赖。
- `Function`、`eval`、`setTimeout(string)`：看动态代码和混淆解包。

Hook 模板见 `references/hooks/hook-templates.md`。Hook 只用于授权调试和证据收集，不修改请求、不批量访问。

## 入口定位记录模板

```markdown
## 加密入口定位记录

### 目标请求
- API：
- 方法：
- 参数：
- 参数位置：

### writer 最终写入
- 类型：fetch / XHR / URLSearchParams / Header / Cookie / Body
- 文件与行列号：
- 调用栈：
- 写入前对象快照：
- 写入后对象快照：

### builder 请求构造
- 函数 / 方法：
- 所在模块：
- 输入：
- 输出：
- 是否经过拦截器：

### entry 加密入口
- 函数名 / 表达式：
- 文件 / chunk / module id：
- 行列号：
- 入参：
- 返回：
- 是否需要初始化：

### source 数据源
- URL / Query：
- Body：
- Cookie：
- localStorage / sessionStorage：
- 时间 / 随机数：
- navigator / screen / document / canvas / WebGL：
- 其他：

### 可信度
- 证据来源：Network / Hook / 断点 / sourcemap / 静态搜索 / 推断
- 是否可进入补环境：是 / 否
- 阻塞点：
```

## 进入补环境的最低条件

- 已列出所有可疑加密参数作为候选假设，并经证据（真实请求 / trace 调用栈）定位确认本次要分析的目标参数。
- 已确认目标参数在成功请求中存在。
- 已确认 `writer`，知道参数最终写入哪里。
- 已定位至少一个可调用或可追踪的 `entry`。
- 已收集入口所在 JS 文件及依赖 chunk。
- 已整理 `source` 初表，哪怕部分字段未知也要明确标记。
- 已有至少一组浏览器真实 fixtures；动态参数建议至少三组。

## 常见案例模式

### Header 签名型
信号：目标参数在 Header 中，例如 `x-sign`、`x-s`、`x-token`
- `setRequestHeader` 或 fetch init.headers 是 writer
- builder 常在 request interceptor 或 SDK request wrapper
- source 往往包含 URL、Body、时间戳、Cookie、设备标识

### Query 混淆参数型
信号：目标参数在 URL Query 中，例如 `sign`、`a_bogus`
- Hook `URLSearchParams.append/set` 与 XHR/fetch URL
- 注意 Query 排序、编码方式、空值和数组序列化
- 常依赖 UA、Referer、时间和随机数

### Body 签名型
信号：目标参数在 JSON 或表单 Body 中
- 保留原始 Body 字符串，避免 JSON 重新序列化造成差异
- 比对 Content-Type 与空格、顺序、转义
- Hook `JSON.stringify` 可辅助定位 builder

### SDK 初始化型
信号：入口函数找到了但直接调用失败，或必须先执行 init
- 查 `init`、`config`、`setConfig`、`install`、`use`、`start`
- 记录全局配置、meta 标签、script 标签参数
- 先补初始化链，再补环境对象

### 异步消息型
信号：签名结果通过 Promise、回调、Worker message 返回
- 记录消息类型和 payload
- 区分初始化消息与签名请求消息
- 用 fixtures 验证最终异步输出
