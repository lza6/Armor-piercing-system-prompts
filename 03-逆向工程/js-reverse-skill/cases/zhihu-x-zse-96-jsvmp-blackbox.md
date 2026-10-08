# Case：知乎 x-zse-96 签名 + __zse_ck 挑战 Cookie（JSVMP 黑盒执行 + MD5 纯算 + Cookie 前缀伪造）

> 难度：★★★（JSVMP 黑盒执行 + 签名 source 编排；无需反编译字节码，环境补全规模中等）
> 还原方案：A 纯算还原（MD5 + source 拼接 + Cookie 静态拼接）+ B vm 沙箱执行（模块 1514 JSVMP 生成 `__g._encrypt`）
> 实现语言：Node.js
> 最后验证日期：2026-08-15（实测 5/5 次 GET 返回 200 + 完整评论 JSON；对照实验：错误签名/前缀 → 403）
> 平台类型：问答社区（zhihu.com，评论区 root_comment 接口）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- [x] 请求头 `x-zse-96` = `"2.0_" + encrypt(md5(source))`，`x-zse-93` = `"101_3_3.0"` 固定
- [x] source = `[x-zse-93, urlPath+search, d_c0, body(≤4096B), xZst81].filter(Boolean).join("+")`
- [x] `__zse_ck` Cookie：`<前缀>_<base64> - <meta content>` 三段式（`-` 分隔），前缀版本号由服务端校验
- [x] 签名生成端：`static.zhihu.com/heifetz/2659.app.*.js`（webpack 模块 88545 编排 + 模块 1514 JSVMP 字节码解释器）
- [x] `x-zse-96` 写入点：模块 88545 `eZ` 内 `Headers.set("x-zse-96", ...)`（RuyiTrace `stack` 字段可定位）
- [x] 请求特征：评论接口需滚动页面到评论区才触发 `root_comment`；首次访问 403 challenge 页动态下发 zse-ck 脚本（URL hash 每次不同）
- [x] 反调试特征：JSVMP 带随机填充（同一输入每次输出不同），服务端私钥解密验证，黑盒执行即可

## 加密方案

- 路径：A 纯算还原（MD5/source/__zse_ck）+ B vm 沙箱执行（JSVMP 模块 1514）
- 框架：不使用（手写最小浏览器环境 + webpack 模块加载器）
- TLS 客户端：不发真实请求（纯 Node.js `https` 直连即可，无需 TLS 指纹模拟）
- 核心思路：webpack 模块 88545 构造 source → 模块 10261 标准 MD5 → 模块 1514 JSVMP 生成的 `__g._encrypt(encodeURIComponent(md5))` → `"2.0_" + 结果` 写 Header；`__zse_ck` 前缀版本号 `009_` + 随机 base64 + 动态 meta content 即可通过校验，无需执行 zse-ck Go WASM。

### 签名公式
```
x-zse-93 = "101_3_3.0"                                # ep(3, "3.0", "101") 固定
source   = [zse93, pathname+search, d_c0, body, xZst81].filter(Boolean).join("+")
x-zse-96 = "2.0_" + __g._encrypt(encodeURIComponent(md5(source)))
```

### __zse_ck 简化方案（2026-08-15 实测）
```
首次访问 → 403 + Set-Cookie: _xsrf + <meta id="zh-zse-ck" content="<随机>">
        + <script src="https://static.zhihu.com/zse-ck/v4/<hash>.js">（Go WASM）
服务端校验 __zse_ck 前缀版本号：
  "009_" 前缀 + 任意 base64 → API 200 ✅
  "001_"（脚本默认 fallback）/ "005_" / "003_" / "101_" → 403
即：__zse_ck = "009_" + 随机base64 + "-" + meta content 即可通过，无需执行 Go WASM
```

### d_c0 Cookie
```
d_c0 = <base64 udid> + "|" + <unix 秒时间戳>
```
自洽即可，但**参与签名 source 的 d_c0 必须与请求 Cookie 一致**。

---

## 踩坑记录（IMPLEMENT 必查）

| # | 坑 | 现象 | 正确做法 |
|---|---|---|---|
| 1 | 缺最小浏览器环境 | JSVMP 解释器抛异常或输出错误签名 | 补 `window/document/navigator/location/screen/history/canvas 2d/atob/Blob/Headers/URLSearchParams`；`atob` 用 `Buffer.from(s,'base64').toString('binary')` 实现 |
| 2 | 评论接口不触发 | 抓包无 `root_comment` 请求 | 需滚动页面到评论区才触发，取证时在窗口内滚动 |
| 3 | `__zse_ck` base64 换字符 | 403 | 前缀 base64 **不能**把 `+` `/` `=` 换成 `-` `_`（破坏 `-` 分隔符解析） |
| 4 | 前缀版本号用默认 fallback | 403 | `001_`（脚本默认值）/`005_`/`003_`/`101_` 均 403，只有 `009_` 通过；依赖服务端当前校验策略 |
| 5 | 快速连续请求 | `403 {"error":{"code":40362}}` IP 风控，风控持续数分钟，所有请求（含页面）全 403 | 控制请求频率 ≥5s，验证逐次进行 |
| 6 | 混淆 JSVMP 字节码 | 试图反编译模块 1514 字节码 | 黑盒执行即可：JSVMP 带随机填充，服务端私钥解密验证，无需精确复现环境指纹 |
| 7 | 忘记 xZst81 参与签名 | 签名校验失败 | source 拼接必须包含 `xZst81`（可为空串经 filter(Boolean) 剔除），字段顺序不可变 |

---

## 可验证事实清单（经验资产，同站升级时逐条核对）

1. `x-zse-93` 恒为 `101_3_3.0`（`ep(3, "3.0", "101")`）
2. `x-zse-96` 前缀恒为 `2.0_`，后接 `encrypt(md5(source))`，encrypt 输入经 `encodeURIComponent`
3. source 拼接顺序固定：`[x-zse-93, urlPath+search, d_c0, body(≤4096B), xZst81].join("+")`，空值剔除
4. 签名生成模块：88545（编排）/ 1514（JSVMP 生成 `__g._encrypt`）/ 10261（标准 MD5）/ 62845（`"2.0"` 常量）
5. 模块 1514 以 `(new l).O("<base64 字节码>")` 方式加载，挂载 `__g._encrypt`
6. `x-zse-96` 写入点为 `88545/eZ` 内 `Headers.set`（RuyiTrace stack 可定位）
7. `__zse_ck` 前缀 `009_` + 任意 base64 + `-` + meta content 通过校验（2026-08-15 实测）
8. `001_`/`005_`/`003_`/`101_` 前缀 → 403
9. `__zse_ck` 的 base64 段必须保留 `+` `/` `=` 原字符
10. 目标接口 `GET /api/v4/comment_v5/answers/{answerId}/root_comment` 匿名可访问，无需登录
11. `d_c0 = <base64 udid>|<unix秒>`，签名 source 中的 d_c0 必须与 Cookie 一致
12. 短时高频请求触发 IP 风控 40362，间隔 ≥5s 可规避
13. zse-ck 脚本（Go WASM，`syscall/js` 导入）URL hash 每次访问动态下发，前缀校验收紧时需黑盒执行它

---

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/decision-tree.md` | 题型判定 + 路径决策（JSVMP 黑盒 vs 纯算） |
| `references/workflow/experience-rules.md` | 规则 8/15（JSVMP 先选路径、环境伪装优先）、规则 18（补丁先于 JSVMP 加载） |
| `references/network/session-chain.md` | 403 challenge 页 → Cookie 链重放（__zse_ck 简化方案） |
| `references/network/ip-risk-control.md` | IP 风控识别（40362 与 403 页面挑战区分） |
| `references/deobfuscation/obfuscation-identify.md` | JSVMP/字节码解释器识别 |
