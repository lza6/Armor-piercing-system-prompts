# 08 - 资源加载器 (ResourceLoader)

`DefaultResourceLoader` 是 pi-agent 的资源加载核心，统一管理 skills、prompt templates、context files（AGENTS.md / CLAUDE.md）、themes 和 extensions 的发现与加载。

**什么时候需要自己构造 ResourceLoader？** `createAgentSession()` 默认会创建 `DefaultResourceLoader`，大多数场景无需手动构造。自己构造的情况：① 自定义系统提示词 / 技能路径 ② 过滤或替换扩展 ③ 运行时动态注入虚拟资源（无需文件）④ 控制项目信任流程。如果只是选模型或调工具白名单，直接用 `createAgentSession` 的参数即可，不需要碰 ResourceLoader。

## ResourceLoader 接口

```ts
interface ResourceLoaderReloadOptions {
  resolveProjectTrust?: (input: { extensionsResult: LoadExtensionsResult }) => Promise<boolean>;
}

interface ResourceLoader {
  getExtensions(): LoadExtensionsResult;
  getSkills(): { skills: Skill[]; diagnostics: ResourceDiagnostic[] };
  getPrompts(): { prompts: PromptTemplate[]; diagnostics: ResourceDiagnostic[] };
  getThemes(): { themes: Theme[]; diagnostics: ResourceDiagnostic[] };
  getAgentsFiles(): { agentsFiles: Array<{ path: string; content: string }> };
  getSystemPrompt(): string | undefined;
  getSystemPromptSource(): { path: string } | undefined;
  getAppendSystemPrompt(): string[];
  getAppendSystemPromptSources(): Array<{ path: string }>;
  extendResources(paths: ResourceExtensionPaths): void;
  reload(options?: ResourceLoaderReloadOptions): Promise<void>;
}
```

> `getExtensions()` 返回的 `LoadExtensionsResult = { extensions: Extension[]; errors: Array<{ path; error }>; runtime: ExtensionRuntime }`。其中 `runtime` 在 loader 阶段是 **throwing stub**（actions 调用即抛错），由 `createAgentSession` 内的 session runner 完成 `runtime.initialize()` 绑定；SDK 使用者一般不直接操作 loader 阶段的 runtime。

## 构造选项

```ts
new DefaultResourceLoader({
  cwd: string;              // 项目根目录，AGENTS.md 向上查找起点、工具执行的基准路径（必填）
  agentDir: string;         // agent 全局配置目录，通常 ~/.pi/agent，存放全局 skills/prompts/themes/SYSTEM.md（必填）
  settingsManager?: SettingsManager; // 设置管理器，默认用 cwd+agentDir 创建，控制项目信任/settings 读写
  eventBus?: EventBus;      // 事件总线，扩展间通信或外部注入事件。默认自动创建，需在 loader 外部 emit/listen 时显式传入

  // 额外资源路径
  additionalExtensionPaths?: string[];
  additionalSkillPaths?: string[];
  additionalPromptTemplatePaths?: string[];
  additionalThemePaths?: string[];

  // 内联扩展工厂（无需文件），接受 ExtensionFactory 函数 或 { name, factory, hidden? } 命名对象
  extensionFactories?: InlineExtension[];

  // 禁用开关
  noExtensions?: boolean;
  noSkills?: boolean;
  noPromptTemplates?: boolean;
  noThemes?: boolean;
  noContextFiles?: boolean;

  // 系统提示词
  systemPrompt?: string;
  appendSystemPrompt?: string[];

  // Override 回调 — 在默认资源加载后、赋值前调用
  extensionsOverride?: (base: LoadExtensionsResult) => LoadExtensionsResult;
  skillsOverride?: (base: { skills: Skill[]; diagnostics }) => { skills: Skill[]; diagnostics };
  promptsOverride?: (base: { prompts: PromptTemplate[]; diagnostics }) => { prompts: PromptTemplate[]; diagnostics };
  themesOverride?: (base: { themes: Theme[]; diagnostics }) => { themes: Theme[]; diagnostics };
  agentsFilesOverride?: (base: { agentsFiles }) => { agentsFiles };
  systemPromptOverride?: (base: string | undefined) => string | undefined;
  appendSystemPromptOverride?: (base: string[]) => string[];
})
```

## 核心工作流

```ts
const loader = new DefaultResourceLoader({
  cwd: "/my/project",
  agentDir: "/home/user/.pi/agent",
});

// 必须调用 reload() 才会加载所有资源
await loader.reload();

// 然后才能获取资源
const { skills } = loader.getSkills();
const { prompts } = loader.getPrompts();
const { agentsFiles } = loader.getAgentsFiles();
const systemPrompt = loader.getSystemPrompt();
const appendPrompts = loader.getAppendSystemPrompt();
const { extensions } = loader.getExtensions();
```

## Override 回调详解

所有 `xxxOverride` 回调在默认资源加载完成后、存储到实例字段前执行。这是 SDK 使用者定制资源的推荐方式。

### systemPromptOverride

完全替换系统提示词：

```ts
new DefaultResourceLoader({
  cwd, agentDir,
  systemPromptOverride: (base) => {
    if (base) {
      return base.replace("You are a helpful assistant", "You are a coding expert");
    }
    return "You are a coding expert";
  },
});
```

注意：`base` 可能为 `undefined`（无 SYSTEM.md 文件时）。

### appendSystemPromptOverride

追加到默认系统提示词末尾：

```ts
new DefaultResourceLoader({
  cwd, agentDir,
  appendSystemPromptOverride: (base) => {
    // base 是 string[]，来自 APPEND_SYSTEM.md
    return [...base, "Always speak like a pirate."];
  },
});
```

### skillsOverride / promptsOverride / themesOverride / agentsFilesOverride

过滤或修改资源集合：

```ts
new DefaultResourceLoader({
  cwd, agentDir,
  skillsOverride: (base) => ({
    ...base,
    skills: base.skills.filter((s) => !s.name.includes("deprecated")),
  }),
  promptsOverride: (base) => ({
    ...base,
    prompts: base.prompts.filter((p) => p.name !== "old-template"),
  }),
  agentsFilesOverride: (base) => ({
    agentsFiles: [...base.agentsFiles, { path: "custom.md", content: "Custom rules..." }],
  }),
});
```

### extensionsOverride

在资源加载完成后调整扩展列表：

```ts
new DefaultResourceLoader({
  cwd, agentDir,
  extensionsOverride: (base) => ({
    ...base,
    extensions: base.extensions.filter((ext) => ext.name !== "disabled_ext"),
  }),
});
```

## extendResources()

运行时动态增加 skills / prompts / themes 路径：

```ts
loader.extendResources({
  skillPaths: [{ path: "/custom/skills/dir", metadata: { source: "local", scope: "project", origin: "top-level" } }],
  promptPaths: [{ path: "/custom/prompts", metadata: { source: "local", scope: "project", origin: "top-level" } }],
  themePaths: [{ path: "/custom/themes", metadata: { source: "local", scope: "project", origin: "top-level" } }],
});
```

该方法不会触发全量 reload，而是将路径合并到现有路径列表后立即重新加载对应类型的全部资源（不只新增路径）。它**只作用于 skills / prompts / themes 三类**，不影响 extensions / 上下文文件（AGENTS.md/CLAUDE.md）/ 系统提示词文件；但因为内部走 `updateXxxFromPaths`，对应的 `skillsOverride` / `promptsOverride` / `themesOverride` 回调会被重新调用。

## 上下文文件自动发现

`loadProjectContextFiles({ cwd, agentDir })` 按以下优先级逐级向上查找：

1. 全局 `AGENTS.md` / `CLAUDE.md` 位于 `agentDir`（约 `~/.pi/agent/`）
2. 从 `cwd` 到根目录逐级递归查找 `AGENTS.md` / `CLAUDE.md`

文件名检测优先级：`AGENTS.md` > `AGENTS.MD` > `CLAUDE.md` > `CLAUDE.MD`。

> git worktree 场景下，主仓库的 AGENTS.md/CLAUDE.md 可能与子 worktree 的同名文件指向同一份 tracked 文件，会被自动去重（`findShadowedContextFile`），避免重复加载。细节见源码 `resource-loader.ts` 的 `findShadowedContextFile`。

## 系统提示词文件

`DefaultResourceLoader` 按以下顺序发现文件：

| 类型 | 文件 | 寻找顺序 |
|------|------|----------|
| 系统提示词 | `SYSTEM.md` | 项目 `.pi/SYSTEM.md`（需项目受信）> 全局 `~/.pi/agent/SYSTEM.md` |
| 追加提示词 | `APPEND_SYSTEM.md` | 项目 `.pi/APPEND_SYSTEM.md`（需项目受信）> 全局 `~/.pi/agent/APPEND_SYSTEM.md` |

也可以在构造时通过 `systemPrompt` / `appendSystemPrompt` 选项直接传入字符串或文件路径，跳过文件发现。传入的值如为已存在的文件路径，会自动读取文件内容；否则视为字面字符串。

完整的系统提示词解析时序（以 `systemPrompt` 为例）：① 构造选项 `systemPrompt`（若为文件路径则读文件）→ ② 否则 `discoverSystemPromptFile()` 发现 `SYSTEM.md` → ③ `resolvePromptInput` 把文件路径转成文件内容 → ④ 最后 `systemPromptOverride`（若有）覆盖。`appendSystemPrompt` 同理（多路径合数组）。

> **项目信任门槛**：项目 `.pi/SYSTEM.md` 和 `.pi/APPEND_SYSTEM.md` 仅在项目受信时生效。项目不受信时，即使文件存在也会被跳过，直接使用全局文件。控制入口是 `reload()` 的 `resolveProjectTrust` 选项。

## reload() 选项与项目信任

`reload()` 接受可选的 `ResourceLoaderReloadOptions`：

```ts
interface ResourceLoaderReloadOptions {
  resolveProjectTrust?: (input: { extensionsResult: LoadExtensionsResult }) => Promise<boolean>;
}
```

`resolveProjectTrust` 在加载用户/全局扩展之后、加载项目级资源之前被调用。它接收不含项目扩展的加载结果，返回 `true` 则标记项目受信——后续会加载项目扩展、项目 SYSTEM.md 等项目级资源。返回 `false` 则跳过所有项目级资源。

典型用法——根据扩展来源决定是否信任项目：

```ts
const loader = new DefaultResourceLoader({ cwd, agentDir });
await loader.reload({
  resolveProjectTrust: async ({ extensionsResult }) => {
    // 项目无扩展 → 自动信任
    if (extensionsResult.extensions.length === 0) return true;
    // 有扩展 → 自定义确认逻辑（弹窗/白名单/配置项）
    return confirm('此项目包含扩展，是否信任？');
  },
});
```

不传 `resolveProjectTrust` 时，项目信任状态由 `settingsManager` 的已有配置决定。

> ⚠️ **默认受信陷阱**：SDK 自己 `new DefaultResourceLoader({ cwd, agentDir })`（即 `createAgentSession` 内部）不传 `settingsManager` 时，会走 `SettingsManager.create(cwd, agentDir)`，其默认 `projectTrusted = true`——也就是项目**默认受信**，`.pi/SYSTEM.md`、`.pi/APPEND_SYSTEM.md`、项目级扩展等项目资源**默认生效**。想关闭项目资源加载，必须显式传 `resolveProjectTrust: async () => false`，或自建一个不受信的 settingsManager 传入。

## 关键注意事项

1. **必须调用 `reload()`**：不调用则所有 getter 返回空数组 / undefined
2. **override 回调先于最终赋值**：input 是默认/文件加载后的值，return 是最终使用的值
3. **`extensionFactories`** 不经过文件系统，直接在 `reload()` 内调用。匿名工厂路径为 `<inline:1>`、`<inline:2>`...（从 1 开始）；命名工厂（`{ name, factory }` 形式）路径为 `<inline:name>`，且可设 `hidden: true` 从启动列表隐藏
4. **`no*` 开关**：设为 `true` 会跳过对应类型的默认路径加载，但 `additional*Paths` 仍会生效
5. **资源冲突自动去重**：同名 prompt / theme / **skill** 会被去重（保留首个，冲突记录为 collision diagnostic）；但 **extensions 不去重**，同名 tool / flag 冲突只记录为 diagnostic，**全部保留**，优先级由加载顺序决定
