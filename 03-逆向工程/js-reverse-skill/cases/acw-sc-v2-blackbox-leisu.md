# Case：阿里云盾 acw_sc__v2 新版挑战（黑盒执行）+ accept 头 AES 签名 + 响应三重编码（leisu.com）

> 难度：★★★★（三层防护：accept 签名 + acw_sc__v2 WAF 挑战 + 响应凯撒位移/gzip/base64；无 JSVMP 字节码级难度，但 acw 新版防篡改只能黑盒）
> 还原方案：A 纯算还原（accept 签名 + 响应解密）+ B vm 沙箱执行（acw_sc__v2 挑战脚本）
> 实现语言：Node.js
> 最后验证日期：2026-08-15（实测 5/5 次 GET 返回 200 + 解密出真实业务 JSON）
> 平台类型：体育数据（leisu.com 雷速体育，移动站 m.leisu.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### 第一层：accept 头业务签名
- [x] 请求头 `accept` = `application/json, text/plain, */*;;<base64url 签名串>`（前缀 + `;;` 分隔 + 签名）
- [x] 签名算法：`AES-128-ECB-Pkcs7`（密钥 `kw@h*8gCIn$8X#df`）加密 `JSON.stringify({auth_data, source})`，`source = "m_leisu"`
- [x] `auth_data = "{unix秒+10}-{32hex uuid}-0-{md5hex}"`，`md5hex = md5("{相对路径}-{ts}-{uuid}-0-uHhANonwd4UdpzOdsUqUsnl5PjurM877")`
- [x] 混淆文件 `a247200.js`（Nuxt 动态模块，base64+RC4 字符串混淆，自举解码器 `_0x4ed1`）

### 第二层：acw_sc__v2 阿里云盾 WAF 挑战（新版）
- [x] 首次请求返回 200 + WAF 挑战页（`document.html`，非 412），响应头 `punish-loc: keepper`
- [x] 挑战页含 `renderData.l1`（动态 `arg1`，56 字符新格式）+ 混淆脚本 `(function xXRQBR(){...})`（约 18KB）
- [x] **新版特征（区别于旧版 hexXor+固定 sign）**：密钥 `vL` 由 `vy(脚本源码哈希, 'xXRQBR', '2deacaf')` 动态计算，依赖**完整函数源码 toString()** + `typeof window['xXRQBR']`
- [x] `acw_sc__v2` 格式：`1234cf0d46-<动态hex>`（10hex 固定前缀 + `-` + 动态部分），**非旧版 40 位 hex**
- [x] 挑战脚本：数组洗牌器（`while(!![])` 用 vL 参与校验打乱字符串常量池）+ 主逻辑（while-switch 控制流平坦化）
- [x] 挑战脚本通过**覆盖全局方法**（如 `Math.random` 等）传递结果，并调用**裸标识符 `reload(值)`**（全局作用域链，非 `window.reload`）

### 第三层：响应 data 三重编码
- [x] 响应形如 `{ code: 100+e, data: "<base64乱码>" }`，`code` 每次请求在 1-130 区间随机
- [x] 前端拦截器：`if(code>=1 && code<=130) return u.a(data, code-100)` —— **`code` 是加密容器标识，`code-100` 是凯撒位移量，不是错误码**
- [x] `data` = 凯撒位移(左移 `code-100`，仅 A-Z/a-z) → gzip 压缩 → base64

---

## 加密方案

### 第一层 accept（A 纯算）
```js
const crypto = require('crypto');
const KEY = 'kw@h*8gCIn$8X#df';
const SALT = 'uHhANonwd4UdpzOdsUqUsnl5PjurM877';

function md5(s){ return crypto.createHash('md5').update(s, 'utf8').digest('hex'); }
function uuid(){ return crypto.randomBytes(16).toString('hex'); }
function base64url(b){ return Buffer.from(b).toString('base64').replace(/\+/g,'-').replace(/\//g,'_').replace(/=/g,''); }
function aesEncrypt(pt, key){
  const cipher = crypto.createCipheriv('aes-128-ecb', Buffer.from(key, 'utf8'), null);
  return Buffer.concat([cipher.update(pt, 'utf8'), cipher.final()]);
}
// ts = parseInt(Date.now()/1000 + 10)
// u = uuid(); md = md5(`${pathname}-${ts}-${u}-0-${SALT}`)
// auth_data = `${ts}-${u}-0-${md}`; source = 'm_leisu'
// accept = `application/json, text/plain, */*;;${base64url(aesEncrypt(JSON.stringify({auth_data, source}), KEY))}`
```
> 验证技巧：**用浏览器抓到的真实签名反解（AES 解密）确认密钥与明文结构**，比正向猜测快得多。真实签名反解明文 `{"auth_data":"1786718876-...","source":"m_leisu"}` 与算法完全一致。

### 第二层 acw_sc__v2（B vm 黑盒执行）
- 密钥 `vL` 依赖**函数源码哈希 + `typeof window['xXRQBR']`**（防篡改），不可反混淆，只能黑盒执行挑战脚本。
- 最小环境：`window.arg1`（挑战页 `renderData.l1`）+ `typeof window['xXRQBR'] === 'undefined'` + `navigator.webdriver === false` + Math/Date/JSON 原生 + 顶层定义 `reload` 函数捕获输出。
- 隔离验证 vL=239 后，替换 `vr()` 返回固定 vL 避免源码修改导致洗牌器死循环。

### 第三层响应解密（A 纯算）
```js
// data --base64--> gzip解压 --> 凯撒位移(右移 code-100, 仅字母) --> JSON.parse
function caesarShift(s, n){ // 右移：字母 - n
  return s.replace(/[A-Za-z]/g, c => {
    const base = c <= 'Z' ? 65 : 97;
    return String.fromCharCode((c.charCodeAt(0) - base - n + 26) % 26 + base);
  });
}
function decryptData(data, code){
  const buf = zlib.gunzipSync(Buffer.from(data, 'base64'));  // 注意是 gzip 不是 zlib
  return JSON.parse(caesarShift(buf.toString('utf8'), code - 100));
}
```
> **关键**：凯撒位移只动字母，base64 的 `+ / =` 与数字不动，位移后仍是合法 base64 字符串；解压用 `gunzipSync`（`1f8b` 头）而非 `inflateSync`。

---

## 踩坑记录（acw 黑盒执行坑点清单，IMPLEMENT 必查）

| # | 坑 | 现象 | 正确做法 |
|---|---|---|---|
| 1 | 反混淆/改源码 | 注入 `__dlog` 日志改变函数源码 → vL 算错 → 解码全空 | **不改源码观测**：用 Proxy 包装环境对象记录属性访问；替换 `vr()` 返回已验证的固定 vL=239 |
| 2 | NFE toString 不一致 | 提取片段拼成 NFE 的 `toString()` 与页面不一致 → vL 错 | NFE 的 `toString()` 必须与页面一致，剥离外层括号与调用括号 |
| 3 | 隔离组装缺洗牌器 | 单独提取主逻辑执行，解码表 URI malformed | **数组洗牌器必须执行**，否则字符串常量池错位；洗牌器与主逻辑共享分组括号无法单独提取 → 直接黑盒执行完整脚本 |
| 4 | 缺 arg1 | 脚本访问 `window.arg1`（页面全局变量）undefined → 输出空 | 挑战页 `renderData.l1` 提取 arg1 后注入沙箱 |
| 5 | 用 window.reload 捕获 | 脚本调用**裸标识符 `reload`**（非 `window.reload`）→ 拦截不到 | 在 sandbox **顶层作用域**定义 `reload(值)` 函数捕获输出 |
| 6 | 找 return 输出 | 脚本静默执行无 return | 挑战脚本通过**覆盖全局方法**（Math.random 等）传递结果，hook document.cookie / 全局方法捕获 |
| 7 | 缺 jQuery | 脚本 1ms 超时/死循环 | 提供 jQuery mock（挑战脚本通过魔改版 jQuery 的 wU 回调存结果） |
| 8 | 重试只带 acw_sc__v2 | 仍返回挑战页 | URL 必须追加 `&acw_sc__v2=<值>&alichlgref=<referrer>`，**alichlgref 不可缺** |
| 9 | 55 vs 50 长度纠结 | 沙箱生成动态 55 字符，真实浏览器 50 字符，纠结 20+ 步 | **先验证是否影响服务端判定**：动态部分每次不同、格式一致即可通过；差异源于 RuyiTrace Proxy 改变 `typeof window['xXRQBR']` 与 Array.fill 覆盖分支，服务端不严格校验长度 |
| 10 | 看到 AES 密钥就套 data | 搜到 `i7!4cH3!IjgE8Rf0` 密钥就假设 data 是 AES，mod16=14 截断硬解失败 | **先查前端 code 分支 + data 编码特征**（见反模式 10）；该密钥作用于别的字段，data 是凯撒+gzip+base64 |

---

## 可验证事实清单（经验资产，同站升级时逐条核对）

1. `acw_sc__v2` 格式 `1234cf0d46-<动态hex>`，前缀 10hex 固定
2. 挑战页 arg1 动态（56 字符新格式，非旧版 40 hex）
3. 密钥 `vL=239` 是唯一收敛点（150-250 扫描其余全死循环）
4. accept 签名密钥 `kw@h*8gCIn$8X#df`，md5 盐 `uHhANonwd4UdpzOdsUqUsnl5PjurM877`
5. accept 明文 `source = "m_leisu"`
6. 响应 `code` 区间 1-130，`code-100` = 凯撒位移量
7. data 解密后业务 JSON 435696 字符（team_info 含 team/transfers/lineups/history 等 13 字段）
8. data = 凯撒位移(仅字母) + gzip(`1f8b`) + base64，ECB 无 IV 故相同请求参数 data 固定
9. 无 accept 签名 → 204 静默拒绝；带签名 → 200 挑战页；带签名+acw+alichlgref → 200 + 加密容器
10. 全链路在纯 Node.js（`https` 直连）跑通，TLS 指纹不参与校验（curl_cffi Firefox 也 114，排除 TLS 因素）

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/crypto/crypto-entry.md` | 响应方向四层 response→reader→decoder→parser（本案例 code 分支 + 编码特征判型） |
| `references/crypto/crypto-patterns.md` | 响应体编码识别表 + mod16≠0 负向判型 |
| `references/workflow/common-pitfalls.md` | 反模式 10（code 非0 + data 乱码误判风控） |
| `references/network/session-chain.md` | 挑战重放链（WAF 挑战 → acw 生成 → alichlgref 重试） |
| `references/network/ip-risk-control.md` | 风控识别决策树（先查 code 分支再判风控） |
