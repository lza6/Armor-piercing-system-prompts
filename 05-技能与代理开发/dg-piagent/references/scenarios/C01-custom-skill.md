# 场景：自定义 Skill (C01)

## 什么时候用 / 不用会怎样

**适合**：当你需要让 Agent 在特定项目中遵循一套专门的工作流、编码规范、领域知识时——例如「本项目所有 API 都要走 /v2 前缀」「数据库 schema 在这里」「部署流程是 X→Y→Z」。Skill 把这些指令以 Markdown 形式注入系统提示词的索引区，Agent 按需用 read tool 读取正文。

**不适合**：
- 简单单条指令 → 用 `systemPrompt` 直接写一行就够，不必包一个 SKILL.md
- 需要在运行时动态切换的指令 → Skill 是会话启动时加载的，不能中途换；考虑 [E05-input-transform](E05-input-transform.md)
- 大段参考数据（schema、API 文档）→ Skill 索引只放摘要，正文仍占 context；数据量大时用 [G01-context-injection](G01-context-injection.md) 按需注入

**不用会怎样**：Agent 只能凭通用知识 + 当前对话上下文猜项目约定，容易绕过规范、反复用工具探查已知信息。一次简单查询可能多消耗 5+ 次工具调用。

## 默认行为（不传 `skillsOverride` 时）

`createAgentSession` 不显式传 `resourceLoader` 时，SDK 内部 `new DefaultResourceLoader(...)` 会自动发现 Skill。实际由 `packageManager.resolve()` 收集所有路径后合并传给 `loadSkills({ includeDefaults: false })`，收集顺序（受信任门槛的标注）：

1. **项目目录** `<cwd>/.pi/skills/`（project scope，**受项目信任门槛控制**——见下方专节）
2. **`.agents/skills/` 祖先目录链**（project scope，受 projectTrusted 控制）：packageManager 沿 cwd 向上收集 `.agents/skills/` 目录，**直到命中 git repo root（`.git`）就停**（不在 git 仓库则一直向上到文件系统根）
3. **用户全局** `~/.pi/agent/skills/`（user scope，不受信任门槛控制）
4. **用户全局** `~/.agents/skills/`（user scope，与第 2 步的祖先 `.agents` 链对应，但属于 user 范围）
5. **显式路径** `additionalSkillPaths` 中的路径（追加到最后）

> 💡 全局目录 `~/.pi/agent/skills/` 可被环境变量 `PI_CODING_AGENT_DIR` 覆盖（该 env 名由 `APP_NAME` 派生，默认 `pi`）。`.pi` 目录名默认可被 `package.json` 的 `piConfig.configDir` 覆盖。

每个目录的发现规则：
- 目录下含 `SKILL.md` → 作为 Skill 根，**不再递归**
- 否则扫描直接 `.md` 子文件
- 进入子目录递归查找 `SKILL.md`
- `node_modules` / `.`开头的隐藏目录 / `.gitignore` 匹配项 → 跳过

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `DefaultResourceLoader` 的 `skillsOverride` | 过滤/替换/追加 Skill 列表 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `additionalSkillPaths` 选项 | 不改写列表，只加路径 | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `noSkills: true` | 跳过默认目录扫描（CLI 注入路径与 `additionalSkillPaths` 仍生效，见变体 B） | [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md) |
| `Skill` / `createSyntheticSourceInfo` | 创建虚拟 Skill 对象（不落地文件） | [sdk_doc/09-skills.md](../sdk_doc/09-skills.md) |
| `loadSkills()` | 不走 ResourceLoader，独立加载查询 | [sdk_doc/09-skills.md](../sdk_doc/09-skills.md) |
| `loader.getSkills()` | 查询已加载的 Skill 列表 + diagnostics | [sdk_doc/09-skills.md](../sdk_doc/09-skills.md) |

## 核心数据模型

### Skill 接口

> ⚠️ **跨包类型差异**：下方是 `@earendil-works/pi-coding-agent` 的 `Skill` 接口（本文档示例 import 的就是这个包）。若改用 `@earendil-works/pi-agent` 的 `Skill`（`agent/src/harness/types.ts`），字段集**完全不同**：没有 `sourceInfo`、没有 `baseDir`，`disableModelInvocation` 是可选 `?`，且多了一个 `content: string` 字段。两个包的 `Skill` 不能混用，照抄时务必确认 import 来源。

```ts
interface Skill {
  name: string;                    // 引用名 + /skill:name 显式调用
  description: string;             // 功能说明，会进系统提示词索引
  filePath: string;                // SKILL.md 路径，模型 read tool 按此读取正文
  baseDir: string;                 // 基础目录，相对路径以此为根
  sourceInfo: SourceInfo;          // 来源元信息
  disableModelInvocation: boolean; // true = 不进索引 XML，只能 /skill:name 调用（frontmatter 中为 `disable-model-invocation: true`）
}
```

**字段约束**：
- `name`：必须仅含 `[a-z0-9-]`，最长 64 字符；不能以 `-` 开头/结尾；不能含连续 `--`。**frontmatter 缺少 `name` 时，回退到 SKILL.md 所在目录名**。违规会触发 `ResourceDiagnostic` warning，Skill 仍加载但行为不可预期。**跨包注意**：以上是 `@earendil-works/pi-coding-agent` 的校验规则；若改用 `@earendil-works/pi-agent` 的 `loadSkills`，name **还必须与父目录名完全一致**，否则会额外报 `name "..." does not match parent directory "..."` warning
- `description`：最长 1024 字符；为空或纯空白 → Skill **不会被加载**（返回 null）
- `filePath`：**强烈建议命名为 `SKILL.md`**——自动发现只识别这个文件名；虚拟 Skill 也建议沿用，便于和其他工具链兼容
- `disableModelInvocation`：**必填字段**，TS 严格模式省略会编译失败

### createSyntheticSourceInfo 签名

```ts
function createSyntheticSourceInfo(
  path: string,
  options: {
    source: string;                                // 来源标识，如 "sdk" / "local" / 自定义字符串
    scope?: "user" | "project" | "temporary";      // 默认 "temporary"
    origin?: "package" | "top-level";              // 默认 "top-level"
    baseDir?: string;
  }
): SourceInfo
```

**注意**：第二参数是 `options` 对象，**不是字符串、不是文件内容**。返回的 `SourceInfo` 会原样存进 `Skill.sourceInfo`，但 `formatSkillsForPrompt` 注入索引时只读 `name`/`description`/`filePath`，**不读 `sourceInfo`**——所以 `sourceInfo` 主要用于 diagnostics 和 source 追溯（debug "这个 Skill 从哪来"），不影响模型可见内容。

## 实现思路

1. 创建 `DefaultResourceLoader`，传入 `skillsOverride` 回调
2. 在回调中对现有 Skill 列表做过滤/替换/追加操作
3. 创建自定义 `Skill` 对象（必填字段：`name` / `description` / `filePath` / `baseDir` / `sourceInfo` / `disableModelInvocation`）
4. `await loader.reload()` 后调用 `loader.getSkills()` 查看结果——**注意返回值是 `{ skills, diagnostics }` 对象，不是数组**

## 核心代码

```ts
import {
  createAgentSession,
  createSyntheticSourceInfo,
  DefaultResourceLoader,
  getAgentDir,
  SessionManager,
  type Skill,
} from "@earendil-works/pi-coding-agent";

const cwd = process.cwd();
const agentDir = getAgentDir();

// 1. 创建虚拟 Skill（不落地文件）
//    filePath 建议命名为 SKILL.md；sourceInfo 第二参数是 options 对象
const customSkill: Skill = {
  name: "my-deploy-skill",
  description: "Deploy the project to production: build → test → ship",
  filePath: "/virtual/skills/my-deploy-skill/SKILL.md",
  baseDir: "/virtual/skills/my-deploy-skill",
  sourceInfo: createSyntheticSourceInfo(
    "/virtual/skills/my-deploy-skill/SKILL.md",
    { source: "sdk", scope: "temporary" }
  ),
  disableModelInvocation: false,  // false = 进索引让模型自动调用；true = 仅 /skill:name 显式
};

// 2. 用 skillsOverride 过滤 + 追加
//    回调签名：(base: { skills, diagnostics }) => { skills, diagnostics }
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  skillsOverride: (base) => {
    const filtered = base.skills.filter(s =>
      s.name.includes("search") || s.name.includes("browser")
    );
    return { skills: [...filtered, customSkill], diagnostics: base.diagnostics };
  },
});
await loader.reload();

// 3. 查看已加载 Skill
//    getSkills() 返回 { skills, diagnostics }，不是 Skill[]
const { skills, diagnostics } = loader.getSkills();
console.log("Loaded skills:", skills.map(s => s.name));
if (diagnostics.length > 0) {
  console.log("Diagnostics:", diagnostics);
}

const { session } = await createAgentSession({
  resourceLoader: loader,
  sessionManager: SessionManager.inMemory(),
});
```

## 变体

### 变体 A：不改写列表，只加路径

适合「我就想多加载一个目录的 SKILL.md，别的不动」的场景。无需 `skillsOverride`：

```ts
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  additionalSkillPaths: ["/abs/path/to/extra-skills-dir"],
});
await loader.reload();
```

`additionalSkillPaths` 是路径数组，可以是文件也可以是目录；目录会被递归扫描发现 `SKILL.md`。不存在的路径会写入 `diagnostics` 而不是 throw。**注意 severity 差异**：DefaultResourceLoader 对 additionalSkillPaths 中的**本地路径**（`isLocalPath(p)` 为真）不存在时推 `{ type: "error" }`（远程/npm 源不走此分支）；独立 `loadSkills()` 中对 skillPaths 任意不存在路径推 `{ type: "warning" }`。

### 变体 B：关闭 Skill 自动发现

`noSkills: true` 跳过默认目录扫描（user scope + project scope），但 `additionalSkillPaths` 和 CLI 注入的 Skill 路径**仍会生效**。要完全清零，需 `noSkills: true` 且不传 `additionalSkillPaths`：

```ts
const loader = new DefaultResourceLoader({
  cwd,
  agentDir,
  noSkills: true,
});
```

### 变体 C：不走 ResourceLoader，独立加载查询

适合「先看看有哪些 Skill 再决定怎么集成」的探测场景：

```ts
import { loadSkills, formatSkillsForPrompt } from "@earendil-works/pi-coding-agent";

const result = loadSkills({
  cwd,
  agentDir,
  skillPaths: ["/extra/skills"],
  includeDefaults: true,
});
console.log(`Found ${result.skills.length} skills`);
console.log(`Diagnostics: ${result.diagnostics.length}`);

// 看 prompt 注入预览
console.log(formatSkillsForPrompt(result.skills));
```

## 关键陷阱

### 陷阱 1：Skill 正文不自动注入，必须配 read 工具

**这是最常踩的坑**。`formatSkillsForPrompt()` 只把 Skill 的**索引**（name + description + location）以 XML 形式注入系统提示词，正文 SKILL.md 从不自动注入。索引开头有四行指令文本（含一个空行），其中两条关键指令是：

> Use the read tool to load a skill's file when the task matches its description.
>
> When a skill file references a relative path, resolve it against the skill directory (parent of SKILL.md / dirname of the path) and use that absolute path in tool commands.

第二条指令解释了为什么 `baseDir` / `filePath` 字段重要——模型被告知把 SKILL.md 里的相对路径解析到 skill 所在目录。随后才是 `<available_skills>` XML 块。

模型看到相关 description 后应主动用 read tool 读 `<location>` 指向的 SKILL.md 全文。**但如果 `createAgentSession({ tools: [...] })` 的工具白名单不含 `read`**（例如只配了业务工具 `["execute_sql"]`），模型永远拿不到正文，Skill 形同虚设。

```ts
// ❌ 错误：Skill 加载了但模型读不到正文
const { session } = await createAgentSession({
  resourceLoader: loader,
  tools: ["execute_sql"],          // 少了 read
});

// ✅ 正确
const { session } = await createAgentSession({
  resourceLoader: loader,
  tools: ["execute_sql", "read"],  // read 让模型按需加载 Skill 正文
});
```

**验证方法**：打印 `formatSkillsForPrompt(loader.getSkills().skills)` 看实际注入——应该只看到索引 XML，看不到正文。

> 内置 read tool 不支持路径白名单（详见 [sdk_doc/06-tools.md](../sdk_doc/06-tools.md) 的「内置工具的安全边界」节）。项目含 `.env`、源码等敏感文件时，建议用 `createReadToolDefinition()` 包装一层路径校验。

### 陷阱 2：`skillsOverride` 在默认加载后才执行

`skillsOverride` 拦截的是 `loadSkills()` 的最终输出——默认目录扫描、`additionalSkillPaths` 都先跑完，再传给回调。回调里返回的 `{ skills, diagnostics }` 就是最终结果，没有后续合并。

这意味着：**回调里不返回某 Skill，它就真的没了**——即便它在默认目录里存在。如果只想追加、不想过滤，记得把 `base.skills` 完整传出去：

```ts
skillsOverride: (base) => ({
  skills: [...base.skills, customSkill],   // 保留默认 + 追加
  diagnostics: base.diagnostics,
})
```

**同名 Skill 冲突**：两个 Skill 同名时，先加载的胜出，后加载的被静默丢弃并写入 `{ type: "collision" }` diagnostic。默认 Skill 先于 additionalSkillPaths 加载，所以自定义同名 Skill **不会覆盖**默认 Skill——需在 skillsOverride 中显式过滤。

### 陷阱 3：项目级资源受「信任门槛」控制

`<cwd>/.pi/skills/` 下的 Skill 受 `projectTrusted` 控制。不信任时 **DefaultResourceLoader** 会跳过 project scope 的扫描（packageManager 在 `projectTrusted=false` 时不收集 `.pi/skills/` 路径）。注意：独立调用的 `loadSkills({ includeDefaults: true })` **不检查信任门槛**，始终扫描 project 目录。编程式 API 中 `SettingsManager` 默认 `projectTrusted=true`，可通过 `SettingsManager.setProjectTrusted(true)` 显式设置。（交互式 CLI 模式另有信任询问 UI，详见 [B04-project-trust.md](B04-project-trust.md)。）

详见 [B04-project-trust.md](B04-project-trust.md) 与 [sdk_doc/13-settings-manager.md](../sdk_doc/13-settings-manager.md)。

### 陷阱 4：`createSyntheticSourceInfo` 第二参数是对象不是字符串

签名是 `(path, options: { source, scope?, origin?, baseDir? })`。**`source` 字段必填**，`scope` / `origin` / `baseDir` 可选但有默认值。传错类型会导致 TS 编译失败或运行时行为异常。

```ts
// ❌ 错误：第二参数不是字符串
createSyntheticSourceInfo("/virtual/SKILL.md", "# My Skill content...")

// ❌ 错误：source 是必填字段
createSyntheticSourceInfo("/virtual/SKILL.md", { scope: "user" })

// ✅ 正确
createSyntheticSourceInfo("/virtual/SKILL.md", { source: "sdk", scope: "temporary" })
```

### 陷阱 5：`getSkills()` 返回对象不是数组

```ts
// ❌ 错误：getSkills() 返回 { skills, diagnostics }，直接 .map 会报错
const skills = loader.getSkills();
skills.map(s => s.name);  // TypeError: skills.map is not a function

// ✅ 正确
const { skills, diagnostics } = loader.getSkills();
skills.map(s => s.name);  // OK
```

### 陷阱 6：`customPrompt` 不是最终 prompt（A03 横向）

Skill 不是孤立系统——它是 `buildSystemPrompt()` 组装链的一环。即便 Skill 正确加载，如果同时传了 `systemPrompt` / `appendSystemPrompt` / `systemPromptOverride` / Agents.md 文件，最终系统提示词是这些的合成。如果你发现 Skill 注入「没生效」，先用 `session.getResourceLoader().getSkills()` 确认加载成功，再检查是否有其他 prompt 源覆盖了模型注意力。详见 [A03-system-prompt.md](A03-system-prompt.md)。

## 集成踩坑：Skill 名称校验

`name` 字段看似自由文本，实际有严格规则（完整规则见上方「核心数据模型 / 字段约束」）：

- 只允许 `[a-z0-9-]`，**大写字母、下划线、点号都非法**
- 最长 64 字符
- 不能以 `-` 开头或结尾
- 不能含连续 `--`

违规不会 throw，但会写入 `diagnostics`（type: `"warning"`），Skill 仍会加载——**模型行为不可预期**，因为 `/skill:<name>` 命令解析也依赖此规则。

```ts
// ❌ 错误示例
{ name: "MySkill", ... }        // 含大写
{ name: "my_skill", ... }       // 含下划线
{ name: "my.skill", ... }       // 含点号
{ name: "-my-skill", ... }      // 以 - 开头
{ name: "my--skill", ... }      // 含连续 --

// ✅ 正确
{ name: "my-skill", ... }
{ name: "deploy-v2", ... }
```

## 横向联动

- Skill 系统详解（结构、来源、注入机制、`loadSkills` 独立 API） → [sdk_doc/09-skills.md](../sdk_doc/09-skills.md)
- ResourceLoader 完整选项（`additionalSkillPaths` / `noSkills` / 其他 override） → [sdk_doc/08-resource-loader.md](../sdk_doc/08-resource-loader.md)
- 同时加载 Prompt 模板 → [场景 C02](C02-prompt-templates.md)
- 同时加载 Context Files（AGENTS.md / CLAUDE.md） → [场景 C03](C03-context-files.md)
- 系统提示词的合成链 → [场景 A03](A03-system-prompt.md)
- 项目信任门槛（`.pi/` 资源生效前提） → [场景 B04](B04-project-trust.md)
- 让 Skill 正文按需注入的策略 → [场景 G01](G01-context-injection.md)
