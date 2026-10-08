# Case 模板

本文件包含**两部分，用途不同**，请按场景取用：

- **Part 1 · 工作目录初始化**：开一个新 case *开始干活* 时，参照此结构在工作区（`case/`）建目录、填 Case 信息与检查清单。这部分**不进经验库**。
- **Part 2 · 经验库归档条目**：任务*结束后沉淀经验*时，复制 Part 2 的骨架填写。先落到 `result/经验沉淀-<站点>.md`（运行期 skill 目录只读），再由维护者周期性并入 `cases/`。归档条目**必须与库内现有案例格式一致**。

---

# Part 1 · 工作目录初始化（干活时用）

## 目录结构

> `<case 根>` 下只有 `case/` 和 `result/` 两个平级子目录（SKILL.md 目录约束）。

```text
<case 根>/
├── case/
│   ├── 阶段报告/          # 仅复杂补环境或用户明确要求时按需生成
│   │   └── 01-需求信息确认.md
│   ├── js/
│   │   ├── original/      # 原始 JS 文件
│   │   ├── pretty/        # 格式化后的 JS
│   │   └── extracted/     # 提取的关键代码片段
│   ├── requests/
│   │   └── request.curl   # Copy as cURL 样本
│   ├── fixtures/
│   │   ├── sample.fixture.json    # 浏览器真实输出 fixture
│   │   └── fingerprint.fixture.json # 指纹采样 fixture（如涉及）
│   ├── notes/
│   │   ├── 代码变更记忆.md
│   │   ├── entry-chain.md # source/entry/builder/writer 链路
│   │   ├── silent-failure-checklist.md
│   │   └── trust-matrix.md
│   ├── ruyi-trace/        # Trace 日志（如使用）
│   │   └── logs/
│   ├── hooks/             # 临时 Hook 脚本（用完即删）
│   ├── env/               # 补环境代码（开发阶段）
│   └── tmp/               # 临时文件（用完即删）
└── result/                # 最终交付（必选，与 case/ 平级）
    ├── final.js
    ├── 最终项目总结.md
    ├── 经验沉淀-<站点>.md  # 必选（按 Part 2 格式）
    └── src/
```

## Case 信息（工作区记录用）

```markdown
# Case 信息

## 目标
- 目标网站 URL：
- 目标页面 URL：
- 目标 API：
- 请求方法：
- 目标加密参数：
- 参数位置：Query / Header / Body / Cookie

## 取证
- 取证来源：ruyipage 网络取证（forensic_ruyipage.py）/ RuyiTrace 日志采集 / 用户手动提供材料
- 最终请求 TLS 客户端：Node.js CycleTLS（需手动实现，client.js 未内置）/ impers / curl-cffi / Python curl_cffi / cffi_curl / cyCronet / 不发真实请求

## 指纹基线
- baselineId：
- baseline 文件：case/notes/fingerprint-baseline.json

## 补环境框架
- 选择：不使用（默认） / vm / sdenv / jsEnv
```

## 阶段报告清单

> **阶段报告默认不生成**。仅多轮复杂补环境 case 或用户明确要求时按需生成到 `case/阶段报告/`（详见 `references/quality/stage-reports.md`）。以下清单供需要时参考；文件名沿用脚本 `--stage` token，"确认"仅为记录含义——取证来源与目标参数按第 1 节确认策略由门禁/证据自动判定，不要求用户确认。

- [ ] 01-需求信息确认.md
- [ ] 02-取证方案确认.md
- [ ] 03-请求样本与可疑参数确认.md
- [ ] 04-JS文件与入口定位.md
- [ ] 05-补环境前置分析.md
- [ ] 06-补环境实现记录.md
- [ ] 07-验证与清理记录.md

## 交付前检查清单

- [ ] 信息完整性检查通过
- [ ] 证据门禁已通过（check_evidence.js 输出无缺失证据）
- [ ] TLS 客户端已自动探测确认（不要求用户选择）
- [ ] 可疑加密参数已由证据定位（候选列出后不等待用户确认）
- [ ] source/entry/builder/writer 链路已确认
- [ ] 补环境框架已选择
- [ ] fixtures 多样本通过
- [ ] 代码质量检查通过
- [ ] 补环境真实性检查通过
- [ ] 最终产物检查通过
- [ ] 清理 dry-run 通过
- [ ] 最终总结已生成（`result/最终项目总结.md`，必选项）

---

# Part 2 · 经验库归档条目模板（沉淀经验时复制以下整段）

> 复制以下整段作为新案例文件的骨架，文件名以技术特征命名（如 `jsvmp-xxx-yyy.md`）。
> 先写到 `result/经验沉淀-<站点>.md`，再由维护者并入 `cases/`。填完后只在 `cases/index.json` 增加索引记录。

```markdown
# Case：<一句话技术特征描述>

> 难度：★（复杂度提示，非难度定级；★最简 ~ ★★★★★最复杂）
> 还原方案：A 纯算还原 / B vm 沙箱执行 / C WASM 加载 / D 环境伪装（可组合，如"A 纯算 + D 环境伪装"）
> 实现语言：Node.js / Python
> 最后验证日期：YYYY-MM-DD
> 平台类型：<平台名（域名）>

---

## 技术指纹（供 CASE_LOOKUP 自动匹配）

> 列出可用于自动检测的稳定特征，每条写成可搜索的模式。`CASE_LOOKUP` 通过 `cases/index.json` 中的这些特征命中。

- JS 特征：（如 "_0x 前缀变量大量出现"、"单文件 200KB+"、"存在 while-switch 解释器循环"）
- 参数特征：（如 "sign 参数，32 位 hex，疑似 MD5"、"token 参数，Base64 格式"）
- 请求特征：（如 "存在 /api/init 预热请求"、"Cookie 中有动态字段 __ac_xxx"）
- 反调试特征：（如 "debugger 定时器"、"Function.toString 检测"）

## 加密方案

> 一句话说明最终采用的方案路径 + 核心方法，其后可展开算法细节 / 关键常量 / 还原代码。

- 路径：A 纯算还原 / B vm 沙箱执行 / C WASM 加载 / D 环境伪装
- 框架：不使用 / vm / sdenv / jsEnv
- TLS 客户端：CycleTLS（需手动实现）/ curl-cffi-node / impers / curl-cffi / curl_cffi / cyCronet / 不发真实请求
- 核心思路：（一句话）

## 踩坑记录

> 每条写成"坑：___ → 正确做法：___"格式。这些是 IMPLEMENT 编码时必须回查的约束。

1. **坑 1**：（描述）→ 正确做法：（描述）
2. **坑 2**：（描述）→ 正确做法：（描述）

## 可验证事实清单（经验资产）

> 5-15 条最小可验证事实。同站升级时逐条核对找出"哪些变了"。每条写成可断言的形式。

1. （如 "X-Bogus 长度 28"）
2. （如 "navigator.webdriver === false"）
3. （如 "签名输入包含完整 query string，不含 #hash"）

## 相关参考

> 填写与本案例相关的参考文档，建立 cases→references 双向链接。

| 参考文档 | 关联点 |
|---------|--------|
| `references/workflow/decision-tree.md` | 题型判定 + 路径决策 |
| `references/workflow/experience-rules.md` | 相关经验法则编号 |
```

> 中间可按需插入案例专属段落（如「禁动清单」「UA 分支矩阵」「还原代码模板」「变体说明」等），但上述 5 个标准段（技术指纹 / 加密方案 / 踩坑记录 / 可验证事实清单 / 相关参考）**必须齐全**。
