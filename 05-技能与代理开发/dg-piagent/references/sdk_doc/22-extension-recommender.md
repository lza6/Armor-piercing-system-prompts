# 22. 扩展推荐 SOP

> **定位**:用户主动询问「有没有能做 XX 的 Pi 扩展」时,AI 按本 SOP **实时查询 npm** 给出推荐清单 + 安装命令。**不依赖任何静态归档**——每次推荐都来自 npm 当下数据。

---

## 何时触发

**主动询问型**场景：

- 用户说「有没有能 XX 的 pi extension / 扩展」
- 用户说「推荐个能做 XX 的包」
- 用户说「pi 有现成的 XX 工具吗」
- 用户提到 `pi install` 但不知道装哪个

**不触发的场景**（避免越权推荐）：

- 用户在写自己的扩展（这是开发任务，走 [07-extensions-api.md](07-extensions-api.md)）
- 用户在排查已有扩展 bug（这是调试任务）

---

## 核心 SOP

### 第 1 步：构造 npm 查询

**API 端点**：

```
https://registry.npmjs.org/-/v1/search
```

**URL 模板**（关键词搜索 + 分页）：

```
?text=keywords:pi-package+{用户需求词}&size=50
```

**调用方式**（任选其一）：

```bash
# 方式 A：curl（Bash 工具）
curl -s "https://registry.npmjs.org/-/v1/search?text=keywords:pi-package+memory&size=50"

# 方式 B：WebFetch（如果 Bash 受限）
WebFetch(url, "提取所有 objects[].package 的 name/version/description/keywords/date")
```

**关键词构造规则**：

| 用户意图 | keywords 构造 |
|---------|--------------|
| 「想做 web 抓取」 | `keywords:pi-package+web+fetch` 或 `keywords:pi-package+scraping` |
| 「想持久化记忆」 | `keywords:pi-package+memory` |
| 「想要子 Agent」 | `keywords:pi-package+subagent` 或 `keywords:pi-package+multi-agent` |
| 「MCP 集成」 | `keywords:pi-package+mcp` |
| 模糊需求 | 先用最宽泛的 `keywords:pi-package+{核心词}`，结果太少再换近义词 |

**多关键词组合**：npm search 的 `text` 参数做全文相关性匹配（非严格 AND），`+` 是 URL 编码的空格。`keywords:pi-package+memory` 表示「keywords 含 pi-package，且文本整体相关性匹配 memory」——结果集不会缩小，但含 memory 的包排序更靠前。若需严格 AND 两个关键词，用 `text=keywords:pi-package+keywords:memory`。

**替代查询入口**：也可直接浏览 [pi.dev/packages](https://pi.dev/packages) 官方包画廊，展示所有带 `pi-package` 标签的包，支持视频/图片预览。

### 第 2 步：过滤与排序

**过滤建议**：

```python
# keywords 必须含 "pi-package"（已在查询条件中保证，此处做二次确认）
# 很多有效 pi 包（如 pi-subagents）不含 "extension"/"pi-extension" 关键词，不要强制排除
# 如需进一步排除工具库/本体，看 description 和 keywords 是否与扩展场景相关
```

**排序优先级**（npm 已经返回 `searchScore`，但还需要交叉验证）：

1. `searchScore` 高（npm 内置相关度）
2. `date` 新（最近更新，避免僵尸包）
3. 月下载量高（社区活跃度，**需要第 3 步单独取**）

**Top N**：取 **5-8 个**。多了用户决策疲劳，少了缺选择。

**关于 npm 的 `score` 字段**：npm search 还返回 `score.final` 和 `score.detail.{popularity,quality,maintenance}`，但**实测**：① `score.final` 与 `searchScore` 是同一个数（两者逐项相等，区间在 38-44 量级，**不是** 0-1 归一化评分）；② `score.detail` 三项对 pi-package 切片**常恒为整数 `1`**，无区分度。**结论：直接用 `searchScore` 即可，无需把 `score.*` 当独立交叉参考**（重复参考等于做无用功）。

### 第 3 步：下载量字段（已实测，npm search 直接返回）

**实测结果**：npm search 接口**已经返回下载量字段**，**通常无需额外请求**：

```json
{
  "package": { "name": "pi-memory", ... },
  "downloads": { "monthly": 13842, "weekly": 6079 },
  "dependents": "1",
  "updated": "2026-07-31T10:14:45.670Z",
  "searchScore": 254.89743
}
```

直接读 `downloads.monthly` 字段即可。

**备用方案**（仅当 search 接口未返回 downloads，或要精确单包验证时）：

```bash
# 单包月下载量（含时间区间）
curl -s "https://api.npmjs.org/downloads/point/last-month/pi-web-access"
# 返回：{"downloads":175572,"start":"2026-07-04","end":"2026-08-02","package":"pi-web-access"}
```

**显示下载量的价值**:影响用户判断「这个包是真有人在用还是个人实验」。pi-web-access 月下载 175K,pi-memory 月下载 13K,差异是决策关键依据。

### 第 4 步：输出推荐表（固定模板）

```
根据您的需求「{用户原话}」，找到以下候选扩展：

| # | 包名 | 简介 | 月下载 | 最后更新 | 推荐理由 |
|---|------|------|--------|----------|----------|
| 1 | [xxx](https://www.npmjs.com/package/xxx) | 一句话简介 | 1.2K | 2026-07-15 | 针对用户场景的一句话 |
| 2 | ... | ... | ... | ... | ... |

> ⚠️ **安全提示**：Pi 扩展运行时拥有**完整系统访问权限**（执行任意代码，可调用 bash/edit 等工具，甚至带 postinstall 脚本）。安装第三方包前请点开 npmjs.com 包页面**审查源码**，尤其留意 README 中的权限声明、bash/edit 工具调用与 `postinstall` 钩子。对未知作者或下载量极低的包格外谨慎。

## 推荐安装

最推荐 **{包名}**，理由：{一句话}。

```bash
pi install npm:{包名}
```

装完后可以用 `pi list` 验证已写入 settings。

如果想试装不持久化，用 `pi -e npm:{包名}` 临时加载到当前会话。**注意**：`pi -e` 临时加载的包**不会**出现在 `pi list`（它不写 settings，仅当前会话生效）。
```

**输出要求**：

- 每个包名必须**带 npmjs.com 包页面超链接**（方便用户核实，反幻觉关键）
- 「推荐理由」必须针对用户场景写，不能照搬 npm description
- 安装命令必须用代码块包裹，方便用户复制
- **不允许**凭训练数据印象补全包名/简介——一切信息必须来自本次查询的真实返回

---

## 反幻觉护栏（关键 ⚠️）

Pi 扩展生态发展快，新包不断出现，旧包可能改名/废弃。AI 训练数据**必然滞后**。必须遵守：

1. **推荐的每个包**：必须来自本次 npm API 真实返回的 `objects[]`
2. **包名/简介/下载量**：必须与本次 API 返回字段一致，不允许 AI 改写或"补全"
3. **训练数据中的"印象"**：可以作为**构造查询关键词的参考**（"我记得有个 pi-xxx 之类的"），但**不能直接作为推荐结果**——必须重新走 npm 查询验证存在性
4. **API 失败时**：老实说「npm 查询失败」，不要凭记忆编推荐
5. **拿不准时**：附上 npmjs.com 包页面链接，让用户自己点开核实
6. **安全提示必附**：每次推荐表都必须带「安装前审查源码」的安全提示（见第 4 步模板），**不允许**只甩 `pi install` 命令引导用户盲装未审计的任意代码包

---

## 退化策略

| 用户输入 | 应对 |
|---------|------|
| 太宽泛（"推荐个扩展"） | 反问：「什么场景？web 抓取 / 记忆 / 子 Agent / MCP / ...」 |
| 完全没匹配 | ① 拓宽关键词（"memory" → "context" / "storage"）<br>② 实在没合适的，建议「自己写扩展」+ 链接到 [07-extensions-api.md](07-extensions-api.md) |
| 候选太多（>20 个相关） | 收紧关键词，明确告诉用户「我收紧了筛选条件，从 N 个收到 M 个」 |
| 用户继续追问对比 | 用 `npm view xxx` 或抓 npmjs.com 包页面拿 README 详细信息 |

---

## 与其他文档的边界

| 想做什么 | 看哪份 |
|---------|--------|
| **找现成扩展装** | 本文件 |
| **自己写扩展** | [07-extensions-api.md](07-extensions-api.md)（ExtensionAPI 接口） |
| **理解扩展加载机制** | [20-pi-package.md](20-pi-package.md)（pi install / npm:source 解析机制） |
| **看扩展层事件** | [04-events.md](04-events.md)（pi.on 派发） |
| **扩展集成实战踩坑** | [E02-extension-basics.md](../scenarios/E02-extension-basics.md) |

---

## 维护说明

本文件**不需要定期同步扩展列表**——因为走实时查询路线，每次推荐都来自 npm 当下数据。仅需在以下情况更新：

- npm API 接口变更（端点/字段/分页规则）
- 新增查询技巧（如更精准的关键词构造模式）
- 反幻觉护栏出现新失效模式（实战中发现 AI 编造包名的新套路）
