# Pi Package -- 扩展打包与分发

## 概述

**什么时候需要打包？** 当你的 extension / skill 想跨项目复用、分享给团队、或发布到社区 gallery 时，就需要打成 pi package。反之，本地单文件 extension 用 `pi -e ./ext.ts` 临时加载即可，不必打包。

Pi Package 是 pi-agent **扩展的分发格式**。它将 extension 代码、skills、prompt templates、themes 等资源打包为一个可分发的单元，通过 npm 或 git 共享给其他用户。

核心价值：
- **生态建设**：让开发者可以发布和分享自己的扩展
- **团队协作**：项目级 packages 在 `.pi/settings.json` 中声明，团队成员自动安装
- **版本管理**：npm 支持版本锁定，git 支持 tag/commit 锁定
- **资源组织**：在一个 package 中同时包含 extensions、skills、prompts、themes

参考文档：`packages/coding-agent/docs/packages.md`
源码位置：`packages/coding-agent/src/core/package-manager.ts`
包管理器接口 `PackageManager` 定义在 `package-manager.ts`。

---

## Package 来源类型

Pi Package 支持三种来源：

### npm 包

```bash
pi install npm:@scope/pkg@1.2.3
pi install npm:pkg
```

- 带版本号时会被锁定，`pi update` 不会自动升级
- 用户级安装到 `~/.pi/agent/npm/`
- 项目级安装到 `.pi/npm/`
- 可通过 `settings.json` 中的 `npmCommand` 字段（`string[]` 数组）指定自定义包管理器：
```json
{
  "npmCommand": ["mise", "exec", "node@20", "--", "npm"]
}
```

### git 仓库

```bash
pi install git:github.com/user/repo@v1          # git 前缀 + 短格式
pi install git:git@github.com:user/repo@v1        # SSH 短格式需要 git: 前缀
pi install https://github.com/user/repo@v1        # 完整 HTTPS URL
pi install ssh://git@github.com/user/repo@v1      # 完整 SSH URL
```

- ref 支持 tag 和 commit hash
- `pi update` 不会移动 ref 到更新版本，但会确保 clone 与配置 ref 一致
- clone 到 `~/.pi/agent/git/<host>/<path>`（全局）或 `.pi/git/<host>/<path>`（项目）

### 本地路径

```bash
pi install /absolute/path/to/package
pi install ./relative/path/to/package
```

- 指向磁盘上的文件/目录，不复制
- 相对路径根据 settings 文件所在目录解析
- 如果路径指向文件，以单 extension 方式加载
- 如果路径指向目录，按包规则加载资源

---

## Package 结构

### 清单方式（`package.json`）

```json
{
  "name": "my-package",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"],
    "video": "https://example.com/demo.mp4",
    "image": "https://example.com/screenshot.png"
  }
}
```

路径相对于包根目录，支持 glob 模式和 `!排除` 语法。`keywords` 中包含 `"pi-package"` 可在包画廊中被发现。

### 约定目录方式（自动发现）

如果没有 `pi` manifest，pi 会从约定目录自动发现：

| 目录 | 加载规则 |
|------|----------|
| `extensions/` | 加载 `.ts` 和 `.js` 文件 |
| `skills/` | 递归查找 `SKILL.md` 文件夹 + 顶层 `.md` 文件 |
| `prompts/` | 加载 `.md` 文件 |
| `themes/` | 加载 `.json` 文件 |

上表是 **package 内部**（或 `.pi/`、`~/.pi/agent/` 这些 pi 管理目录下）的约定。此外，**skills 还会从 `.agents/skills` 自动发现**（不限于 pi 管理目录）：

- **项目级**：从当前目录逐层向上查找 `.agents/skills`，直到 git 仓库根（无 git 则到文件系统根）。项目需受信任（trusted）才会加载。
- **用户级**：`~/.agents/skills`

> 源码：`collectAncestorAgentsSkillDirs` / `collectAutoSkillEntries(dir, "agents")` in `package-manager.ts`。注意只有 skills 有这层 `.agents/` 发现，extensions/prompts/themes 没有。

---

## 依赖管理

### 运行时依赖

第三方依赖放 `dependencies`。pi 安装 npm/git 包时会自动 `npm install`。

### Peer Dependencies（不要打包）

以下包是 pi 内置的**核心包**，扩展应该通过 `peerDependencies` 引用，**不要打包进自己的 tarball**：

- `@earendil-works/pi-ai`
- `@earendil-works/pi-agent-core`
- `@earendil-works/pi-coding-agent`
- `@earendil-works/pi-tui`
- `typebox`

### 打包其他 Pi Package

如果依赖其他 pi package，必须同时声明在 `dependencies` 和 `bundledDependencies` 中，并通过 `node_modules/` 路径引用其资源：

```json
{
  "dependencies": {
    "other-pi-pkg": "^1.0.0"
  },
  "bundledDependencies": ["other-pi-pkg"],
  "pi": {
    "extensions": [
      "extensions",
      "node_modules/other-pi-pkg/extensions"
    ],
    "skills": [
      "skills",
      "node_modules/other-pi-pkg/skills"
    ]
  }
}
```

---

## 过滤与选择

在 `settings.json` 中可以用对象形式精细控制 package 加载的内容：

```json
{
  "packages": [
    "npm:simple-pkg",
    {
      "source": "npm:my-package",
      "extensions": ["extensions/*.ts", "!extensions/legacy.ts"],
      "skills": [],
      "prompts": ["prompts/review.md"],
      "themes": ["+themes/legacy.json"]
    }
  ]
}
```

规则说明：
- **省略 key**：加载该类型所有资源
- **`[]`**：跳过该类型所有资源
- **`!pattern`**：排除匹配项
- **`+path`**：强制包含一个精确路径
- **`-path`**：强制排除一个精确路径
- 过滤器在 manifest 基础上叠加（只缩小范围）

---

## CLI 命令

```bash
# 安装
pi install npm:@foo/bar@1.0.0
pi install git:github.com/user/repo

# 试用（不写入 settings，仅当前运行）
pi -e npm:@foo/bar
pi -e git:github.com/user/repo

# 移除
pi remove npm:@foo/bar

# 列表
pi list

# 更新
pi update                    # 仅更新 pi 自身
pi update --all              # 更新 pi + packages + 同步 git ref
pi update --extensions       # 仅更新 packages + 同步 git ref
pi update --self             # 仅更新 pi 自身（同 pi update）
pi update --self --force     # 强制重新安装 pi（即使已是最新）
pi update npm:@foo/bar       # 更新单个 package
pi update --models           # 仅刷新模型目录（不更新 pi 本身或 packages）
pi update --extension <source> # 更新单个 extension package（单数形式）

# 资源管理（TUI 界面）
pi config                    # 打开 TUI 配置界面，启用/禁用 resources（extensions/skills/prompts/themes 四类）
                             # 支持 -l 写入项目级，Tab 切换全局/项目模式
```

**注意**：这些命令管理的是 pi packages，不是 pi CLI 本身的安装。安装路径默认写在用户设置 (`~/.pi/agent/settings.json`)，使用 `-l` 标志写入项目设置 (`.pi/settings.json`)。

---

## PackageManager 类型

> **用途提示**：`PackageManager` 主要供 pi 内部使用，外部集成推荐走 CLI（`pi install` / `pi config` 等）。下面类型签名供需要编程式控制的场景参考。

源码中 `PackageManager` 是定义在 `package-manager.ts` 中的接口类型，核心方法包括：

```ts
interface PackageManager {
  // 解析所有已配置包的资源路径
  resolve(onMissing?: (source: string) => Promise<MissingSourceAction>): Promise<ResolvedPaths>
  //   onMissing: 包未安装时的回调，返回 "install" 自动安装 / "skip" 跳过该包 / "error" 抛错

  // 安装、更新、移除包
  install(source: string, options?: { local?: boolean }): Promise<void>
  installAndPersist(source: string, options?: { local?: boolean }): Promise<void>
  remove(source: string, options?: { local?: boolean }): Promise<void>
  removeAndPersist(source: string, options?: { local?: boolean }): Promise<boolean>
  update(source?: string): Promise<void>
  //   local: 是否写项目级设置（true → .pi/settings.json，false/省略 → ~/.pi/agent/settings.json）

  // 列出已配置包
  listConfiguredPackages(): ConfiguredPackage[]

  // 解析指定扩展源（-e 试用走这里）
  resolveExtensionSources(sources: string[], options?: { local?: boolean; temporary?: boolean }): Promise<ResolvedPaths>
  //   temporary: true 时不写 settings、装到临时目录，仅当前运行生效

  // 添加/移除 source 到 settings（不触发安装）
  addSourceToSettings(source: string, options?: { local?: boolean }): boolean
  removeSourceFromSettings(source: string, options?: { local?: boolean }): boolean

  // 进度回调与路径查询
  setProgressCallback(callback: ProgressCallback | undefined): void
  getInstalledPath(source: string, scope: "user" | "project"): string | undefined
}

interface PathMetadata {
  source: string
  scope: SourceScope
  origin: "package" | "top-level"
  baseDir?: string
}

interface ResolvedResource {
  path: string
  enabled: boolean
  metadata: PathMetadata
}

interface ResolvedPaths {
  extensions: ResolvedResource[]
  skills: ResolvedResource[]
  prompts: ResolvedResource[]
  themes: ResolvedResource[]
}
```

**辅助类型**（定义在 `package-manager.ts` / `settings-manager.ts`）：

```ts
// package-manager.ts:77
type MissingSourceAction = "install" | "skip" | "error"

// package-manager.ts:79-86
interface ProgressEvent {
  type: "start" | "progress" | "complete" | "error"
  action: "install" | "remove" | "update" | "clone" | "pull"
  source: string
  message?: string
}
type ProgressCallback = (event: ProgressEvent) => void

// package-manager.ts:95-100
interface ConfiguredPackage {
  source: string
  scope: "user" | "project"
  filtered: boolean
  installedPath?: string
}

// package-manager.ts:126
type SourceScope = "user" | "project" | "temporary"

// settings-manager.ts:75-84
type PackageSource =
  | string
  | {
      source: string
      autoload?: boolean
      extensions?: string[]
      skills?: string[]
      prompts?: string[]
      themes?: string[]
    }
```

---

## 安装逻辑

`DefaultPackageManager`（`PackageManager` 的默认实现，统一处理 npm / git / local 三种来源）处理流程：

1. **身份去重**：如果同一 package 同时出现在全局和项目设置中，项目设置优先
   - npm：按包名去重
   - git：按仓库 URL 去重（不含 ref）
   - 本地：按解析后的绝对路径去重

2. **Git 包**：clone 或 fetch 已有 clone，checkout 到配置的 ref，有 `package.json` 则 `npm install`

3. **npm 包**：通过 `npm install` 安装到对应目录（实际调用形如 `npm install <spec> --prefix <installRoot> --legacy-peer-deps`）

4. **本地包**：直接引用路径

---

## 关键细节与陷阱

### 1. 安全注意

Pi packages 以完整系统权限运行。**Extension 可以执行任意代码**，Skills 可以引导模型执行任意操作。在安装第三方 package 之前务必审查源码。

### 2. 项目级 vs 用户级

`pi install` 默认写入用户设置 (`~/.pi/agent/settings.json`)。
使用 `-l` 写入项目设置 (`.pi/settings.json`)。
项目设置可被团队成员共享，pi 启动时会自动安装缺失的 packages。

### 3. Git ref 锁定

`git:host/repo@v1` 中的 `v1` 是锁定的。`pi update` 不会移动 ref，只会确保已 clone 的仓库与配置 ref 一致。要更新到新 ref，需要：
```bash
pi install git:host/user/repo@new-ref
```

### 3b. 项目级包条目的 delta 叠加语义

项目级 `packages` 条目（`.pi/settings.json`）中带 `autoload: false` 的包条目，会作为全局条目（`~/.pi/agent/settings.json`）的 **delta 叠加**而非替换。即：项目级可以禁用全局已启用的某个包的资源自动加载，**而不影响全局配置本身**（全局条目原样保留，项目只是在其上打补丁）。

最小对比示例：

```jsonc
// ~/.pi/agent/settings.json（全局）
{ "packages": ["npm:my-pkg"] }

// .pi/settings.json（项目：在该项目里关掉 my-pkg 的 skills 自动加载，但 extensions/prompts/themes 仍按全局加载）
{ "packages": [{ "source": "npm:my-pkg", "autoload": false, "skills": [] }] }
```

### 4. Git 更新行为

当 reconciliation（sync）改变 checkout 时，pi 会：
1. `git reset` + `git clean` 工作区
2. 如果存在 `package.json`，自动 `npm install`

### 5. 环境变量控制

- `PI_OFFLINE=1|true|yes`（不区分大小写）：禁用网络更新检查，所有操作使用已有本地缓存
- `GIT_TERMINAL_PROMPT=0`：禁用 git 凭据提示（CI 环境）
- `GIT_SSH_COMMAND`：自定义 SSH 命令（如设置超时：`ssh -o BatchMode=yes -o ConnectTimeout=5`）

### 6. 包解析时模块隔离

不同的 package 有独立的模块根（module root）。这意味着不同 package 的 npm 依赖不会冲突，也不会共享实例。

### 7. Gallery 元数据

在 `package.json` 的 `pi` 字段中添加 `video` (MP4) 或 `image` (PNG/JPEG/GIF/WebP) 可在包画廊展示预览。如果同时设置两者，video 优先。注意：`video` / `image` 是画廊端元数据，不在运行时 `PiManifest` 接口中（`PiManifest` 只有 `extensions` / `skills` / `prompts` / `themes` 四个字段）。
