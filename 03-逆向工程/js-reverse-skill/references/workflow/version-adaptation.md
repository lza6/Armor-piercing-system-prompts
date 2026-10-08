# 反爬版本追踪与快速适配

> **触发条件**：已完成的 case 遇到站点 SDK 更新（版本号变化、字节码变化、环境检测项增加），签名失效需要快速适配时读。新 case 完成时也读，用于沉淀可追踪的版本化资产。

## 核心问题

站点更新 SDK 后，之前的逆向成果可能失效。常见变化：

| 变化类型 | 影响 | 检测方式 |
|---|---|---|
| SDK 版本号变化（如 webmssdk 1.0.0.20 → 1.0.0.21） | 可能改了算法或检测项 | 抓包对比 JS URL 版本号 |
| 字节码数组变化 | 签名算法逻辑改变 | 对比 JS 文件 sha256 |
| 环境检测项增加 | 原补环境不够，签名被拒 | 重新 trace 对比 |
| 拦截路径配置变化 | enablePathList 变化 | 检查 _SdkGlueInit 配置 |
| Cookie 字段变化 | 新增 / 改名动态 Cookie | 对比 Cookie 字段集 |
| API 路径变化 | 接口 URL 或参数位置变化 | 抓包对比 |

## 新旧 SDK diff 方法论

### Step 1：确认版本变化

```
下载最新 SDK 文件
计算 sha256
与 case/notes/resource-manifest.json 中记录的旧 sha256 对比
  ├─ 一致 → SDK 未变，问题在别处（Cookie/IP/参数）
  └─ 不一致 → 进入 Step 2
```

### Step 2：文件级 diff

```bash
# 对比两个版本的 SDK 文件
diff <(js-beautify old-webmssdk.js) <(js-beautify new-webmssdk.js) > sdk-diff.txt

# 或用 AST diff（更准确）
node assets/ast-patterns/scripts/run-pipeline.js new-webmssdk.js ./diff-output
node assets/ast-patterns/scripts/compare-with-reference.js --reference ./old-decoded.js --current ./new-decoded.js
```

**重点关注**：
- 字节码数组内容变化（`var X = [3,15,7,...]`）
- 新增的检测代码（`typeof xxx` / `xxx.length` / `Function.prototype.toString`）
- 新增的环境访问（`navigator.xxx` / `window.xxx` / `document.xxx`）
- 拦截器路径配置变化

### Step 3：环境检测项 diff

```
用最新 SDK 在 ruyiPage + RuyiTrace 模式下运行
trace_property_access(mode="summary", collect_values=True)
→ 获得最新读取的属性列表
与旧 case 中记录的 trace 覆盖矩阵对比
  ├─ 新增属性 → 需要补这些属性的环境
  ├─ 删除属性 → 可移除补丁（可选）
  └─ 值变化 → 更新 fixture
```

### Step 4：签名输出对比

```
同一输入（URL + 参数 + Cookie + 时间戳固定）
分别用旧 SDK 和新 SDK 生成签名
  ├─ 签名一致 → 算法未变，只是检测项变了
  └─ 签名不一致 → 算法变了，需要重新分析
```

## 可验证事实清单的版本化使用

每个 case 末尾的"可验证事实清单"是版本追踪的核心资产。格式：

```markdown
## 可验证事实清单

1. a_bogus 长度 180-192 字符
2. navigator.webdriver === false
3. webmssdk.es5.js 必须在 _SdkGlueInit 前加载
4. a_bogus 由 XHR 拦截器注入 URL
5. 58 项环境检测（见 trace 覆盖矩阵）
6. Firefox UA 下 navigator.plugins.length === 5
7. ttwid 为动态生成 Cookie
8. hot_keys 列表长度 42（来自 trace）
```

**版本追踪用法**：

1. **新版本验证**：SDK 更新后，逐条验证可验证事实清单
   - 签名长度变了 → 算法变化
   - 环境检测项数变了 → 补环境需更新
   - 加载顺序变了 → 需要重新分析初始化链路

2. **快速定位变化**：哪条事实失效，就重点排查对应区域

3. **沉淀新版本**：将变化项更新到 case，保留旧版本作为变体说明

## case 的版本化沉淀

在 case 的"变体说明"段记录版本差异：

```markdown
## 变体说明

| 版本 | SDK 版本 | sha256 | 变化点 | 适配策略 |
|---|---|---|---|---|
| v1 | webmssdk 1.0.0.20 | abc123... | 初版 | 58 项补环境 |
| v2 | webmssdk 1.0.0.21 | def456... | 新增 navigator.connection 检测 | 补 navigator.connection stub |
| v3 | webmssdk 1.0.0.22 | ghi789... | 字节码数组更新，算法不变 | 无需改补环境，签名长度一致 |
```

## 快速适配流程

```
签名失效（服务端拒绝）
  │
  ├─ Step 1：确认 SDK 版本变化
  │   下载最新 JS → sha256 对比 → 不一致则进入 Step 2
  │
  ├─ Step 2：文件级 diff
  │   AST diff → 定位变化区域（字节码/检测项/路径配置）
  │
  ├─ Step 3：验证可验证事实清单
  │   逐条验证 → 找到失效的事实
  │
  ├─ Step 4：针对性修复
  │   ├─ 检测项增加 → 重新 trace → 补新属性
  │   ├─ 算法变化 → 重新走 FORENSIC_CAPTURE → TRACE_CAPTURE → TRACE_ANALYZE → IMPLEMENT
  │   └─ Cookie 变化 → 更新 Cookie 生成逻辑
  │
  └─ Step 5：更新 case 变体说明
      记录新版本信息 + 适配策略
```

## SDK 版本号监控建议

| 监控方式 | 实现 | 频率 |
|---|---|---|
| 定期抓包检查 JS URL 版本号 | 脚本化 `fetch(JS_URL)` 解析版本 | 每天/每周 |
| 检查 JS 文件 sha256 | `crypto.createHash('sha256')` | 每次签名失效时 |
| 监控接口响应码 | 200 → 正常，412/403 → 可能 SDK 变化 | 持续 |

**不建议**：主动监控会导致请求频率升高，触发风控。只在签名失效时被动检查。

## 代码变更记忆的版本化复用

`references/quality/code-change-memory.md` 的变更记录可跨版本复用：

- 旧版本的"禁止回退"条目在新版本中通常仍有效
- 旧版本的"已失败尝试"可避免新版本重复踩坑
- 新版本的变更要追加到同一记忆文件，标注版本号

```markdown
## 变更 015 - v2 版本适配 navigator.connection 检测

- 时间：2026-07-09 14:00
- 版本：webmssdk 1.0.0.21（v2）
- 涉及文件：result/src/env/install-env.js
- 修改前逻辑：无 navigator.connection 补丁
- 问题证据：v2 SDK 新增 typeof navigator.connection 检测，返回 undefined 触发风控
- 本次修改：补 navigator.connection stub（{ effectiveType: '4g', rtt: 50, downlink: 10 }）
- 禁止回退：不要移除 navigator.connection 补丁；Firefox UA 下 connection 应为 undefined，但本站点检测的是存在性而非值
```

## 相关参考

| 参考文档 | 关联点 |
|---------|--------|
| `references/quality/code-change-memory.md` | 代码变更记录机制（跨版本复用） |
| `references/network/dynamic-resource.md` | 动态资源清单（sha256 追踪） |
| `references/fingerprint/fingerprint-baseline-consistency.md` | 指纹基线对比（版本间 diff） |
| `cases/_template.md` | case 模板（变体说明段） |
| `assets/ast-patterns/scripts/compare-with-reference.js` | AST diff 工具 |
