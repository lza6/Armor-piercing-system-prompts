# 场景：打包发布 Pi Package (I01)

## 这是什么

**Pi Package** 是 pi-agent 扩展的**标准分发单元**。把零散的 extensions、skills、prompt templates、themes 打包到一个 npm 包或 git 仓库里，让别人通过 `pi install` 一键加载。

**不是什么**：
- 不是「二进制压缩包」——它就是普通的 npm 包或 git 仓库，pi 直接读源码
- 不是「subagent 任务包」——subagent 是运行时把任务委托给子 Agent（见 [I05](I05-subagent.md)），和打包分发无关
- 不是「SDK 内联加载」——SDK 通过 `additionalExtensionPaths` 参数加载（见 [A06](A06-load-extensions.md)），不走 `pi install` 命令

## 什么时候用 / 不用会怎样

| 触发场景 | 用 Pi Package | 不用的话 |
|---------|--------------|---------|
| 跨项目复用自己的扩展 | `pi install` 一行加载 | 每个项目手动 cp 目录 |
| 团队共享扩展 | 写入 `.pi/settings.json` 提交 git | 每个成员手动安装、版本漂移 |
| 公开发布到社区 | 发 npm + tag `pi-package` 进画廊 | 用户找不到你 |
| 单项目一次性扩展 | 直接用 `.pi/extensions/` 目录 | 打成 package 是过度工程 |

**铁律**：如果你**只有一两个扩展且不打算分发**，直接放 `.pi/extensions/` 即可——pi 启动时自动发现（约定目录方式，见下文）。打成 package 是为了**复用、共享、版本管理**。

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `package.json` 中的 `pi` 字段 | 声明包内的资源路径 | [sdk_doc/20-pi-package.md](../sdk_doc/20-pi-package.md) |
| `pi install` / `pi remove` / `pi update` / `pi list` | CLI 管理已安装的包 | 同上 |
| `pi config` | 启用/禁用已安装的资源（交互式选择） | 同上 |
| `pi -e <source>`（即 `--extension`） | 临时加载到当前运行（不写入 settings，**但会下载到磁盘**） | 同上 |
| `settings.json` 的 `packages` 字段 | 声明项目 / 用户级依赖 | 同上 |

> 本文档覆盖**完整决策与命令矩阵**。类型签名（PackageManager 接口、ResolvedPaths 等）详见 sdk_doc/20。

---

## 实现思路

1. **写 `package.json`**：声明 `pi` 字段列出 extensions/skills/prompts/themes 的路径
2. **声明 peerDependencies**：5 个 pi 核心包必须列在 `peerDependencies` 里，**不能打包进 tarball**
3. **声明 bundledDependencies**：如果依赖其他 pi package，必须同时放 `dependencies` + `bundledDependencies`，并通过 `node_modules/` 路径引用其资源
4. **选分发渠道**：npm（带版本锁定）、git（带 ref 锁定）、本地路径（不复制）
5. **项目中加载**：在 `.pi/settings.json`（项目级，需项目 trust）或 `~/.pi/agent/settings.json`（用户级）的 `packages` 数组里声明

---

## 核心：`package.json` 的 `pi` 字段

### 完整示例

```json
{
  "name": "my-pi-extension",
  "version": "1.0.0",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions/*.ts"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"],
    "image": "https://example.com/screenshot.png",
    "video": "https://example.com/demo.mp4"
  },
  "peerDependencies": {
    "@earendil-works/pi-ai": "*",
    "@earendil-works/pi-agent-core": "*",
    "@earendil-works/pi-coding-agent": "*",
    "@earendil-works/pi-tui": "*",
    "typebox": "*"
  },
  "dependencies": {
    "axios": "^1.0.0"
  }
}
```

### 字段说明

| 字段 | 类型 | 含义 |
|------|------|------|
| `name` | string | npm 包名（npm 分发时必填，本地/git 可省略） |
| `version` | string | 语义化版本（npm 分发必填） |
| `keywords` | string[] | **必须包含 `"pi-package"`** 才能进 pi.dev 包画廊 |
| `pi.extensions` | string[] | 扩展文件路径，相对包根，支持 glob 和 `!排除` |
| `pi.skills` | string[] | skill 目录（递归找 `SKILL.md`） |
| `pi.prompts` | string[] | prompt 模板 `.md` 文件 |
| `pi.themes` | string[] | 主题 `.json` 文件 |
| `pi.image` | string | **画廊端元数据**，画廊预览图（PNG / JPEG / GIF / WebP）。运行时 `PiManifest` 接口不读取此字段 |
| `pi.video` | string | **画廊端元数据**，画廊预览视频（仅 MP4，桌面端 hover 自动播放；与 image 同时设时 video 优先）。运行时 `PiManifest` 接口不读取此字段 |

**路径相对包根目录解析**。glob 支持 `*`/`?`/`!`/`+`/`-`（filter 语义见下文「PackageSource 对象形式」）。

### 约定目录方式（自动发现）

如果**没有 `pi` manifest**，pi 启动时从包根的约定目录自动发现资源：

| 目录 | 加载规则 |
|------|---------|
| `extensions/` | 加载所有 `.ts` 和 `.js` 文件；如果子目录含 `package.json`（带 `pi.extensions`）或 `index.ts`/`index.js`，该子目录被识别为单独的扩展入口 |
| `skills/` | 递归查找含 `SKILL.md` 的文件夹 + 顶层 `.md` 文件 |
| `prompts/` | 加载所有 `.md` 文件 |
| `themes/` | 加载所有 `.json` 文件 |

约定方式是**渐进式起步**的捷径——扩展多了再升级到 `pi` manifest。

### peerDependencies：5 个核心包不要打包

以下包是 pi 内置的，扩展应通过 `peerDependencies` 引用（`"*"` 范围），**不要打包进自己的 tarball**：

- `@earendil-works/pi-ai`
- `@earendil-works/pi-agent-core`
- `@earendil-works/pi-coding-agent`
- `@earendil-works/pi-tui`
- `typebox`

证据：`docs/packages.md`。打成 tarball 会导致模块重复加载、版本冲突。

### bundledDependencies：打包其他 pi package

依赖其他 pi package 时，**必须同时声明在 `dependencies` 和 `bundledDependencies`**，并通过 `node_modules/` 路径引用其资源：

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

原因：pi 用独立的 module root 加载每个 package，bundledDependencies 保证依赖包**和你的包一起进 tarball**，不会因为 npm 解析失败而缺失资源。

---

## 三种 source 类型对比

`pi install` 和 `settings.json.packages` 都接受三种 source：

| 类型 | 格式 | 身份去重 | 版本锁定 | 安装位置（user scope） |
|------|------|---------|---------|----------------------|
| **npm** | `npm:@scope/pkg@1.2.3` 或 `npm:pkg` | 包名 | 精确版本号被 pin，range 不 pin | `~/.pi/agent/npm/node_modules/<pkg>/` |
| **git** | `git:host/path@ref` / `git:git@host:path@ref` / `https://...` / `ssh://...` | host/path（**忽略 ref**） | ref 是 pin（branch/tag/commit） | `~/.pi/agent/git/<host>/<path>/` |
| **local** | `/abs/path` / `./rel/path` | 解析后的绝对路径 | 无（直接引用，不复制） | 原路径 |

**项目级安装到 `.pi/npm/` / `.pi/git/`**。

### npm source

```bash
pi install npm:@foo/bar@1.2.3   # 精确版本，pi update 跳过
pi install npm:@foo/bar          # 不 pin，pi update 会查最新
pi install npm:bar               # 无 scope 也可以
```

`npm install <spec> --prefix <installRoot> --legacy-peer-deps` 直接安装到目标目录（npm 内部处理下载和解压，pi 不显式调用 `npm pack`）。不同包管理器有适配：bun 用 `--omit=peer`，pnpm 用 `--config.auto-install-peers=false`，npm 用 `--legacy-peer-deps`。

### git source（**两种格式分清楚**）

```bash
# 短格式：必须有 git: 前缀
pi install git:github.com/user/repo@v1           # HTTPS 短
pi install git:git@github.com:user/repo@v1       # SSH 短（scp-like）

# 完整 URL：不需要 git: 前缀（必须是协议 URL）
pi install https://github.com/user/repo@v1
pi install ssh://git@github.com/user/repo@v1
pi install git://github.com/user/repo@v1
```

证据：`utils/git.ts`——没有 `git:` 前缀时**只接受协议 URL**（`https?://`、`ssh://`、`git://`）；有 `git:` 前缀时接受所有简写（包括 `github.com/user/repo`、`git@host:path`）。

**SSH 短格式 `git@github.com:user/repo` 必须配 `git:` 前缀**——不写前缀会被当成 local 路径。

### local source

```bash
pi install /absolute/path/to/package
pi install ./relative/path/to/package
```

- 相对路径根据 settings 文件所在目录解析（`~/.pi/agent/` 或项目 `.pi/`）
- 指向文件：按单扩展加载
- 指向目录：按包规则加载（先看 `package.json` 的 `pi` 字段，否则用约定目录）

### Git ref 锁定行为（关键陷阱）

`git:host/repo@v1` 中的 `v1` 是**锁定 ref**：

```bash
# 已 pin 到 v1.0.0
pi install git:github.com/me/repo@v1.0.0

# 这不会移动 ref，只确保 checkout 与配置一致
pi update --extensions

# 要切到新 ref，必须重新 install（会改 settings）
pi install git:github.com/me/repo@v2.0.0
```

更新 checkout 时 pi 会 `git reset --hard` + `git clean -fdx`，然后如有 `package.json` 则 `npm install`。**本地修改会被清掉**（`-fdx` 连被 gitignore 的文件一起删）。

**无 ref 的 git source**：如果安装时没指定 ref（如 `pi install git:github.com/user/repo`），`pi update --extensions` 时 pi 会 fetch upstream 默认分支并 reset 到最新 HEAD（`getLocalGitUpdateTarget` 处理无 ref 情况）。这与 pinned git（带 ref）行为不同——pinned git 只 reset 到配置的 ref，不跟随 upstream。

---

## CLI 命令矩阵

```bash
# 安装
pi install npm:@foo/bar@1.0.0
pi install git:github.com/user/repo
pi install ./local/path
pi install -l npm:@foo/bar           # 写到项目 .pi/settings.json（需项目 trust）

# 临时加载（-e 是 --extension 短形式）
pi -e npm:@foo/bar                   # 加载到当前运行，不写 settings
pi -e git:github.com/user/repo       # 同上
# ⚠️ -e 仍然会下载到临时目录 ~/.pi/agent/tmp/extensions/（temporary scope），只是不写 settings

# 移除
pi remove npm:@foo/bar               # 别名：pi uninstall
pi remove -l npm:@foo/bar            # 从项目设置移除

# 列表
pi list                              # 显示 user + project 已配置的包

# 启用/禁用资源（交互式 TUI）
pi config

# 更新（5 种粒度）
pi update                            # 默认 = --self，只更新 pi 本身
pi update --self                     # 同上
pi update --self --force             # 强制重装 pi（即使最新）
pi update --extensions               # 更新所有 package（pinned npm 跳过，pinned git 协调到配置 ref）
pi update --all                      # 更新 pi + packages
pi update npm:@foo/bar               # 更新单个 package（pinned npm 跳过）
pi update --extension npm:@foo/bar   # 同上
pi update --models                   # 仅刷新 model catalog（不涉及 package）

# 禁用扩展加载
pi -ne                               # 即 --no-extensions，禁用所有扩展发现（-e 显式指定的仍生效）
```

**`-l` 需要 project trust**：未 trust 的项目写 `.pi/settings.json` 会报错 `Project is not trusted. Use --approve to modify local package config.`。可用 `--approve` 一次性覆盖，或预先 `pi config` trust。

**`pi update <pinned pkg>` 不会切版本**：pinned npm（精确版本号）会被跳过；pinned git（带 ref）**不会被跳过**——pi 会 fetch 配置的 ref 并 `git reset --hard` 到该 ref（确保 checkout 与配置一致），但**不会移动到更新的 ref**。换版本 / 换 ref 必须重 `pi install`。证据：`updateConfiguredSources`（package-manager.ts，npm 仅 `!pinned` 入列，git 无条件入列）。

### CLI 的环境变量

| 变量 | 作用 |
|------|------|
| `PI_OFFLINE=1` | 禁用网络检查，全部用本地缓存 |
| `GIT_TERMINAL_PROMPT=0` | 禁用 git 凭据提示（CI 必备） |
| `GIT_SSH_COMMAND` | 自定义 SSH 命令（如 `ssh -o BatchMode=yes -o ConnectTimeout=5`）。注意：这是 **git 原生环境变量**，pi 不显式设置——但 git 继承自进程环境，所以在 shell 中 `export` 即生效 |

`npmCommand` 字段（写在 `settings.json` 里）可指定 npm 包装器：

```json
{
  "npmCommand": ["mise", "exec", "node@20", "--", "npm"]
}
```

---

## settings.json 的 `packages` 字段

```jsonc
// ~/.pi/agent/settings.json 或 .pi/settings.json
{
  "packages": [
    // 字符串形式：加载该 package 全部资源
    "npm:my-pi-extension",

    // 对象形式：精细控制加载哪些资源
    {
      "source": "npm:another-pkg",
      "extensions": ["extensions/*.ts", "!extensions/legacy.ts"],
      "skills": [],
      "prompts": ["prompts/review.md"],
      "themes": ["+themes/legacy.json"]
    }
  ]
}
```

### PackageSource 对象形式字段表

| 字段 | 类型 | 含义 |
|------|------|------|
| `source` | string | **必填**。npm/git/local source 字符串 |
| `extensions` | string[] \| undefined | 扩展过滤器（省略 = 加载全部） |
| `skills` | string[] \| undefined | skill 过滤器（省略 = 加载全部） |
| `prompts` | string[] \| undefined | prompt 过滤器（省略 = 加载全部） |
| `themes` | string[] \| undefined | theme 过滤器（省略 = 加载全部） |

### Filter pattern 语义

| 写法 | 含义 |
|------|------|
| 省略 key | **加载该类型所有资源**（manifest 允许的全部） |
| `[]` | **跳过该类型所有资源**（注意与「省略」相反！） |
| `pattern`（含 `*`/`?`） | glob 匹配 |
| `!pattern` | 排除匹配项 |
| `+path` | 强制包含精确路径（绕过 manifest 的排除） |
| `-path` | 强制排除精确路径 |

**过滤器在 manifest 基础上叠加，只缩小范围**（不能放大到 manifest 没声明的资源）。

### Scope 去重规则

同一个包可能同时出现在 user 和 project settings 中：

| source 类型 | 身份 key | 冲突时谁赢 |
|------------|---------|-----------|
| npm | `npm:<name>`（不含版本） | **project 胜** |
| git | `git:<host>/<path>`（**忽略 ref**） | **project 胜** |
| local | 解析后的绝对路径 | **project 胜** |

证据：`getPackageIdentity`（package-manager.ts，身份去重）+ `dedupePackages`（package-manager.ts，project 覆盖 user）。

### `autoload: false`：增量而非覆盖

project 条目默认完全覆盖同身份的 user 条目。但如果 project 条目写成 `{ source, autoload: false }`，它就不再是「覆盖」而是「增量」——两者都保留，project 条目作为 user 条目之上的过滤器叠加。源码：`dedupePackages`（package-manager.ts，`autoload === false` 时保留两份）+ `findAutoloadDeltaBase`（package-manager.ts）。

```jsonc
// ~/.pi/agent/settings.json（全局：加载全部资源）
{ "packages": ["npm:my-pi-extension"] }

// .pi/settings.json（项目级：只加载部分资源，不全量覆盖）
{
  "packages": [
    { "source": "npm:my-pi-extension", "autoload": false, "extensions": ["extensions/*.ts"] }
  ]
}
// → 最终加载 user 条目的全部 + project 条目过滤后的 extensions（增量叠加）
```

这适合团队共享场景：全局条目加载完整扩展，项目条目用 `autoload: false` + filter 只启用部分资源。

---

## 安全警告（**必读**）

> Pi packages 以**完整系统权限**运行。Extension 可以执行任意代码，skill 可以引导模型执行任意操作（包括运行可执行文件）。
>
> **安装第三方 package 前必须审查源码**。`pi install npm:@stranger/awesome-ext` 等价于 `npm install` + 让陌生人代码接管你的 Agent。

证据：`docs/packages.md`。

---

## I01 vs A06：CLI 路径 vs SDK 路径

| 维度 | I01（本文档） | [A06](A06-load-extensions.md) |
|------|--------------|------------------------------|
| 入口 | `pi install` CLI 命令 | SDK API `additionalExtensionPaths` 参数 |
| 写哪里 | `~/.pi/agent/settings.json` 或 `.pi/settings.json` | 不写 settings，运行时传参 |
| 何时用 | 长期复用 / 团队共享 / 公开发布 | 一次性试用 / 程序化动态加载 / 测试 |
| 临时模式 | `pi -e` 短形式（仍下载到磁盘，只是不写配置） | 直接 API 调用 |
| 共同点 | 都走 `resolveExtensionSources` + 支持 npm/git/local 三种 source | 同上 |

**简单选择**：终端用户场景 → I01；SDK 集成场景 → A06。

---

## 变体与延伸

- **分发扩展到团队 / 社区** → [I02](I02-distribute-extension.md)
- **扩展引用第三方 npm 依赖**（axios、lodash 等）→ [I03](I03-extension-deps.md)
- **在沙箱里跑不可信扩展** → [I04](I04-sandbox.md)
- **SDK 内联加载**（不走 CLI）→ [A06](A06-load-extensions.md)
- **Subagent 进程级委托**（与打包分发无关，但容易混淆）→ [I05](I05-subagent.md)

---

## 常见误期待与陷阱

1. **「`pi update <pkg>` 能升级 pinned 包」**——部分错。pinned npm（精确版本号）会被跳过；pinned git（带 ref）**不会被跳过**——pi 会 fetch + reset 到配置的 ref（协调 checkout），但不会移动到更新的 ref。要真正换版本/换 ref 必须重 `pi install`。
2. **「`pi -e <source>` 不下载」**——错。`-e`（`--extension`）只是不写 settings，**仍然会 `npm install` / `git clone` 到磁盘**——下载到临时目录 `~/.pi/agent/tmp/extensions/`（temporary scope）。要真正不下载只能用 local 路径。
3. **「`skills: []` 等于加载全部 skill」**——错。`[]` 是「跳过全部」，省略 key 才是「加载全部」。
4. **「`git@github.com:user/repo` 不写 `git:` 前缀也行」**——错。无前缀只接受协议 URL（`https://`/`ssh://`/`git://`）。scp-like 短格式必须 `git:` 前缀。
5. **「pi 核心包应该 `dependencies`」**——错。`pi-ai`/`pi-agent-core`/`pi-coding-agent`/`pi-tui`/`typebox` 必须 `peerDependencies` + `"*"`，否则 tarball 重复打包、模块实例分裂。
6. **「`pi install -l` 默认能写项目配置」**——错。需要项目已 trust，否则报错。用 `--approve` 覆盖或先 `pi config` trust。
7. **「同时装在 user 和 project 会出现两份」**——错。project 胜出，user 那条被去重忽略（按身份 key，不按 ref）。
8. **「git ref 会自动跟随 upstream 更新」**——错。ref 是 pin，`pi update --extensions` 只 `git reset` 到**配置的 ref**，不会移动到新 tag。要切 ref 必须重 install。
9. **「`keywords: ["pi-package"]` 是 pi 校验的」**——半错。pi 不强制校验，但 pi.dev 包画廊用这个 keyword 发现包——不发 npm 只本地用可以不写。
