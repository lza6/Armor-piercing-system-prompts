# Case：纯算还原 SHA1 参数排序签名 + appSignKey 静态密钥（丁香园 BBS）

> 难度：★
> 还原方案：A 纯算还原
> 实现语言：Node.js
> 最后验证日期：2026-08-16
> 平台类型：丁香园 BBS（www.dxy.cn / bbs/newweb/*）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] umi webpack 构建，`umi.bundle.js`（3.7MB 单行压缩）+ `p__*.async.*.js` 路由 chunk
- [x] 请求链：页面组件 → 96132（参数合并）→ 77773（去 /pc 前缀）→ 7295.U（请求封装）→ 88086.sign（参数组装 + SHA1）+ 4070.bl（签名密钥）
- [x] 模块 88086 内嵌完整 crypto-js，SHA1 初始常量 `1732584193` 可定位
- [x] 签名密钥是**页面级静态常量**（模块 4070 导出 `bl`），URL 正则区分测试/生产，非动态下发

### 参数特征
- [x] query 四件套：`serverTimestamp` + `timestamp` + `noncestr` + `sign`（40 位 hex，SHA1 标准长度）
- [x] `noncestr` 为 8 位纯数字随机串（`0123456789` 中随机取 8 位）
- [x] `timestamp = Date.now() + 时间偏移`（偏移在首个带非零 serverTimestamp 请求时计算并缓存）
- [x] `serverTimestamp` 来自前置接口 `/bbs/newweb/sys/time-millis/info`（请求时带 `serverTimestamp=0`），返回 `data` 为服务器毫秒时间戳

### 请求特征
- [x] 业务接口缺 `Referer` 头返回 `TD0300000000 非法请求`（**不是签名错误**，易误判）
- [x] 无 412 循环、无 webmssdk、无 JSVMP
- [x] 匿名可访问，无需登录 Cookie
- [x] 页面正常加载，无反爬挑战

### 混淆类型
- [x] 压缩但无混淆（单行压缩代码，可用 search_js.js 定位模块边界，无需反混淆）

---

## 加密方案

- **路径**：A 纯算还原
- **框架**：不使用（Node.js crypto 原生模块）
- **TLS 客户端**：Node.js https（keepAlive Agent 复用）
- **核心思路**：RuyiTrace 定位 `noncestr` 参数写入点 → webpack 模块链定位 sign 函数与密钥 → 纯 Node.js 复现

### 算法细节

**sign = SHA1(拼接串)**，适用于所有 `/bbs/newweb/*` 接口：

1. 参数集 = 业务参数 ∪ `{serverTimestamp, timestamp, noncestr}`
2. 过滤值为 `undefined` 或空串的参数
3. 追加固定键 `appSignKey`（值为签名密钥）
4. 全部键按字典序（JS `sort()`）排序
5. 拼接 `k=v&k=v` 字符串
6. `sign = SHA1(字符串)` → 40 位 hex

**签名公式**：`sign = hex_sha1(join('&', sort(merge(params, {appSignKey: secret}))))`

**动态参数来源**：

| 参数 | 来源 |
|---|---|
| `serverTimestamp` | 先请求 `/sys/time-millis/info`（serverTimestamp=0）获取服务器毫秒时间 |
| `timestamp` | `Date.now() + timeOffset`，偏移在首次带非零 serverTimestamp 请求时计算并缓存 |
| `noncestr` | 8 位随机数字串 |
| `sign` | 上述 SHA1 算法 |

**签名密钥**：模块 4070 导出的 `bl` 常量（生产环境固定，从 bundle 提取一次即可，无需动态获取）。

---

## 方案方向

纯静态分析：RuyiTrace 定位写入点 → webpack 模块链逐步提取 → 确认标准 SHA1 → Node.js `crypto.createHash('sha1')` 复现。

无需 vm 沙箱：算法是标准 SHA1，可直接用 crypto 复现。
无需补环境：无环境依赖、无混淆、无 JSVMP。

## 标准流程

### FORENSIC_CAPTURE → TRACE_CAPTURE：定位 + 提取

```
1. forensic_ruyipage.py --targets "paid-post/page,time-millis" 抓取目标接口（time-millis 是配置来源接口，必须加 --targets 才留响应体）
2. RuyiTrace 采集，--target-signal 传 noncestr（参数写入点，trace 不记录密钥常量名 appSignKey）
3. 页面 chunk p__PaidList.async.*.js 中 apiMockUrl 前缀 `62060555115a191200a823e7` 是业务标识（不是密钥）
4. 请求链定位：96132 → 77773 → 7295 → 88086.sign + 4070.bl
5. 模块 4070 导出 bl 常量 = 生产签名密钥
6. 模块 88086 是 crypto-js，SHA1 初始常量 1732584193 确认算法
```

### TRACE_ANALYZE：纯算复现

```javascript
const crypto = require('crypto');

const SIGN_KEY = '4bTogwpz7RzNO2VTFtW7zcfRkAE97ox6ZSgcQi7FgYdqrHqKB7aGqEZ4o7yssa2aEXoV3bQwh12FFgVNlpyYk2Yjm9d2EZGeGu3';

function genNoncestr() {
  const digits = '0123456789';
  let result = '';
  for (let i = 0; i < 8; i++) {
    result += digits[Math.floor(Math.random() * digits.length)];
  }
  return result;
}

function sign(params, secretKey = SIGN_KEY) {
  const signedParams = { ...params };
  signedParams.noncestr = genNoncestr();
  signedParams.timestamp = Date.now();
  const keys = Object.keys(signedParams)
    .filter((key) => signedParams[key] !== undefined && signedParams[key] !== '')
    .concat('appSignKey')
    .sort();
  const signStr = keys
    .map((key) => `${key}=${key === 'appSignKey' ? secretKey : signedParams[key]}`)
    .join('&');
  signedParams.sign = crypto.createHash('sha1').update(signStr).digest('hex');
  return signedParams;
}
```

### IMPLEMENT：验证

```
1. 3 个独立真实样本（2 个抓包 + 1 个 trace）SHA1 复算全部一致
2. 对真实接口 5 次请求（不同页/尺寸）均返回 200 + 正确业务数据（total=1293），无需登录 Cookie
3. 动态参数随时间、随机数正确变化
```

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|---|---|
| 1 | Referer 硬校验 | 业务接口缺失 Referer 返回 `TD0300000000 非法请求`，易误判为签名错误 | 补 `Referer: https://www.dxy.cn/bbs/newweb/pc/paid-list`；两者都返回 200，靠 body code 区分 |
| 2 | capture.json 无响应体 | time-millis 接口响应体在 capture.json 里找不到（纯元数据） | 配置来源接口必须加进 `--targets`，响应体才进 target-hits.json |
| 3 | --target-signal 传密钥名 | 传 `appSignKey` 在 trace 中 0 次命中（trace 记录运行时写入点，不记录密钥字面量） | 应传参数写入点 `noncestr`（98 次命中）；trace 未覆盖目标接口 URL 字面量属预期，需声明豁免 |
| 4 | 单行压缩 bundle 切割 | 误匹配更大 ID 的子串，模块边界切割错误 | 用正则精确锚定模块头 `n(7295)` 等；大文件用 search_js.js / Python 提取 |

## 边界判断

```
算法是标准 SHA1 吗？
  ├─ 是 → 纯算还原（本案例）
  └─ 否（自定义变种 / 混淆不可读）
      ├─ 能 vm 执行 → vm 沙箱
      └─ 需完整环境 → 补环境
```

## 可验证事实清单（经验资产）

1. sign 长度 40 字符（SHA1 hex 标准长度）
2. 算法类型：标准 SHA1（crypto-js 内嵌，初始常量 1732584193 确认）
3. 算法依赖的环境属性：无（纯计算）
4. 签名输入：`sort(merge(params, {appSignKey: secret}))` 后 `k=v&k=v` 拼接
5. 动态参数：`serverTimestamp`（前置 time-millis 接口）/ `timestamp`（Date.now()+偏移）/ `noncestr`（8 位数字）
6. 签名密钥为页面级静态常量（模块 4070 `bl`），URL 正则区分测试/生产
7. 3 个独立样本 SHA1 复算一致 + ≥5 次真实请求稳定通过
8. 业务接口必须带 `Referer`，缺失返回 `TD0300000000 非法请求`（非签名错误）

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | 统一日志驱动逆向流程 |
| `references/workflow/decision-tree.md` | 题型判定边界 |
| `references/crypto/algorithm-families.md` | SHA1 标准算法识别 |
| `cases/sha1-sort-params-zhitongcaijing.md` | 同算法族（SHA1 + 参数排序，无前置时间接口） |
