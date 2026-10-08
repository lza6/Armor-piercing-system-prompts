# Case：京东 h5st（js_security_v3 JSVMP）—— vm 沙箱执行原版 + TLS 指纹 Firefox 系 + 会话级 fp/eid 绑定

> 难度：★★★★★
> 还原方案：B vm 沙箱执行 + D 环境伪装
> 实现语言：Node.js
> 最后验证日期：2026-08-10
> 平台类型：京东（www.jd.com / api.m.jd.com / cactus.jd.com / jra.jd.com）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- JS 特征：`js_security_v3_*.js`（取证版 0.1.4）核心算法在 `_$sdnmd` **while-switch 字节码虚拟机**中，静态还原不可行；入口对象 `window.ParamsSign`（`new ParamsSign({appId:'b5216'}).sign({appid,clientVersion,client,functionId,t,body})` → Promise `{h5st}`）；beta 模式 `_$fI`=console.log；`_$ft.encode`/`_$fa.parse` 为 CryptoJS 魔改加解密
- 参数特征：`h5st` 分号分隔 **10 字段**（v5.3）：17 位时间串；fp；appId(b5216)；token(tk03/tk06)；hash1(64hex sha256)；版本号；13 位毫秒时间戳；**base64url 加密环境数据**（明文为 JSON，含 pp/plugins 等）；hash2(64hex)；固定 base64 token
- 请求特征：动态密钥接口 `POST cactus.jd.com/request_algo`（返回 `data.result.tk`=远程 token tk03 + `data.result.algo`=服务端下发算法）；业务接口 `GET api.m.jd.com`（functionId=pc_home_feed 等）**校验 TLS 指纹（JA3/JA4）**；eid token 接口 `POST jra.jd.com/jsTk.do`（pc-tk.js 采集环境指纹后请求）；响应头 `X-Rp-Sdtoken: set;1800;<token>` 续期 sdtoken
- 反调试特征：深度环境检测——函数 toString native 检测、localStorage 指纹缓存（`WQ_gather_cv1` canvas / `WQ_gather_wgl1` webgl / `WQ_dy1_vk` fp 缓存 / `JDst_behavior_flag`）、错误栈伪装（bu2 字段）、navigator.plugins 采集（pp 字段）

## 加密方案

- 路径：B vm 沙箱执行（+ D 环境伪装）
- 框架：Node.js `vm.createContext` + `vm.runInContext`（**不覆盖标准内置对象**）
- TLS 客户端：curl-cffi-node（impersonate `firefox133`）——**Node 原生/Chrome 指纹均 403，仅 Firefox 系指纹 200**
- 核心思路：vm 沙箱执行原版 js_security_v3 → mock XHR 转发 request_algo 真实请求（预热异步获取远程 token）→ `ParamsSign.sign()` 生成 h5st → curl-cffi-node(Firefox 指纹) 携带会话 cookie 请求 api.m.jd.com

### 预热时序（关键）

```
首次 sign → 本地 token（tk06）兜底
request_algo 异步完成 → 后续 sign 用远程 token（tk03）
浏览器 pc_home_feed 实际用 tk06 也成功 → 预热非硬性，但 tk03 更稳
```

### h5st 生成链路

```
signer 构造:
  createSignSandbox(env) → vm 内加载 js_security_v3 → 注册 XHR mock（onXhrResponse 监听 request_algo）
预热:
  sandbox 内触发 sign() → js_security_v3 内部 XHR 调 request_algo → mock 转发真实网络 → onRequestAlgoDone resolve
生成:
  sign({appid,clientVersion,client,functionId,t,body}) → Promise {h5st}
  body 参数 = SHA256(JSON.stringify(bodyObj))，与请求 body 明文完全一致
  t 参数必须等于 h5st 第 7 字段（sign 时的 ts），同进程生成即请求
```

## 踩坑记录

1. **坑：vm 覆盖标准内置对象（win.Function 等）** → 正确做法：`vm.createContext` 提供原生全局，覆盖 `win.Function` 导致 `Function.prototype.bind` 丢失，js_security_v3 加载直接报错。只补缺失的浏览器环境桩，不动原生对象。
2. **坑：TLS 指纹 403（空 body）** → 正确做法：`api.m.jd.com` 校验 JA3/JA4，Node 原生 https / Chrome 指纹一律 403；仅 Firefox 系指纹（curl-cffi-node `firefox133`）返回 200。**先确认指纹层再过业务风控**。
3. **坑：605「the request needs to authenticate」当成 IP 问题** → 正确做法：605 = 业务风控，与 IP 无关（同 IP 下浏览器 h5st 200、vm h5st 605）。根因是 fp/eid 不匹配或环境数据不完整，不是换 IP 能解决的。
4. **坑：fp 跨会话混用 → 605** → 正确做法：fp 存 localStorage `WQ_dy1_vk`（缓存优先），且 **fp 与会话 eid token 绑定**（同会话生成）。跨会话混用必 605。
5. **坑（本 case 最隐蔽）：第 8 字段环境数据 `pp`（navigator.plugins 详情）为空 → 605** → 正确做法：浏览器 5 个 Firefox 插件（Widevine/OpenH264 等），vm 采集失败 → 明文短 ~115 字符 → 密文短 ~135-150 字节 → 服务端校验失败。patch js_security_v3 的 pp 采集函数，用真实 Firefox 插件名填充 p1/p2/p3（Widevine Content Decryption Module / OpenH264 Video Decoder / application/x-mpegURL）→ 第 8 字段 429→549 字节 → 校验通过。
6. **坑：eid token 过期（会话级 ~20 分钟）导致重放 605** → 正确做法：纯协议自动获取——vm 执行 `pc-tk.js` → `POST jra.jd.com/jsTk.do` → 服务端下发 token（jdd03 前缀，= x-api-eid-token）+ eid（=3AB9D23F7A4B3C9B），每次运行自动刷新，不依赖浏览器抓包。
7. **坑：localStorage 指纹缓存缺失导致环境数据短** → 正确做法：从浏览器 profile 的 `data.sqlite` 提取注入（`WQ_gather_cv1`/`WQ_gather_wgl1`/`WQ_dy1_pFlag`/`JDst_behavior_flag`；Firefox 压缩值需 LZ4 解压，明文值可直接注入）。
8. **坑：debug 时看不到加密前明文** → 正确做法：js_security_v3 的 beta 模式（`_$fI`=console.log）+ patch `_$ft.encode`/`_$fa.parse`（CryptoJS 魔改）捕获第 8 字段**加密前明文 JSON**，精确定位缺失字段；pc-tk.js 的 `_0x1d73` 可解码混淆字符串。
9. **坑：重放抓包 h5st 验证失败** → 正确做法：重放验证注意时效——t 与 h5st 第 7 字段一致 + eid 会话有效期内，且用同会话 cookie。

## 可验证事实清单（经验资产）

1. h5st 结构 = 10 字段，`;` 分隔，v5.3：`20260809182941712;nn22zzez51beibz3;b5216;tk03...;64hex;5.3;1786271376712;base64url;64hex;base64`
2. 第 1 字段 = 17 位：`yyyyMMddHHmmssSSS` + 2 位随机
3. 第 3 字段 appId = `b5216`（pc_home_feed 场景）
4. 第 4 字段 token：tk06 本地 / tk03 远程（request_algo 下发），**tk03 必须为远程 token 才稳定通过**
5. 第 5 / 第 9 字段 = 64 hex = SHA256（hash1 / hash2）
6. 第 7 字段 = 13 位毫秒时间戳，与请求 URL `t` 参数同源（样本差 8ms）
7. 第 8 字段 = base64url 加密的环境数据明文 JSON；**浏览器 582 字节 / vm 补全后 549 字节 / 未补 pp 仅 396 字节**（差 186 字节）
8. `body` query 参数 = `SHA256(JSON.stringify(bodyObj))`，与请求 body 明文完全一致
9. `x-api-eid-token` = eid token（jdd03 前缀），会话级 ~20 分钟；cookie `3AB9D23F7A4B3CSS` 对应
10. eid 标识 = cookie `3AB9D23F7A4B3C9B`
11. `sdtoken` cookie 由响应头 `X-Rp-Sdtoken: set;1800;<token>` 续期（1800s）
12. 403（空 body）= TLS 指纹层拦截（Node/Chrome 指纹）；605 = 业务风控（fp/eid 绑定或环境数据不完整）
13. `api.m.jd.com` 仅放行 Firefox 系 TLS 指纹（JA3/JA4）；curl-cffi-node impersonate `firefox133` 可用
14. 纯协议自动获取 eid：vm 执行 pc-tk.js → `POST jra.jd.com/jsTk.do` → 下发 `token` + `eid`
15. fp 缓存键 `WQ_dy1_vk`（结构 `{"5.3":{"<appId>":{"v":"<fp>"}}}`），与 eid token 同会话绑定，跨会话混用 → 605

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/decision-tree.md` | JSVMP 题型判定 → 选 B vm 沙箱而非纯算还原 |
| `references/network/tls-validation.md` | TLS 指纹层验证（JA3/JA4，403 vs 605 分层） |
| `references/network/session-chain.md` | eid/token 会话链（fp 与会话绑定） |
| `references/env/env-native-protection.md` | vm 沙箱补环境（不覆盖标准内置对象） |
| `references/fingerprint/fingerprint-value-replay.md` | localStorage 指纹缓存提取与回放（data.sqlite/LZ4） |
| `references/quality/validation.md` | 交叉验证 ≥5 次、以业务 code 判定成功 |
| `references/workflow/experience-rules.md` | 相关经验法则编号（时效性校验 / 全字段解密） |
