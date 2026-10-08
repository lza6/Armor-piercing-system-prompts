# Case：SM2/SM4/SM3 国密三参数签名纯算还原（jobonline.cn 就业在线）

> 难度：★★
> 还原方案：A 纯算还原（SM2 + SM4-CBC + SM3）
> 实现语言：Node.js（sm-crypto 库）
> 最后验证日期：2026-07-11
> 平台类型：就业在线（jobonline.cn）

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

- JS 特征：app.js 内联 SM2 公钥常量（变量 `D`，130 位 hex，以 `04` 开头）；引用国密算法（sm-crypto 同款）；`EncryptFlag=2` 表示 SM 系列加密
- 参数特征：`businessData`（请求体，SM4-CBC hex，192 字符）、`E-CONTENT-PATH`（Header，SM2 密文以 `04` 开头，456 字符）、`E-SIGN`（Header，SM3 哈希 64 字符 hex）
- 请求特征：目标接口 POST `api.jobonline.cn/jobtbao-platform-srv/v1/platform/unsafe/position/getPositionDetail?target=POSITION_DETAIL`；Header 带 `platform:3`、`E-VERSION:v2.0.0`、`Content-Type: application/json;charset=UTF-8`
- 反调试特征：无（标准国密加密，无环境检测）

## 加密方案

### 三参数生成逻辑

每次请求生成 4 个随机 32 位 hex 值：`w`（SM4 密钥）、`S`（SM4 IV）、`x`（响应解密密钥）、`C`（响应解密 IV）。

| 参数 | 算法 | 公式 |
|---|---|---|
| `businessData` | SM4-CBC | `sm4.encrypt(JSON.stringify(data), w, {iv: S})` |
| `E-CONTENT-PATH` | SM2 | `"04" + sm2.doEncrypt(w+","+S+","+x+","+C, publicKey, 1)` |
| `E-SIGN` | SM3 | `sm3("business" + businessData + "data")` |

### 关键常量

- **SM2 公钥**（硬编码在 app.js 变量 D）: `043f4a9673db98fd52a87e087da75ca8d4978748188e29373acc131887d7b78ee89b07364f644352e4cb4029d8330509368b27b10638345c8afd41149626d917aa`
- **EncryptFlag**: `2`（表示 SM 系列加密）
- **E-VERSION**: `v2.0.0`
- **其他 Header**: `platform: 3`, `Content-Type: application/json;charset=UTF-8`

### SM2 doEncrypt 返回格式

sm-crypto 库的 `sm2.doEncrypt` 返回 C1+C3+C2（不含 "04" 前缀），原始代码手动补 `"04"` 前缀拼到 E-CONTENT-PATH。

### 响应解密

响应体 `object` 字段中如含 `Down_Encrypt_Flag`：
- `flag === "1"`: AES-128-CBC 解密（固定密钥 `1jdj12480daced33` / IV `d22b0a851e014f7b`）
- 其他: SM4-CBC 解密（用请求时生成的 x/C）

本次案例响应未加密，直接返回明文 JSON。

## 踩坑记录

1. **参数名大小写**：请求头参数名是 `E-CONTENT-PATH` / `E-SIGN`（带连字符），不是 `e-content-path` / `eSign`。Grep 搜索时需不区分大小写。
2. **SM2 返回格式**：sm-crypto 的 `doEncrypt` 不含 "04" 前缀，需手动拼接 `"04" +`。不同 SM2 库返回格式可能不同，需验证。
3. **随机密钥生成**：原始代码用 `window.crypto.getRandomValues`，Node.js 等价用 `crypto.randomBytes`。
4. **请求体结构**：`{positionId, activityCode, businessEnum}`，其中 `positionId` 需转字符串。
5. **公钥选择**：app.js 中有两个公钥——默认公钥 D 和 jobtbao 接口专用公钥。目标接口走默认公钥 D。

## 可验证事实清单（经验资产）

1. `businessData` 是 SM4-CBC 加密的 hex 字符串，长度 192 字符（对应 96 字节密文，即 6 个 16 字节块）
2. `E-CONTENT-PATH` 以 "04" 开头，长度 456 字符（SM2 密文 C1=128 + C3=64 + C2=密钥信息长度×2）
3. `E-SIGN` 是 SM3 哈希，固定 64 字符 hex
4. 5 次请求全部返回 HTTP 200 + 正确职位数据
5. 响应未加密（无 Down_Encrypt_Flag）
6. SM4 密钥/IV 每次随机生成，businessData 每次不同
7. E-SIGN 输入是 `"business" + businessData + "data"` 字符串拼接

## skill 流程经验

### 本次执行中的问题

1. **ENV_READY 全量检测耗时**：`check_external_tools.js` 一次性检测所有工具（10+ 次 spawnSync），纯算场景不需要。**已优化**：增加 `--quick` 模式。
2. **INTENT_CONFIRM 未等待用户确认**：AI 在范围尚未确认时直接进入 ENV_READY，没有问用户选择哪种分析路径。**已优化（当时）**：INTENT_CONFIRM 增加用户确认门禁。**（2.3.24 修正注记）**：该门禁在 2.3.23 已撤销——它把"要素缺失时该问"和"要素齐全时也要确认"混为一谈，导致要素齐全仍反复询问"是否继续"。现行政策（第 1 节确认策略）：要素齐备（目标 URL + 参数名可确定）默认自动推进，不要求用户确认；仅当要素缺失且无法合理提取时才问一次最小信息（WAIT_USER）。本条教训的正确读法是"信息缺失时不得凭猜测推进"，而非"每个节点都要确认"。
3. **trace 取证不可用时默默走静态分析**：AI 没有告知用户有"安装 trace 取证工具走动态调试"的选项。**已优化**：trace 取证不可用时不默默走静态分析，必须先告知用户有安装选项，由用户选择。**（2.3.24 修正注记）**：此处的"由用户选择安装方式"已撤回——取证工具按 GATE-1 自动安装（`install_all.js --yes`，执行前先宣布缺失组件、安装目标与规模），仅自动安装失败后才由用户选择。本条教训的正确读法是"不得静默降级、不得绕过 trace 取证"，而非"必须让用户先选择安装方式"。
4. **静态分析效率问题**：2.25MB 压缩 JS 需要多次 Grep + 上下文提取才能定位算法，效率不如 trace 取证的 `get_request_initiator` 直接定位签名函数。

### 静态分析 vs 动态调试

| 维度 | 纯静态分析（降级） | trace 取证动态调试（首选） |
|---|---|---|
| 定位签名函数 | Grep 搜索 + 上下文提取（多次试探） | `get_request_initiator` 一步到位 |
| 算法验证 | 需手动比对中间值 | Hook 函数直接观察输入输出 |
| 大文件处理 | 2MB+ 压缩 JS 难以阅读 | 不需要读源码，动态跟踪 |
| 适用场景 | 算法简单、参数名明确、JS 结构清晰 | 所有场景，尤其是复杂场景 |
| 依赖 | 仅 Node.js | 需 trace 取证工具 |

**结论**：对于参数名明确、JS 结构清晰的场景，静态分析是合理选择；对于大文件或参数名不明确的场景，trace 取证动态调试效率更高。trace 取证不可用时不默默走静态分析，应告知用户有安装选项。

## 交付物

- `final.js` — 纯 Node.js 协议脚本，依赖仅 `sm-crypto`
- 运行：`node final.js [positionId]`

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/crypto/crypto-patterns.md` | SM2/SM4/SM3 国密算法识别与还原 |
| `references/workflow/decision-tree.md` | 题型判定（标准国密 → 纯算路径 A） |
| `references/workflow/trace-flow.md` | 静态分析 vs trace 取证动态调试取舍 |
| `references/network/tls-validation.md` | 仅当怀疑 TLS 指纹时才看 |
