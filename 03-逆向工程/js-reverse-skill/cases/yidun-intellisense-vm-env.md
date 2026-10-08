# Case：网易易盾 智能无感验证码（type=5）—— vm 沙箱补环境运行 SDK

> 难度：★★★★
> 还原方案：D 环境伪装（vm 沙箱补环境运行 core-optimi SDK，不反编译混淆）
> 实现语言：Node.js（纯协议，无浏览器）
> 最后验证日期：2026-08-08
> 平台类型：网易易盾智能无感（dun.163.com/trial/sense?force=true 试用页，type=5 INTELLISENSE）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- JS 特征：主 SDK `core-optimi.m25b40.v2.28.5.min.js`（631KB，obfuscator.io 多字符串数组混淆）；字符串解码 = 多数组共用同一自定义 base64 函数（标准 base64 字符集 + URL decode）；自定义 AES `__SBOX__`(512字符) + `__SEED_KEY__`=fd6a43ae25f74398b61c03c83be37449；模块访问器注入 `window.__RUIYU_MODS__(n)` 可枚举 webpack 模块
- 参数特征：`fp`（window.gdxidpyhxde + cookie gdxidpyhxdE，含 `\` 字符 + `:时间戳` 后缀）；`cb`（uuid32 插入 `vfnv46` 于 pos=[1,10,12,13,26,31] + AES，92 字符含 `.`）；`dt`（getconf 下发存 localStorage key `ujg3ps2znyw`）；`data`（check 提交 `{d:'', m, p, ext}`，智能无感 d 恒空串，**与滑块 type=2 相反：无感 d 空/m 非空，滑块 d 非空/m 空串**）
- 请求特征：`GET c.dun.163.com/api/v3/get`（JSONP，参数 fp/cb/dt/type=5/version=2.28.5）；`GET c.dun.163.com/api/v3/check`（JSONP，参数 token/type=5/width=320/data）；`POST ir-sdk.dun.163.com/v4/j/up`（JSON，返回 `{code:200,data:{ed,es,td,tk,ts}}`）；`POST dun.163.com/node/api/check-guardian.json`（form `token=<tk>` → 204）
- 反调试特征：obfuscator.io 字符串数组混淆为主，无额外反调试

## 加密方案

- 路径：D 环境伪装（vm 沙箱加载 SDK，模块访问器直调模块）
- 框架：Node.js `vm.createContext` + `vm.runInContext`
- TLS 客户端：Node.js 原生 https（多轮真实 API 验证通过；生产站按 `references/network/tls-validation.md` 评估）
- 核心思路：
  - fp：加载 SDK 自动生成，读 `window.gdxidpyhxde`
  - cb：模块 3 `uuid` + 模块 10 `aes`
  - data：模块 10 `xorEncode` + 模块 3 `sample` + 模块 10 `aes`
  - neguardian tk：与 core-optimi 共用同一套自定义 AES（`fd6a43ae25f74398b61c03c83be37449`），用其加密任意数据即可通过服务端解密校验

### data 构造（verifyIntelliCaptcha 还原）

```
token     = get 接口返回
xor(p)    = mod10.xorEncode(token, p)
mapped    = traceData.map(p => xor(p))            # 轨迹点逐个 XOR 编码
m = aes(sample(mapped, 50).join(':'))             # 均匀采样 ≤50 点，':' 连接，AES
p = aes(xor(clickPoint))                          # 点击点 [dx,dy,dt]
ext = aes(xor('1,' + traceData.length))
data = { d: '', m, p, ext }
```

- 轨迹点格式：`[Math.round(clientX-left), Math.round(clientY-top), now-beginTime]` 字符串
- `sample`（模块3）：均匀间隔采样，`i >= idx*(len-1)/(n-1)` 取点；长度 ≤50 直接返回
- 模块 5 常量：`CAPTCHA_TYPE.INTELLISENSE=5`、`SAMPLE_NUM=50`

### neguardian /v4/j/up

```
body = {
  p: 'YD00615509752509',   # 页面固定 appId（core.js createNEGuardian({appId}) 传入，非 SDK 生成）
  v: '1.0.1',              # neguardian 版本
  vk: 'd44593ca',          # 固定版本 key
  n: '<32位hex随机>',
  d: aes(任意行为数据)       # 同一套 AES，服务端只校验可解密性
}
resp = {code:200, data:{ed, es, td, tk, ts}}   # tk 用于 check-guardian
```

## 踩坑记录

1. **坑：data 不在 check 请求函数内构造** → 正确做法：check 的 data 参数由组件 `verifyIntelliCaptcha` 的 `onVerifyCaptcha` 经 store dispatch 传入；定位方法：搜 `'ext':` / `'p':` 对象字面量，逆查上游构造点。
2. **坑：把智能无感 data 的 d 当成需要加密的字段** → 正确做法：智能无感 d 恒为空字符串 `{d:'',...}`；只有滑块（type=2）等题型 d 才非空（注意：滑块的 m 恒空串，与无感正好相反）。
3. **坑：以为 neguardian 的 AES 是独立体系** → 正确做法：neguardian 与 core-optimi **共用** `fd6a43ae25f74398b61c03c83be37449` 一处 key，这是 /v4/j/up 复用模块 10 aes() 的关键。
4. **坑：/v4/j/up 的 p 参数填错** → 正确做法：p 是页面传的 appId `YD00615509752509`（来自 core.js `createNEGuardian({appId:...})`），p 错误 → 5509 "Invalid Product"。
5. **坑：以为 /v4/j/up 的 d 需要真实行为数据** → 正确做法：d 服务端只校验可解密性，任意明文 AES 后均返回 200 + tk（错误码从 5504 变 5509 提示解密已通过）。
6. **坑：过度补齐 check-guardian 的 token 校验** → 正确做法：check-guardian.json 任意 token 均 204，守卫校验极弱（或仅记日志）。
7. **坑：ir SDK（ir.2.0.13）环境不足导致卡住** → 正确做法：ir SDK 需多环境（toLocaleLowerCase 等），但 get/check 链路无需 irToken 即可通过，跳过不补。

## 可验证事实清单（经验资产）

1. 主 SDK `core-optimi.m25b40.v2.28.5.min.js` 约 631KB，obfuscator.io 多字符串数组混淆
2. 自定义 base64 字母表 = `MB.CfHUzEeJpsuGkgNwhqiSaI4Fd9L6jYKZAxn1/Vml0c5rbXRP+8tD3QTO2vWyo`（标准 base64 字符集 + URL decode）
3. 自定义 AES：`__SBOX__` 512 字符 + `__SEED_KEY__=fd6a43ae25f74398b61c03c83be37449`；模块 10 `aes()` 输出含 `.` 分隔段
4. 模块 10 另导出 `xorEncode(token, data)` / `xorDecode`（带 token 的 XOR 编码）
5. `cb`：uuid(32) 插入 `vfnv46` 于 pos=[1,10,12,13,26,31] + AES → 92 字符含点
6. `fp`：`window.gdxidpyhxde` + cookie `gdxidpyhxdE`，含 `\` 字符 + `:时间戳` 后缀
7. `dt`：getconf 接口下发，localStorage key `ujg3ps2znyw`
8. `data` = `{d:'', m, p, ext}`，m/p/ext 均为 `aes(xorEncode(token, 明文))` 双层加密
9. `sample` 为均匀采样：`i >= idx*(len-1)/(n-1)` 取点，≤50 点，`:` 连接
10. `CAPTCHA_TYPE.INTELLISENSE=5`、`SAMPLE_NUM=50`（模块 5 常量）
11. neguardian 请求体 `{p: appId, v:'1.0.1', vk:'d44593ca', n:32hex, d:aes(任意)}`；p 错误 → 5509 "Invalid Product"
12. check 返回 `{"data":{"result":true,"token":"...","validate":"..."},"error":0}`
13. get 返回 `{bg:[null,null], token, type:5}`（智能无感不返回背景图）
14. check-guardian.json 任意 token 均 HTTP 204
15. 同一易盾 captchaId 换页：fp/cb/data 生成逻辑完全通用，只换 CAPTCHA_ID 和 referer

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/captcha/captcha-providers.md` | 易盾厂商条目（识别信号/参数/关注点） |
| `references/captcha/captcha-request-chain.md` | load→solve→verify 三段链模型 |
| `references/captcha/captcha-motion-encryption.md` | 轨迹加密 + 风控校验点（本案轨迹为明文数组+xorEncode） |
| `references/captcha/captcha-solving-handoff.md` | 答案层接入硬约束（无感题型不需图像求解） |
| `references/env/env-native-protection.md` | vm 沙箱补环境 + 字符串数组还原 |
| `references/workflow/experience-rules.md` | 规则 5（时效性校验）+ 规则 20（全字段解密） |
