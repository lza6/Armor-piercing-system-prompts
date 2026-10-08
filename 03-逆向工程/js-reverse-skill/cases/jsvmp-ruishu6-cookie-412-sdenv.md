# Case：JSVMP RS6（412 挑战 Cookie）+ 业务层 md5 签名 + GET 接口（nmpa.gov.cn）

> 难度：★★★★★（双层防护：RS6 Cookie 包裹 + 业务 sign 校验，任一层错都 412/签名失败）
> 还原方案：D 补环境（sdenv 生成 RS6 Cookie）+ A 纯算法还原（业务 sign，Node crypto）
> 实现语言：Node.js（运行在 WSL2，因为原生模块需 Linux 编译）
> 最后验证日期：2026-07-13（实测 6/6 次 GET 返回 200 + 真实 JSON）
> 平台类型：政府监管类（nmpa.gov.cn 国家药监局数据查询）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

### 第一层：RS6 412 挑战（Cookie 生成）
- [x] 单文件 200KB+，变量名 `_$` 前缀加 2-3 位随机字母（如 `_$bV`、`_$ku`），入口 `if($_ts.cd){`，函数名每次请求动态变化
- [x] 页面底部 `<script>_$xx();</script>` 入口调用，函数名每次不同
- [x] HTML 含 `<meta id="固定ID" content="动态token">`，content 约 100 字符每次变化
- [x] 内联脚本设 `$_ts` 全局（含 `nsd` 数字种子 + `cd` 约 1800 字符配置串）
- [x] Cookie 名格式 `NfBCSins2OywS` / `NfBCSins2OywO` / `NfBCSins2OywP`（服务端 Set-Cookie）+ 客户端 JS 生成项
- [x] 首次请求 HTTP 412（非标准），响应体为精简 HTML（meta + `$_ts` 内联配置 + 外部 JS 引用 + 入口函数）
- [x] 412 的 Set-Cookie 同时下发 `acw_tc` 和 `NfBCSins2OywS`（HttpOnly，过期 10 年）
- [x] 成功请求需携带 `acw_tc` + `NfBCSins2OywS`（服务端）+ 客户端生成项共 4 个 Cookie
- [x] RS JS 路径含随机目录名，版本号后缀（如 `.e17ed02.js`）一段时间固定
- [x] JSVMP 用内部函数表+直接调用（非 `Function.prototype.apply/call`），标准 JSVMP Hook 工具拦截不到
- [x] `$_ts` 在 JS 执行后被清理，运行时无法 console 访问
- [x] 检测 `typeof document.all === "undefined"`（浏览器特有，`document.all` 可调用但 typeof 为 undefined）——纯 jsdom 返回 `"object"` 会卡死

### 第二层：业务接口 sign 校验（必过 412 之后）
- [x] 业务 JS（`ajax.js`）经 jsjiami.com.v6 混淆（自举解密 `_0xdfc7`），含 `appSecret` / `jsonMD5ToStr` / `getSign` / `pajax` 命名空间
- [x] **请求方法是 GET，不是 POST**；参数序列化为 query string（`itemId=&isSenior=N&searchValue=&pageNum=1&pageSize=15&timestamp=<T>`）
- [x] `sign = jsonMD5ToStr(getSign(params))`，`timestamp` 取**服务器时间**（见下）
- [x] headers 带 `{ token, timestamp, sign }`（`token` 在本站上下文为 `"false"`）
- [x] `appSecret = "nmpasecret2020"`（写死在 ajax.js 里，裸 jsdom 可提取）

---

## 两层防护结构（关键认知）

```
浏览器请求 search 接口
   │
   ├─ 第 0 层：WAF / RS6 412 挑战
   │     → 需先过 412 生成 RS6 Cookie（sdenv 补环境）
   │     → 任何业务请求都必须带这套 Cookie，否则直接 412 重挑战
   │
   └─ 第 1 层：业务 sign 校验（200 之后才校验）
         → GET + query string + 正确 sign + 合法 itemId/keyword
         → 错则 200 + {"code":500,"message":"对不起，请求签名验证失败!"}
```

> **两个根因都曾卡死我们**：① 一直用 POST，真实是 GET；② timestamp 用 `Date.now()`/`NaN` 都失败，真实用**服务器时间**。这两点不靠猜，靠"加载真实 ajax.js 跑一遍抓真值"一次性锁定（见下方"已验证定位路径"）。

---

## 加密方案

### 第一层（RS6 Cookie，算法不需复刻，sdenv 直接生成）
- 算法链：Huffman → XOR → AES-128-CBC → CRC32 → AES-128-CBC → Base64（URL-safe）
- 密钥由 `$_ts.cd` 配置串 XOR offset 推导，每次 412 动态变化
- **结论**：不要复刻，用 sdenv 在 Node 里真实执行 RS JS 生成 Cookie（见还原模板）

### 第二层（业务 sign，纯算法，必须复刻）
```js
const crypto = require('crypto');
const md5 = s => crypto.createHash('md5').update(s, 'utf8').digest('hex');
const appSecret = 'nmpasecret2020';

// 过滤空值 + 按 key 排序拼接为 "k=v&k=v"
function getSign(o) {
  const a = [];
  for (const k in o) {
    const v = o[k];
    if (v !== '' && v !== undefined && v != null) a.push(k + '=' + v);
  }
  return a.sort().join('&');   // a.sort() 默认按字符串排序 key
}

// 字符串版：拼 appSecret → encodeURIComponent → 4 个防御性 replace → md5
function jsonMD5ToStr(str) {
  let s = str + '&' + appSecret;
  s = encodeURIComponent(s);
  s = s.replace(/!/g, '%21').replace(/\(/g, '%28').replace(/\)/g, '%29').replace(/~/g, '%7E');
  return md5(s);
}

// 调用链：sign = jsonMD5ToStr(getSign(params))
// getSign 接受【对象】，返回排序字符串；jsonMD5ToStr 接受【字符串】
```

**签名覆盖的字段集合（真值）**：
```js
const params = {
  itemId: '<真实分类id，如 ff80808183cad75001840881f848179f>', // 境内生产药品
  isSenior: 'N',
  searchValue: '<关键词，不能为空>',
  pageNum: 1,
  pageSize: 15,
  timestamp: T,   // 服务器时间（见下）
};
const sign = jsonMD5ToStr(getSign(params));  // 空值已被 getSign 过滤
```

**timestamp 来源（关键，曾反复失败）**：
- 页面真实逻辑 `getdate()` = 同步 XHR 读 `<itemFileUrl>/config/DATE.json?date=<now>`，返回 `new Date(responseText).getTime()`。
- 但该站 `DATE.json` 当前**返回空 `[]`** → `new Date("").getTime()` = `NaN`。
- 实测：用 `Date.now()`、`NaN` 作 timestamp 均"签名验证失败"；用**服务器自身时钟时间 T**（来自任意响应的 `Date` 响应头 `Date.parse(r.headers.get('date'))`）作 T，sign 即被服务端接受。
- **工程做法**：每次请求前用一次轻量请求拿服务器 `Date` 头作为 T，且 query string 里的 `timestamp` 与参与签名的 `timestamp` 必须为**同一个 T**。

**合法查询参数（否则"请求参数不能为空"）**：
- `itemId` 必须是**真实分类 id**，取自 `/datasearch/config/NMPA_DATA.json`（如 `ff80808183cad75001840881f848179f` 境内生产药品、`ff80808183cad7500183cb66fe690285` 境内医疗器械等）。
- `searchValue` 不能为空。
- 注：页面另有高级检索变体用 `itemIds`（复数、逗号拼接），但标准 `queryList` 用**单数 `itemId` + 真实分类 id**。

---

## 已验证定位路径（本次实际走的：WSL2 内 sdenv + 裸 jsdom 插桩）

> **关于 ruyipage / RuyiTrace（重要澄清）**：本 case **并非因为工具不行而没用**——它们完全能直接定位加密参数与 `hasTokenGet → getSign → jsonMD5ToStr` 调用链（这正是其设计目的，也是 skill 第一原则"以 RuyiTrace NDJSON 为优先证据源"的默认路径）。真实 Firefox 还能自然过 RS6 的 412 挑战。
>
> 本次未用是**环境可用性**决定，不是能力问题：
> ① RuyiTrace 内核 + 定制 trace Firefox 当时未安装（需 GitHub 下载，skill 自身提示常因代理/自签 CA 失败）；
> ② 实际逆向在 **WSL2（node）** 中执行（sdenv 原生模块在此编译），而 `ruyiPage` 是 Windows python 包，跨环境驱动浏览器未搭建。
>
> 因此本 case 走的是**降级路径**：sdenv 过 RS6（也是 skill 对 `document.all` 检测的推荐升级项）+ 裸 jsdom 加载真实 `ajax.js` 插桩抓真值。**当你所在环境已装好 ruyipage + RuyiTrace 时，应优先走 skill 默认取证链**；本 jsdom 路径仅作为"浏览器工具链不可用时"的等价备选。

**步骤 1：WSL2 搭建 sdenv 环境（生成 RS6 Cookie 必需）**
```
# WSL2 Ubuntu：node v20.20.2（managed 或系统均可），需 build-essential + canvas/pango 系统库
sudo apt update && sudo apt install -y build-essential
# 安装 sdenv（tgz）+ jsdom，走 npmmirror 镜像（GitHub 直连易失败）
cd /home/<user>/nmpa-test
npm init -y
npm install /path/sdenv-1.1.3.tgz jsdom --registry=https://registry.npmmirror.com
# 原生模块 documentAll.node 会被 node-gyp 编译（约 17KB），需 build-essential
export NODE_PATH=/home/<user>/nmpa-test/node_modules
# 旧版 OpenSSL 站点需降级协商（注意变量名是 RENEGOTIATION，曾误写为 RENEGOTIATION）
export OPENSSL_LEGACY_RENEGOTIATION=1
export NODE_TLS_REJECT_UNAUTHORIZED=0
```

**步骤 2：sdenv 生成 RS6 Cookie**
```js
const { jsdomFromUrl } = require('sdenv');
async function genCookie(homeUrl, UA) {
  const dom = await jsdomFromUrl(homeUrl, {
    userAgent: UA,
    consoleConfig: { error: () => {} },   // 吞掉 RS 内部噪音
  });
  // 监听 sdenv:exit（location.replace/assign 触发）→ Cookie 落入 cookieJar
  await new Promise((res) => {
    dom.window.addEventListener('sdenv:exit', () => res());
    setTimeout(res, 60000);  // 兜底超时
  });
  const ck = dom.cookieJar.getCookieStringSync(homeUrl);
  dom.window.close();
  return ck;  // 形如 acw_tc=...; NfBCSins2OywS=...; NfBCSins2OywO=...; NfBCSins2OywP=...
}
```

**步骤 3：裸 jsdom 提取业务 sign 函数（sdenv 自带 jsdom 不执行内联 script，必须用裸 jsdom）**
```js
const { JSDOM } = require('jsdom');
const fs = require('fs');
const html = `<!doctype html><html><body>
  <script>${fs.readFileSync('dl/md5.js','utf8')}</script>
  <script>${fs.readFileSync('dl/base64.js','utf8')}</script>
  <script>${fs.readFileSync('dl/ajax.js','utf8')}</script>
</body></html>`;
const dom = new JSDOM(html, {
  runScripts: 'dangerously',
  beforeParse(w) {
    // 用 try/catch + defineProperty 补浏览器特征（jsdom 里 navigator 属性只读）
    try { Object.defineProperty(w.navigator, 'userAgent', { value: UA, configurable: true }); } catch(e){}
    try { Object.defineProperty(w.navigator, 'platform', { value: 'Win32', configurable: true }); } catch(e){}
    w.axios = { create: () => ({ interceptors: { request: { use(){} }, get(){}, post(){} } }) };
    w.jQuery = () => ({ cookie: () => '' });
  },
});
const w = dom.window;
// 提取真值
console.log('appSecret =', w.appSecret);          // "nmpasecret2020"
console.log('getSign    =', typeof w.getSign);    // function
console.log('jsonMD5..  =', typeof w.jsonMD5ToStr);// function
```

**步骤 4（核心突破）：加载真实 ajax.js，插桩抓"真实发出的请求"——不猜字段**
```js
// 在裸 jsdom 里加载真实 ajax.js，用假 axios 重放真实请求拦截器，
// 覆盖 pajax.getdate 绕过同步 XHR，WRAP getSign/jsonMD5ToStr 记录入参出参：
let getSignInput, getSignOutput, signInput, signOutput;
const o1 = w.getSign;
w.getSign = function () { getSignInput = JSON.stringify(arguments[0]); getSignOutput = o1.apply(this, arguments); return getSignOutput; };
const o2 = w.jsonMD5ToStr;
w.jsonMD5ToStr = function () { signInput = arguments[0]; signOutput = o2.apply(this, arguments); return signOutput; };
// 覆盖 getdate（页面真实逻辑走同步 XHR 读 DATE.json，这里直接给固定 T）
w.pajax.getdate = () => T_FIXED;
// 假 axios：service({url,method,headers,params}) → 记录真实 method/query/headers
w.axios.create = () => ({
  interceptors: { request: { use: (fn) => { intercept = fn; } },
  get(url, cfg) { const r = intercept({ ...cfg, url, method: 'get' }); record(r); return Promise.resolve({ status:200, data:{} }); },
});
// 触发：w.pajax.hasTokenGet(w.api.queryList, {itemId, isSenior:'N', searchValue, pageNum, pageSize})
// → 直接得到：method='GET'、params 进 query string、headers={token,timestamp,sign}
```
> **这一招一次性锁定两个根因**：① method 是 GET 不是 POST；② timestamp 来自 `getdate()`（服务器时间）不是 `Date.now()`。之前反复失败就是因为这两点猜错。

---

## 还原代码模板

### 完整可运行骨架（final.js 核心）
```js
const crypto = require('crypto');
const https = require('https');
const { jsdomFromUrl } = require('sdenv');

const appSecret = 'nmpasecret2020';
const md5 = s => crypto.createHash('md5').update(s, 'utf8').digest('hex');
const UA = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36';
const HOME = 'https://www.nmpa.gov.cn/datasearch/search-result.html';
const API  = 'https://www.nmpa.gov.cn/datasearch/data/nmpadata/search';

function getSign(o){const a=[];for(const k in o){const v=o[k];if(v!==''&&v!==undefined&&v!=null)a.push(k+'='+v);}return a.sort().join('&');}
function jsonMD5ToStr(str){let s=str+'&'+appSecret;s=encodeURIComponent(s);s=s.replace(/!/g,'%21').replace(/\(/g,'%28').replace(/\)/g,'%29').replace(/~/g,'%7E');return md5(s);}

async function genCookie(){ /* 见步骤 2 */ }
async function serverTime(ck){ // 轻量请求拿 Date 头
  return new Promise((res)=>{ https.get({hostname:'www.nmpa.gov.cn',path:'/datasearch/search-result.html',headers:{'User-Agent':UA,'Cookie':ck,'Referer':'https://www.nmpa.gov.cn/datasearch/'}}, r=>{res(Date.parse(r.headers['date']));r.resume();}).on('error',()=>res(NaN)); });
}

async function oneSearch(ck, itemId, keyword){
  const T = await serverTime(ck);
  const params = { itemId, isSenior:'N', searchValue:keyword, pageNum:1, pageSize:15, timestamp:T };
  const sign = jsonMD5ToStr(getSign(params));
  const qs = new URLSearchParams(params).toString();   // URL 顺序无关，服务端按对象重排
  return new Promise((res)=>{
    https.get({hostname:'www.nmpa.gov.cn',path:`/datasearch/data/nmpadata/search?${qs}`,
      headers:{ 'User-Agent':UA, 'Cookie':ck, 'Referer':'https://www.nmpa.gov.cn/datasearch/',
                'token':'false', 'timestamp':String(T), 'sign':sign }},
      r=>{ let b=''; r.on('data',c=>b+=c); r.on('end',()=>res({status:r.statusCode,body:b})); }).on('error',e=>res({status:0,body:String(e)}));
  });
}
```

---

## 踩坑记录（按实际踩过的排，含已纠正的误判）

| # | 坑 | 现象 | 正确做法 |
|---|------|------|---------|
| 1 | 误判反爬类型 | webFetch 直连 412，猜是 JSL（`__jsl_clearance_s`） | 看 412 响应体 + Set-Cookie：`NfBCSins2OywS` 是 RS 特征 |
| 2 | **用 POST 而非 GET** | 所有 `test_search1-4` 都"签名验证失败"/412 | 真实是 **GET**，参数进 query string；插桩真实 ajax.js 一眼锁定 |
| 3 | **timestamp 源错** | `Date.now()`、`NaN` 都失败，且失败信息完全相同 | 用**服务器时间 T**（响应 `Date` 头 `Date.parse`），query 与签名用同一 T |
| 4 | sdenv 自带 jsdom 不执行内联 script | `sign_env.js` 取不到 `appSecret`/`getSign`（全 undefined，无报错） | 改裸 `jsdom` 的 `JSDOM(html,{runScripts:'dangerously'})` 提取 |
| 5 | jsdom `navigator.platform='Win32'` 抛错 | navigator 属性只读 | 全改 `Object.defineProperty(...,{value,configurable:true})` 并包 try/catch |
| 6 | `jsonMD5ToStr(OBJ)` 抛错 | 该函数是**字符串版**，对对象调 `.replace` 报错 | 签名走 `jsonMD5ToStr(getSign(params))`；getSign 接受对象返回字符串 |
| 7 | 业务 JS 下载 404 / 拿到 RS6 包裹 | 直连 `https://.../js/ajax.js` 返回 404 | 先生成 RS6 Cookie 再带 Cookie 下载；且用 `new URL(rel, homeUrl)` 解析（homeUrl 在 `/datasearch/` 下） |
| 8 | `itemId:''` 触发"请求参数不能为空" | 空分类 id + 空关键词 | `itemId` 填真实分类 id（取自 `/config/NMPA_DATA.json`），`searchValue` 非空 |
| 9 | WSL `/root/nmpa-test` 会话重启后丢失 | 目录空了但 WSL 本体完好 | 重建目录 + 重装 sdenv(tgz)+jsdom（`registry.npmmirror.com`） |
| 10 | 环境变量名笔误 | 写成 `OPENSSL_LEGACY_RENEGOTIATION`（应为 `RENEGOTIATION`） | 正确：`OPENSSL_LEGACY_RENEGOTIATION=1` + `NODE_TLS_REJECT_UNAUTHORIZED=0` |
| 11 | RS6 Cookie 在两次请求间过期 | 第二次请求突然 412 | 每次请求前重新 `genCookie()` 生成新鲜 Cookie，并复用同一 T |
| 12 | JSVMP Hook 工具对 RS6 无效 | 标准 `Function.prototype.apply/call` Hook 日志为空 | RS6 用内部函数表+直接调用；不要硬 Hook，用 sdenv 真实执行 |

---

## 变体说明

| 变体 | 差异点 | 影响 |
|------|--------|------|
| RS 4/5 代 vs 6 代 | RS6 JS ≈ 230KB，比 4/5 代更大、检测项更多 | 4/5 代可手动补环境；6 代建议直接 sdenv |
| Cookie-only vs Cookie+URL 后缀 | 大部分 RS 站只需 Cookie；少数还需 URL 后缀签名 | sdenv 只生成 Cookie，需后缀的站建议 JsRpc 方案 |
| HTTP vs HTTPS | 部分 RS 站用 HTTP | sdenv 两种都支持；HTTPS 政府站点常需 `NODE_TLS_REJECT_UNAUTHORIZED=0`（证书链不完整）+ `OPENSSL_LEGACY_RENEGOTIATION=1` |
| 静态页 vs JSON API | 列表/详情可能是 SSR HTML（cheerio 解析）；也可能有 JSON API | API 请求需额外 sign（本 case：`sign=md5(排序参数+appSecret)`，GET query string） |
| `$_ts` 配置差异 | 不同站 `$_ts` 结构可能不同（有的仅 `nsd`+`cd`，有的还有 `cp`/`aebi`） | sdenv 不关心，让 RS JS 自己解析 |
| 高级检索 vs 标准检索 | 高级检索用 `itemIds`（复数逗号拼接），标准 `queryList` 用单数 `itemId` | 本 case 走标准路径：单数 `itemId` + 真实分类 id |

---

## 可验证事实清单（经验资产，同站升级时逐条核对）

1. 首次请求返回 412 + Set-Cookie 挑战
2. `NfBCSins2OywS` / `NfBCSins2OywO` / `NfBCSins2OywP` 为 RS6 特征参数
3. `$_ts` + `nsd`/`cd` 字段为 RS6 标志
4. sdenv 必须在 412 响应后执行生成 Cookie（裸 jsdom 会因 `typeof document.all` 卡死）
5. 第二次请求带生成的 Cookie 返回 200
6. sdenv = 魔改 jsdom + C++ V8 扩展（`documentAll.node`，`MarkAsUndetectable()` 实现 `document.all`）
7. 业务 sign 与 RS6 Cookie 是**两层独立校验**：RS6 过 412，sign 过 200 后的业务层
8. 业务请求是 **GET**（非 POST），参数进 query string
9. `appSecret = "nmpasecret2020"`（写死在 ajax.js，裸 jsdom 提取）
10. `sign = jsonMD5ToStr(getSign(params))`；getSign 过滤空值并按 key 排序；jsonMD5ToStr 拼 appSecret→encodeURIComponent→4 个防御性 replace→md5
11. `timestamp` 必须用**服务器时间**（响应 `Date` 头），`Date.now()`/`NaN` 会被拒
12. `itemId` 必须填真实分类 id（来自 `/config/NMPA_DATA.json`），`searchValue` 非空
13. 纯 Node.js（WSL2 + sdenv）即可完整跑通，无需浏览器自动化

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/env/env-native-protection.md` | document.all native HTMLDDA 能力（C++ MarkAsUndetectable） |
| `references/env/native-capability-gap.md` | native 能力缺口（typeof document.all === "undefined"） |
| `references/env/runtime-frameworks.md` | 框架选择（sdenv vs 裸 jsdom vs 手动补环境） |
| `references/network/session-chain.md` | Session 请求链（412 挑战 → Cookie 生成 → 200 重试） |
| `references/workflow/decision-tree.md` | RS6 路径决策（签名型反爬 → 补环境） |
| `references/crypto/crypto-entry.md` | 四层链路 source→entry→builder→writer（本 case 业务 sign 的 entry=getSign） |

---

## 关键方法论（可复用到其他 sign 逆向）

**不要猜签名字段集合——加载真实代码抓真值。**
> 能力允许时，**ruyipage + RuyiTrace 取证链是首选**（真实浏览器自然过 RS6 + NDJSON 直接给出 `hasTokenGet → getSign → jsonMD5ToStr` 调用链与入参/出参）。**本 jsdom 插桩路径仅当浏览器工具链不可用时的等价替代**。

当反复"签名验证失败"且本地公式已验证正确时，几乎总是"客户端签名的字段集合/顺序/来源"与服务端不一致，而非公式错。此时最有效的一步：

1. 用**裸 jsdom**（`runScripts:'dangerously'`）加载目标站**真实的**业务 JS（如 `ajax.js`）。
2. **WRAP** 真实签名函数（`getSign` / `jsonMD5ToStr` / `sign` 等），记录每次调用的入参与返回值。
3. **Stub 传输层**（假 `axios` / 假 `XMLHttpRequest`），重放真实请求拦截器，记录真实发出的 `method` / `query string` / `headers`。
4. 必要时**覆盖时间/随机源函数**（如本 case 覆盖 `pajax.getdate`），排除外部依赖干扰。

一次运行即可拿到"真实签名串 + 真实请求形态"，直接对照自己复刻的版本，差异一目了然。本 case 正是靠这一步发现"GET 而非 POST"和"服务器时间而非 Date.now()"两个根因。
