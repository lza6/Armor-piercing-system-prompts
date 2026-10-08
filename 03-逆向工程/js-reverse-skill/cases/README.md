# 逆向经验库（Cases）

本目录存放已验证的逆向分析经验案例，供 CASE_LOOKUP 按需检索。

> **本目录是经验库（只读参考）**：运行期 skill 目录通常不可写，**新增经验先沉淀到 `result/`**（按 `_template.md` 的 **Part 2** 格式，文件名如 `result/经验沉淀-<站点>.md`），由 skill 维护者周期性合并进本目录；agent 运行期不要直接写本目录。

## 案例索引

案例的机器可读索引以 [`index.json`](index.json) 为唯一入口，记录标题、域名、技术信号、推荐策略、案例文件和最后验证日期。不要维护或依赖 README 中的手工案例表、关键词速查表，也不要为查找案例而逐个扫描或读取全部 Markdown 文件。

使用 `scripts/search_cases.js` 检索索引：

```powershell
node scripts/search_cases.js <关键词...>
node scripts/search_cases.js --domain jd.com
node scripts/search_cases.js --signal h5st --strategy vm
node scripts/search_cases.js a_bogus --json
```

- 普通关键词会匹配标题、域名、技术信号、策略和文件名。
- `--domain`、`--signal`、`--strategy` 可重复传入；多个条件必须同时命中。
- 匹配不区分大小写，采用子串匹配。
- `--json` 用于需要结构化结果的脚本或 agent；无条件时列出全部索引记录。
- 方法论骨架模板（`kind:template`）默认从结果排除，需用 `--include-templates` 显式包含。
- 参数和输出格式以 `node scripts/search_cases.js --help` 为准。

## CASE_LOOKUP 使用方式

1. 从目标 URL、参数名、SDK 名称、状态码和网络特征中提取最小关键词组合。
2. 运行 `search_cases.js`，只读取命中的案例文件。
3. 从命中案例中提取可复用的定位方法、已知坑点和最后验证日期。
4. 对命中案例执行 JS 资源、内容和参数结构的时效性校验；只有全部一致时才复用算法细节，否则只作为方法论参考。
5. 未命中时直接走标准 INTENT_CONFIRM → ENV_READY → EVIDENCE_GATE → CASE_LOOKUP → IDENTIFY → TRACE_ANALYZE → IMPLEMENT → REAL_VERIFY → DELIVER → CLEANUP → DONE，结束后把新经验沉淀到 `result/`，不写本目录。

## 新增案例

> 运行期不要直接写本目录：新经验先落 `result/经验沉淀-<站点>.md`，由维护者周期性并入本目录。

1. 复制 `_template.md` 的 **Part 2 · 经验库归档条目** 骨架为新文件，以技术特征命名（如 `jsvmp-xxx.md`）。
2. 补全头部元数据块：难度、还原方案、实现语言、最后验证日期、平台类型。
3. 填写 5 个标准段（**缺一不可**）：技术指纹、加密方案、踩坑记录、可验证事实清单、相关参考。
4. 在“可验证事实清单（经验资产）”段列出 5-15 条最小可验证事实。
5. 只在 `index.json` 新增对应记录，确保 `domains`、`signals`、`strategy`、`file`、`verifiedAt` 与案例正文一致。骨架/方法论模板（非具体站点实证）须加 `"kind": "template"` 标记，与实证案例区分。
6. 使用 `search_cases.js` 按域名和核心信号检索，确认新记录可被命中且目标文件存在。
