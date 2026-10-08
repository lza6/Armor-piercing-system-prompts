# 证据可信度矩阵

本文件用于判断某个环境值、入口结论或参数依赖的可信程度，避免把猜测写成事实。所有关键结论、fixture 值、入口定位、参数依赖都必须标注可信度等级。

## 可信度等级

| 等级 | 来源 | 使用方式 |
|---|---|---|
| A | 浏览器真实运行样本、DevTools 断点、Hook 命中调用栈、RuyiTrace NDJSON 轨迹 | 可作为 fixtures 和入口证据，直接用于补环境 |
| B | HAR / cURL / Network 导出、sourcemap 定位、ruyiPage trace_function 输出 | 可作为强证据，但需复核动态值 |
| C | 静态搜索、格式化代码、webpack module 名称、AST 分析 | 可作为线索，不单独定论 |
| D | 根据常见模式推断、经验猜测、UA 或平台默认值 | 只能写入"疑似"，必须继续验证 |

## 环境值采信顺序

环境值（navigator、screen、WebGL、Canvas、Cookie、Storage 等）的采信优先级：

1. 浏览器运行时读取到的真实值（RuyiTrace / Hook / DevTools）。
2. 成功请求样本中携带的值（HAR / cURL / Network 导出）。
3. 页面 HTML / script / meta / window 配置中的值。
4. JS 静态常量。
5. 根据 UA 或平台推断的默认值。

采信时必须记录来源等级，不要把低等级来源的值当作高等级证据使用。

## 入口定位采信顺序

加密入口（source → entry → builder → writer 链路）的采信优先级：

1. **source（参数出口）**：Hook 命中 + 调用栈（A 级）。
2. **writer（写入请求）**：Hook XHR / fetch / Cookie + 调用栈（A 级）。
3. **builder（参数构建）**：trace_function 追踪 + 断点数据（A/B 级）。
4. **entry（加密入口）**：调用栈反推 + sourcemap 定位（B/C 级）。
5. **source（源码位置）**：静态搜索 + 代码格式化（C 级）。

entry 和 source 的定位如果只有 C 级证据，必须在阶段报告中标注"待验证"，并通过 Hook 或 trace 升级到 A/B 级后才能用于补环境。

## 输出要求

### 关键字段标注

每个关键 source 字段都标注来源等级：

```markdown
## 环境值采信

- navigator.userAgent：A 级（RuyiTrace 采样，baselineId=fp-20260627-001）
- navigator.platform：A 级（RuyiTrace 采样）
- screen.width：A 级（Hook 采样）
- WebGL vendor：B 级（HAR 未包含，从 RuyiTrace 属性访问推断）
- Canvas fingerprint：A 级（fixture 回放）
- Cookie sessionid：B 级（cURL 样本，需复核是否过期）
- localStorage token：C 级（静态搜索发现，未在运行时确认）
```

### 不确定字段处理

- 不确定字段写"未知 / 待验证"，不要用默认值掩盖。
- Cookie、token、localStorage 敏感值只写键名、用途和脱敏摘要。
- 若结果依赖 D 级推断，不能宣称补环境完成。
- D 级推断必须标注下一步验证计划。

### 阶段报告模板

```markdown
## 证据可信度评估

- A 级证据数量：
- B 级证据数量：
- C 级证据数量：
- D 级推断数量：
- 是否存在未验证的 D 级推断：是 / 否
- 下一步验证计划：
- 是否可以进入补环境：是 / 否（原因：）
```

## 与其他文件的关系

- 指纹基线一致性：见 `fingerprint/fingerprint-baseline-consistency.md`
- 补环境调试循环：见 `env/env-debug-loop.md`
- 加密入口四层链路模型：见 `crypto/crypto-entry.md`
- Hook 模板与证据输出：见 `hooks/hook-templates.md`
