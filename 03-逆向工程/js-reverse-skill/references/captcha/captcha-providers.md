# 验证码厂商矩阵（封装层视角）

> **交叉引用**：完整厂商识别信号表（40+ 厂商）见 `provider-products.md`；本文件只收录与**封装层逆向直接相关**的内容：load/verify 参数构成、加密关注点、通过凭据形态。具体加密算法以每 case 的 RuyiTrace 证据为准，本表不替代取证。

## 国内主流厂商

### 极验 GeeTest（v3 / v4）—— 首选练手目标

| 项 | v3 | v4 |
|---|---|---|
| 识别信号 | `gt`、`challenge`、`api.geetest.com` | `captcha_id`、`lot_number`、`gcaptcha4.geetest.com` |
| load 接口 | `register` → `gettype.php` → `get.php` | `load` |
| verify 接口 | `ajax.php` | `verify` |
| 核心加密参数 | `w`（AES 加密明文 + RSA 加密 AES key） | `w`（明文含 PoW 字段，加密同族） |
| w 明文关键字段 | passtime / userresponse / 轨迹 / rp（校验值，形态以 trace 为准） | lot_number / pow_msg / pow_sign / 轨迹 / 答案 |
| 通过凭据 | `validate`（业务侧再组装 seccode 二次校验，组装方式版本相关） | seccode 四件套：lot_number / captcha_output / pass_token / gen_time |
| 答案类型 | 滑块为主（slide_match 可解） | 滑块 / 点选 / icon / gobang / 无感 |

### 数美 Shumei（标签：`shumei-captcha`）

- 识别信号：`castatic.fengkongcloud.cn/pr/.../smcp.min.js`、`initSMCaptcha`、`organization`、`SMCaptcha.getResult()`
- 关键参数：`organization`（商户标识）、`rid`（请求标识，verify 后回传）、`captchaUuid`、`pass`
- 加密关注点：配置接口下发 `conf`（含加密开关/算法版本）；答案与轨迹合并加密；`rid` 是链路绑定核心，全流程必须同 rid
- 通过凭据：`pass: true` + `rid`，业务侧用 rid 二次验签
- 题型：滑块 / 点选 / 语序 / 无感

### 顶象 DingXiang（标签：`dingxiang-captcha`）

- 识别信号：`cdn.dingxiang-inc.com`、`captcha-ui/v5/index.js`、`_dx.Captcha`、`constId`
- 关键参数：`appId`（业务）、`constId`（设备/会话指纹相关）、`apiServer`、dx token
- 加密关注点：`constId` 由独立指纹 JS 生成（先逆指纹再逆验证码，两条链）；token 分段拼接（指纹段 + 行为段 + 答案段）
- 题型极多：滑块 / 文字点选 / 图标点选 / 语序 / 刮刮卡 / 旋转 / 乱序拼图 / 面积 / 差异点击——**先按 `classify_verify.py` 定题型再定解法**

### 腾讯防水墙 TCaptcha（标签：`tencent-tcaptcha`）

- 识别信号：`captcha.gtimg.com`、`TCaptcha`、`aid`
- 关键参数：`aid`（业务 appid）、`ticket`、`randstr`
- 加密关注点：JS 重度混淆（JSVMP 路线，走补环境）；`ticket`+`randstr` 与页面 Session 强绑定
- 通过凭据：`ticket` + `randstr` 两件套，业务接口都要带
- 题型：滑块 / 点选 / 语音 / 无感

### 网易易盾 Yidun（标签：`netease-yidun`）

- 识别信号：`captcha.yidun`、`dun.163.com`、`c.dun.163.com`、`NECaptcha`
- 关键参数：`captchaId`、`validate`、`fp`（指纹）、`token`、`data{d,m,p,ext}`
- 加密关注点：`fp` 独立指纹链（先逆 fp）；`validate` 由答案+轨迹加密生成；行为采集与题面混合
- 接口链：`GET c.dun.163.com/api/v3/get`（JSONP，拿 token/type）→ `GET /api/v3/check`（JSONP，提交 data）→ 业务二次校验 `POST ir-sdk.dun.163.com/v4/j/up`（neguardian tk）+ `POST dun.163.com/node/api/check-guardian.json`（token=<tk> → 204）
- 题型：滑块 / 点选 / 无感（type=5 INTELLISENSE 智能无感，`data.d` 恒空串、`m` 非空；滑块 type=2 `data.d` 非空、**`data.m` 恒空串**——d 与 m 的空串陷阱正好相反，易盾滑块案例实证）
- 实测要点（见 `cases/yidun-intellisense-vm-env.md`）：SDK `core-optimi.*.min.js` obfuscator.io 混淆，vm 沙箱补环境直跑；自定义 AES（`__SEED_KEY__` 共享）+ 带 token 的 XOR 编码双层加密 `aes(xorEncode(token, 明文))`；行为采集 SDK neguardian 与 core-optimi **共用同一套 AES key**，`/v4/j/up` 的 `d` 服务端只校验可解密性，`p` 为页面固定 appId（错 → 5509）
- 滑块实证（type=2，2026-08-09，纯协议 3/3 result:true，见 `cases/yidun-jigsaw.md`）：check 的 `data.m` 必须为空串 `""`（写数字 0 全量失败，真实 SDK 6 份样本一致）；`p` 明文=parseInt(slider.left)/320*100，slider.left=jigsaw.left-10.5，d 轨迹末点=jigsaw.left；坐标误差 6-13px 需 ±扫描；irToken 一次性
- 缺口坐标获取：**ddddocr / OpenCV 自动识别不可用**（拼图块被重着色，偏差 12-145px），走 `scripts/click_gap.py` 人工点击或打码平台（`solver_request_template.py`）

### 阿里云 NoCaptcha / AWSC（标签：`aliyun-captcha`）

- 识别信号：`AWSC`、`nc_`、`afs`、`aliyuncs.com`
- 关键参数：`appkey`、`scene`、`sessionId`、`sig`、`token`、`nc_token`
- 加密关注点：`sig` 由答案+会话材料签名；`sessionId` 贯穿链路；`afs` 是行为采集段
- 题型：滑块 / 一点即过 / 无痕 / 图像复原

## 海外 / 组件类（简要）

| 厂商 | 识别信号 | 封装层要点 |
|---|---|---|
| reCAPTCHA v2/v3 | `g-recaptcha`、`/api2/anchor` | `g-recaptcha-response` token；v3 是评分制，token 有效≠通过 |
| hCaptcha | `hcaptcha.com/1/api.js`、`rqdata` | `h-captcha-response`；rqdata/rqtoken 绑定 |
| Cloudflare Turnstile | `challenges.cloudflare.com/turnstile` | `cf-turnstile-response`；与 CF WAF challenge 区分 |
| Arkose FunCaptcha | `arkoselabs`、`fc-token`、`blob` | blob 是加密行为数据，游戏类题型不适合本地求解 |
| 自托管（AJ-Captcha / Tianai） | `blockPuzzle`、`clickWord`、`TAC` | 开源库，算法可直接读源码还原，是除极验外的低难度练手目标 |

## 选型建议（按逆向难度）

```text
入门：自托管开源库（AJ-Captcha/Tianai，源码可读）→ 极验 v3 滑块（链路标准、资料多）
进阶：极验 v4（PoW 字段）、数美（conf 动态配置）、阿里云
困难：顶象（题型多+指纹双链）、易盾、腾讯（JSVMP 补环境）
```

**纪律**：厂商判断与题型判断分开（命中顶象≠滑块）；同一厂商不同业务的配置可能不同（mode/appId 维度），case 之间不能直接复用 w 明文结构，只能复用方法论。
