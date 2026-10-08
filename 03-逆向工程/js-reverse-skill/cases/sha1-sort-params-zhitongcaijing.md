# Case：纯算还原 SHA1 参数排序签名（智通财经 H5）

> 难度：★
> 还原方案：A 纯算还原
> 实现语言：Node.js
> 最后验证日期：2026-07-08
> 平台类型：智通财经（m.zhitongcaijing.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### JS 特征
- [x] Vue 构建，`app.js` + `chunk-vendors.js` 分包，无混淆无压缩（可直接 Grep）
- [x] 模块 `1ae8` 请求构造器：GET `o+="&token="+u["a"].hex_sha1(o)` / POST `a.token=u["a"].hex_sha1(s()(a))`
- [x] `f(t)` = `Object.keys(t).sort()` 按字母序重排参数对象
- [x] `s()` = chunk-vendors 模块 `cfd4` 序列化器：`for in` + `encodeURIComponent` + `&` 连接
- [x] `u["a"].hex_sha1` = 标准 SHA1（hex 输出），无非标改动

### 参数特征
- [x] URL query 中的 `token`，40 位 hex（SHA1 标准长度）
- [x] 每次值不同（随业务参数变化），但同参数可复现
- [x] GET 追加到 query 末尾，POST 写入 body.token

### 请求特征
- [x] 缺/错 token → 接口返回业务错误码（非 403/412）
- [x] 无 412 循环、无 webmssdk、无 JSVMP
- [x] 页面正常加载，无反爬挑战

### 混淆类型
- [x] 无混淆（可直接静态读取算法逻辑）

---

## 加密方案

- **路径**：A 纯算还原
- **框架**：不使用（Node.js crypto 原生模块）
- **TLS 客户端**：Node.js fetch
- **核心思路**：Grep 定位 `hex_sha1` 调用 → 确认是标准 SHA1 → 纯 Node.js 复现参数排序 + 序列化 + SHA1

### 算法细节

**token = SHA1(序列化串)**，算法放之 GET/POST 皆准：

1. 合并基础参数 `{__mode__:"history", tradition_chinese:"0", access_token:"", language:"zh-cn"}` 与业务参数
2. `f(t)` = `Object.keys(t).sort()` 按键名字母序重排
3. `s()(a)` 序列化为 `encodeURIComponent(k)=encodeURIComponent(v)` 以 `&` 连接（对象递归为 `k[v]`）
4. `token = SHA1(上述串)` → 40 位 hex
5. GET 追加 `&token=...` 到 query；POST 写入 body.token

**签名公式**：`token = hex_sha1(serialize(sortKeys(merge(baseParams, businessParams))))`

---

## 方案方向

纯静态分析：下载 JS + Grep 定位签名函数 → 确认标准 SHA1 → Node.js `crypto.createHash('sha1')` 复现。

无需 vm 沙箱：算法是标准 SHA1，可直接用 crypto 复现。
无需补环境：无环境依赖、无混淆、无 JSVMP。

## 标准流程

### FORENSIC_CAPTURE → TRACE_CAPTURE：定位 + 提取

```
1. 下载 app.js + chunk-vendors.js（curl -k，无混淆可直接读）
2. Grep "token=" → 定位 app.js 模块 1ae8 请求构造器
3. Grep "hex_sha1" → 定位 u["a"].hex_sha1 调用
4. Grep "hex_sha1" 定义 → 确认是标准 SHA1（与 Python hashlib.sha1 比对吻合）
5. Grep "Object.keys.*sort" → 定位 f(t) 排序函数
6. Grep "encodeURIComponent" → 定位 cfd4 序列化器
```

### TRACE_ANALYZE：纯算复现

```javascript
const crypto = require('crypto');

function serialize(obj, prefix) {
  const out = [];
  for (const key in obj) {
    if (!Object.prototype.hasOwnProperty.call(obj, key)) continue;
    const val = obj[key];
    const encKey = encodeURIComponent(key);
    const newPrefix = prefix ? `${prefix}[${encKey}]` : encKey;
    if (val !== null && typeof val === 'object') {
      out.push(serialize(val, newPrefix));
    } else {
      out.push(`${newPrefix}=${encodeURIComponent(val === undefined || val === null ? '' : val)}`);
    }
  }
  return out.join('&');
}

function sortKeys(obj) {
  const sorted = {};
  for (const k of Object.keys(obj).sort()) sorted[k] = obj[k];
  return sorted;
}

function hexSha1(str) {
  return crypto.createHash('sha1').update(str, 'utf8').digest('hex');
}

function signGet(params) {
  const base = { __mode__: 'history', tradition_chinese: '0', access_token: '', language: 'zh-cn' };
  const merged = Object.assign({}, base, params);
  const sorted = sortKeys(merged);
  const query = serialize(sorted);
  const token = hexSha1(query);
  return query + '&token=' + token;
}
```

### IMPLEMENT：验证

```
1. 已知 token 928079f344b3d4b20faf84ed7afc989a774ab328（page=1 用例）复现成功
2. 对真实接口连续 5 页请求均返回 200 + 正确 JSON
3. 签名每次合法且随参数变化
```

## 踩坑记录

| # | 坑 | 现象 | 解决方法 |
|---|---|------|---------|
| 1 | 透明代理自签 CA | Node.js/Python TLS 校验失败 | `NODE_TLS_REJECT_UNAUTHORIZED=0`（仅本地验证用） |
| 2 | trace 取证动态调试受阻 | addons.mozilla.org 返回 451 + playwright 版本不兼容 | 改用纯静态分析（curl + Grep），结果等价 |
| 3 | 序列化对象递归格式 | 嵌套对象需序列化为 `k[v]` 格式 | 参照 cfd4 序列化器实现递归 serialize |

## 边界判断

```
算法是标准 SHA1 吗？
  ├─ 是 → 纯算还原（本案例）
  └─ 否（自定义变种 / 混淆不可读）
      ├─ 能 vm 执行 → vm 沙箱
      └─ 需完整环境 → 补环境
```

## 可验证事实清单（经验资产）

1. token 长度 40 字符（SHA1 hex 标准长度）
2. 算法类型：标准 SHA1（与 `hashlib.sha1` 比对完全吻合）
3. 算法依赖的环境属性：无（纯计算）
4. 签名输入：`serialize(sortKeys(merge(baseParams, businessParams)))`
5. 基础参数：`{__mode__:"history", tradition_chinese:"0", access_token:"", language:"zh-cn"}`
6. GET 追加 `&token=...`，POST 写入 `body.token`
7. 已知 token `928079f344b3d4b20faf84ed7afc989a774ab328`（page=1）可复现
8. ≥5 次请求签名稳定通过

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/trace-flow.md` | 统一日志驱动逆向流程 |
| `references/workflow/decision-tree.md` | 题型判定边界 |
| `references/crypto/algorithm-families.md` | SHA1 标准算法识别 |
| `cases/sm2-sm4-sm3-guomi-jobonline.md` | 同类标准算法案例（国密） |
