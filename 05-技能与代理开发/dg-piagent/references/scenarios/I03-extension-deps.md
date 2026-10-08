# 场景：在扩展中引用第三方依赖 (I03)

## 这是什么

**扩展依赖管理** = 在扩展代码里 `import` 第三方 npm 包（axios / lodash / diff）或其他 pi package 时，正确声明 `dependencies` / `peerDependencies` / `bundledDependencies`，让 pi 在消费者侧能正确安装、隔离、加载这些依赖。

**不是什么**：
- 不是「**打包 / 分发 pi package**」——打包 / 分发是 [I01](I01-pi-package.md) / [I02](I02-distribute-extension.md) 的事，I03 只管**依赖声明**这一维度
- 不是「**SDK 内联加载**」——SDK 通过 `additionalExtensionPaths` 程序化加载扩展（npm/git 源走**临时 install** 到 `~/.pi/agent/tmp/extensions/`，本地磁盘路径才不 install），机制详见 [A06](A06-load-extensions.md)
- 不是「**subagent 委托**」——subagent 是运行时任务委托（[I05](I05-subagent.md)），与依赖管理无关

---

## 什么时候要管 / 不管会怎样

| 触发场景 | 按 I03 做 | 不做的话 |
|---------|----------|---------|
| 扩展 `import` 了 pi 内核包（`pi-ai` / `pi-agent-core` 等）| 写完整 `peerDependencies` 5 包 + `"*"` 范围 | tarball 重复打包 pi 内核 → 模块实例分裂 → 运行时不一致 |
| 扩展 `import` 了第三方 npm 包（axios / lodash）| 写进 `dependencies` + 正确版本范围 | 消费者 install 时缺包 → 加载报错 `Cannot find module 'axios'` |
| 扩展依赖**另一个 pi package**（含 extensions / skills 等资源）| `dependencies` + `bundledDependencies` 同时声明 + 用 `node_modules/<pkg>/` 引用资源 | tarball 里没有那个 pi package → 消费者加载时资源缺失 |
| 本地路径扩展（`./my-ext.ts`）| **不用管依赖**——pi 不 install，node_modules 由你自己维护 | 按 npm 包那套写也没用，pi 不读 |

**铁律**：依赖管理的核心是「**让 pi 知道这个包该装哪里、该不该打包、该和谁共享**」。三种依赖类型（第三方 npm / pi 内核 / 其他 pi package）走完全不同的路径，**不能混着写**。

---

## I03 vs I01 vs I02 边界

| 场景 | 定位 | 关心的 package.json 字段 |
|------|------|------------------------|
| [I01](I01-pi-package.md) | **打包** | `pi.extensions` / `pi.skills` / `pi.prompts` / `pi.themes` / `keywords` / `files` |
| [I02](I02-distribute-extension.md) | **分发** | `files` / `name` / `version` / npm registry 配置 / `.pi/settings.json` |
| **I03（本文档）** | **依赖声明** | **`dependencies`** / **`peerDependencies`** / **`bundledDependencies`** |

**一句话**：I01 决定**包里有什么**、I02 决定**怎么送出去**、I03 决定**运行时依赖怎么解析**。

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `package.json` `dependencies` | 声明第三方 npm 依赖（pi install 时被 npm 解析） | [sdk_doc/20-pi-package.md](../sdk_doc/20-pi-package.md) |
| `package.json` `peerDependencies` | 声明 pi 内核包（**不打包，由 pi 提供运行时实例**） | 同上 |
| `package.json` `bundledDependencies` | 物理打包其他 pi package 进 tarball | 同上 |
| jiti `alias` / `virtualModules`（pi 内部机制）| 把 `import "@earendil-works/pi-ai"` 重定向到 pi 自身实例 | 见下文「核心机制」|

> I03 不直接调用 SDK API，只通过 `package.json` 字段控制 pi 的安装 / 加载行为。

---

## 核心机制：为什么依赖声明这么重要

> 这节是 I03 的灵魂——不读懂这节，所有字段规则都只是死记硬背。**先讲清楚 pi 怎么加载扩展，再讲字段规则就顺理成章。**

### 机制一：扩展代码被 jiti 即时编译

扩展文件是 `.ts` 或 `.js`，pi 不预编译——启动时用 [jiti](https://github.com/unjs/jiti) 即时编译 `.ts` → `.js` 再 import。

**对依赖的影响**：扩展 `import` 的所有包**必须是真实的 npm 包**（有 `package.json` + 入口文件），不能是 TypeScript path mapping 或 monorepo workspace 链接（除非走 local source）。

证据：`loader.ts` —— `createJiti(import.meta.url, { moduleCache: false, ... })` + `jiti.import(extensionPath, { default: true })`。

### 机制二：pi 内核包被「重定向到 pi 自身」

扩展代码里的 `import { ExtensionAPI } from "@earendil-works/pi-coding-agent"` **不会真的去 node_modules 里找这个包**——pi 用 jiti 把这些 import **重定向到 pi 自身的运行时实例**。重定向方式按运行模式分**三分支**（loader.ts）：

- **Bun binary 模式**：`virtualModules` + `tryNative: false`
- **Node 跑 .ts 源码**（如 `tsx`/`node` 直接跑源码，开发调试时）：`virtualModules` + `tsconfigPaths: true`
- **built Node 跑 dist**（生产安装的编译产物）：`alias: getAliases()`

也就是说，`virtualModules` 在前两种模式都用，`alias` 仅 built Node 用——**自己用 tsx 调试扩展时走的是 `virtualModules`，不是 `alias`**。

被重定向的包清单（**是 peerDependencies 5 包的超集**——5 个主包之外还含子路径和老名别名）：

```ts
// loader.ts 的核心逻辑（简化展示）
const VIRTUAL_MODULES = {
  // pi 内核包 → 重定向到 pi 自身实例
  "@earendil-works/pi-agent-core": bundledPiAgentCore,
  "@earendil-works/pi-ai": bundledPiAiCompat,
  "@earendil-works/pi-ai/compat": bundledPiAiCompat,          // 子路径
  "@earendil-works/pi-ai/oauth": bundledPiAiOauth,            // 子路径
  "@earendil-works/pi-ai/providers/all": bundledPiAiProviders,// 子路径（扩展常用）
  "@earendil-works/pi-coding-agent": bundledPiCodingAgent,
  "@earendil-works/pi-tui": bundledPiTui,
  "typebox": bundledTypebox,
  "typebox/compile": bundledTypeboxCompile,                   // 子路径
  "typebox/value": bundledTypeboxValue,                       // 子路径
  "@sinclair/typebox": bundledTypebox,                        // 双名兼容
  // 兼容老名 @mariozechner/*
  "@mariozechner/pi-agent-core": bundledPiAgentCore,
  "@mariozechner/pi-ai": bundledPiAiCompat,
  // ...（老名同样支持子路径，共约 24 项）
};
```

> **关键认知**：重定向表远不止 5 项。除 5 个 peerDeps 主包外，还覆盖：
> - **子路径**（`/compat`、`/oauth`、`/providers/all`、`typebox/compile`、`typebox/value`）——扩展 `import "@earendil-works/pi-ai/providers/all"` 同样被重定向，不需要装；
> - **`@sinclair/typebox` 双名**——`typebox` 和 `@sinclair/typebox`（及其子路径）都指向同一份 bundled 实例；
> - **`@mariozechner/*` 老名别名**——pi 迁移到 `@earendil-works/*` scope 前的老扩展无需改 import。
>
> peerDependencies 只声明 5 个主包是因为它们是这些子路径/别名的「入口声明」——npm 把主包当作 peer 不下载，子路径自然也不会单独下载。

**为什么这样设计**：所有扩展共享**同一个 pi 运行时实例**——扩展注册的 tool / handler / eventBus / provider 全部走同一个 registry，不会出现「扩展 A 的 pi 和扩展 B 的 pi 是两个对象」的割裂状态。

**对依赖声明的影响（关键）**：
- `peerDependencies` 列这 5 个包 = **告诉 npm 别下载**，由 pi 在运行时提供
- `dependencies` 误写这 5 个包 = npm 真下载到 installRoot 的 `node_modules/`，但 pi 加载时**仍然走重定向**（alias / virtualModules）——下载的包**被完全忽略**（`node_modules/<pkg>/` 占空间但永不 `require`）。注意：单纯写 `dependencies` **不会**进你的 tarball（npm 打包时 `dependencies` 不内联进包），只是消费者 install 时多下一份被忽略的副本；**只有同时把这些包列进 `bundledDependencies`** 时才会物理打进 tarball，造成真正的体积膨胀 + 模块重复实例

证据：loader.ts（`VIRTUAL_MODULES` 定义、`getAliases` alias 表、三分支切换）。

### 机制三：installRoot 共享 node_modules

pi 把所有 npm 来源的扩展装在**同一个目录**下：

| scope | installRoot |
|-------|-------------|
| user | `~/.pi/agent/npm/` |
| project | `<cwd>/.pi/npm/` |
| temporary（`pi -e`）| `~/.pi/agent/tmp/extensions/<prefix>/<hash>[/<suffix>]`，按 hash 复用（持久缓存，源码未见运行结束清理）。prefix：npm 源为 `npm`，git 源为 `git-<host>`（如 `git-github.com`）；git 源还带 `<suffix>` = 仓库内 path |

每个 installRoot 下有一个**共享的 `package.json`**（`{ "name": "pi-extensions", "private": true }`）和**共享的 `node_modules/`**——所有 user-scope 的 pi package 的第三方 dependencies（axios / lodash 等）都装在这同一个 node_modules 里。

```bash
~/.pi/agent/npm/
├── package.json          # {"name":"pi-extensions","private":true}
├── node_modules/
│   ├── axios/            # pi-package-a 的 dependency
│   ├── lodash/           # pi-package-b 的 dependency
│   └── @myteam/
│       └── pi-code-reviewer/   # ← 这是一个 pi package
└── .gitignore            # pi 自动维护
```

**关键含义**：
- ✅ **多个 pi package 共享同一个 axios 实例**（npm hoisting 后的唯一版本）
- ⚠️ **两个 pi package 依赖同一个包的不同大版本时会冲突**——npm hoisting 选一个版本，另一个版本被嵌套到子 `node_modules/`（npm 标准行为）。如果嵌套发生在 pi package 自己的目录下（`node_modules/<pi-pkg>/node_modules/<dep>/`），jiti 解析时**可能找不到嵌套版本**——jiti 默认从扩展文件所在目录向上找 node_modules
- ❌ **不是「不同 package 独立模块根」**——官方 docs `packages.md` 原话 "Pi loads packages with separate module roots" 指的是 **bundledDependencies** 物理打包进 tarball 时的隔离（见下文），不是 installRoot 共享的 dependencies

证据：package-manager.ts（`ensureNpmProject` 写共享 package.json、`getNpmInstallRoot`、`installNpmBatch` 多 spec 一次 install 到同一 root、`getManagedNpmInstallPath` = `<installRoot>/node_modules/<pkg-name>/`）。

### 机制四：bundledDependencies 物理打包

如果扩展 A 依赖另一个 pi package B（B 自己也是 npm 发布的 pi package），你需要让消费者 `pi install A` 时**自动拿到 B 的资源**（extensions/skills/prompts/themes）。两种做法：

| 做法 | 结果 |
|------|------|
| 只写 `dependencies: { "B": "^1.0.0" }` | npm install 时 B **被装到 installRoot 共享 node_modules**。能否被 A 的 `pi.extensions`（`node_modules/B/extensions`）加载**取决于 npm hoisting**：只有 B 被嵌套进 A 自己的目录（`node_modules/A/node_modules/B/`）时该相对路径才解析得到；若 B 被 hoist 到共享根（`node_modules/B/`），则 `node_modules/B/extensions`（相对 A 包根）解析失败——**但这种写法 B 无论如何都不会被 publish 进 A 的 tarball** |
| 同时写 `dependencies: { "B": "^1.0.0" }` + `bundledDependencies: ["B"]` | **npm pack 时把 B 物理打进 A 的 tarball**（`A-1.0.0.tgz` 内含 `node_modules/B/`）。消费者 install 时 tarball 直接展开，B 的资源在 A 的 `node_modules/B/` 下 |

**为什么必须同时声明两处**：
- `dependencies` 告诉 npm 版本范围（`^1.0.0` 是什么意思）
- `bundledDependencies` 告诉 npm pack 时把这个包**打进 tarball**（而不是只记版本号让消费者自己装）

**bundledDependencies 的物理隔离**：tarball 内的 `node_modules/B/` 是 B 完整的副本——B 自己的 dependencies 也被打包进去（除非 B 的 dependencies 又是 pi 内核包，那 B 的 package.json 应该也写 peerDependencies）。这就是官方 docs `packages.md` 说的「separate module roots」——**bundledDependencies 才有真正的模块隔离**。

证据：`packages.md`（原文示例 + 「Pi loads packages with separate module roots, so separate installs do not collide or share modules」——紧接 bundledDependencies 段落）。

---

## 三种依赖类型对比

| 类型 | 写哪里 | 安装路径 | 运行时实例来源 |
|------|--------|---------|---------------|
| **第三方 npm 包**（axios / lodash）| `dependencies` | installRoot 共享 `node_modules/` | 标准 node 模块解析（从扩展文件目录向上找）|
| **pi 内核包**（pi-ai / pi-agent-core 等 5 个）| `peerDependencies` + `"*"` | **不下载**（`--legacy-peer-deps` 禁用 peer 自动安装）| jiti alias / virtualModules 重定向到 pi 自身 |
| **其他 pi package**（含 extensions/skills 资源）| `dependencies` + `bundledDependencies` | tarball 内 `node_modules/<pkg>/`（物理打包）| 标准 node 解析，从 A 的 `node_modules/B/` 找 |

> **不要混着写**：第三方包不能写 peerDependencies（pi 不会重定向它），pi 内核包不能写 dependencies（下载了也用不上，浪费空间），其他 pi package 不能只写 dependencies 不写 bundledDependencies（tarball 没那个包）。

---

## pi install 的实际命令（关键真相）

**pi 不只是跑 `npm install`**——不同包管理器和不同 source 类型，实际命令完全不同：

### npm 包的安装命令

| 包管理器 | 实际命令 | 关键 flag |
|---------|---------|----------|
| **npm**（默认）| `npm install <spec> --prefix <installRoot> --legacy-peer-deps` | **禁用 peer 自动安装** |
| **pnpm**（`npmCommand` 配置后）| `pnpm install <spec> --prefix <installRoot> --config.auto-install-peers=false --config.strict-peer-dependencies=false --config.strict-dep-builds=false` | 禁用 peer 自动安装 + 关严格检查 |
| **bun**（`npmCommand` 配置后）| `bun install <spec> --cwd <installRoot> --omit=peer` | 跳过 peer dependencies |

**为什么统一禁用 peer 自动安装**：pi 自己用 alias / virtualModules 提供 pi 内核包（机制二），如果包管理器又把 `@earendil-works/pi-*` 自动下载一份，会导致版本冲突 / 模块重复。源码注释原话：「Disable peer dependency resolution for managed installs so package managers do not install or solve host-provided @earendil-works/pi-* peers. Stale auto-installed pi peers can otherwise block updates.」

证据：`package-manager.ts`（`getNpmInstallArgs` 三段分支）。

### git 包的安装命令

git 包**不走 `getNpmInstallArgs`**——pi 直接 `git clone` 后在 clone 目录内跑：

| 条件 | 实际命令 | 关键差异 |
|------|---------|---------|
| 用户**未**配置 `npmCommand` | `npm install --omit=dev` | **跳过 devDependencies** + **不带 `--legacy-peer-deps`** |
| 用户**已**配置 `npmCommand` | `<configured-cmd> install` | **完整依赖**（含 dev）+ **不带 `--legacy-peer-deps`** |

**关键差异**：git 包安装**不带 `--legacy-peer-deps`**——如果 git 仓库的 `package.json` 写了 peerDependencies，包管理器可能会尝试自动下载 peer。所以 **git 分发的 pi package 更要把 peerDependencies 写对**，否则消费者 install 时 npm 会去 registry 拉一份 pi 内核包污染 node_modules。

证据：package-manager.ts（`getGitDependencyInstallArgs`、`installGit` 流程：`git clone` → 可选 `git checkout` → 有 package.json 跑 `npm install`）。

### local 包的安装命令

**不跑 npm install**。pi 直接读磁盘路径，扩展的 node_modules 由用户自己维护。

证据：`package-manager.ts`（`resolveLocalExtensionSource` 直接 resolve 路径，不调 `runNpmCommand`）。

---

## 完整示例：三种依赖都声明

```json
{
  "name": "my-pi-analyzer",
  "version": "1.0.0",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": [
      "./extensions/analyzer.ts",
      "node_modules/other-pi-pkg/extensions"
    ],
    "skills": [
      "./skills",
      "node_modules/other-pi-pkg/skills"
    ],
    "prompts": [
      "./prompts",
      "node_modules/other-pi-pkg/prompts"
    ]
  },
  "peerDependencies": {
    "@earendil-works/pi-ai": "*",
    "@earendil-works/pi-agent-core": "*",
    "@earendil-works/pi-coding-agent": "*",
    "@earendil-works/pi-tui": "*",
    "typebox": "*"
  },
  "dependencies": {
    "axios": "^1.7.0",
    "diff": "^5.2.0",
    "other-pi-pkg": "^2.0.0"
  },
  "bundledDependencies": ["other-pi-pkg"]
}
```

**字段为什么这么写**：

| 字段 | 为什么 / 不写会怎样 |
|------|-------------------|
| `peerDependencies` 5 包 + `"*"` | 让 pi 在运行时通过 alias / virtualModules 提供实例；npm install 时 `--legacy-peer-deps` 跳过下载。漏列 → tarball 多打包 pi 内核 / 消费者 install 时 npm 尝试自动下载 |
| `dependencies.axios` / `diff` | 第三方包声明版本范围，pi install 时下载到 installRoot 共享 `node_modules/`。漏写 → 消费者加载时报 `Cannot find module 'axios'` |
| `dependencies.other-pi-pkg` + `bundledDependencies.other-pi-pkg` | 必须同时声明。`dependencies` 给版本范围，`bundledDependencies` 让 npm pack 把它打进 tarball。漏 bundledDependencies → tarball 里没这个包，消费者 install 后 `node_modules/other-pi-pkg/` 不存在 |
| `pi.extensions` 含 `node_modules/other-pi-pkg/extensions` | 通过相对路径引用 bundled 的 pi package 资源。pi 用 `bundledDependencies` 打包后会保证这个路径在 tarball 内存在 |

### 扩展代码示例

```ts
// extensions/analyzer.ts — 三种 import 都能正常工作
import axios from "axios";                          // dependencies（installRoot node_modules）
import { diffLines } from "diff";                   // dependencies（installRoot node_modules）
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";  // peerDependencies（pi 提供）
// ⚠️ 不能 import "other-pi-pkg" —— 它是 pi package 不是 JS 模块
// 它的资源（extensions/skills）通过 pi.prompts 字段加载，不通过 import

export default (pi: ExtensionAPI) => {
  pi.registerTool({
    name: "analyze_deps",
    description: "Analyze npm dependencies for vulnerabilities",
    parameters: Type.Object({ packageName: Type.String() }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      const { data } = await axios.get(
        `https://api.example.com/audit/${params.packageName}`
      );
      return {
        content: [{ type: "text", text: JSON.stringify(data, null, 2) }],
      };
    },
  });
};
```

---

## 自定义包管理器（`npmCommand`）

CI / 多版本 Node 环境下，可能需要用 mise / asdf / nvm 包装的 npm。在 `settings.json` 里配置：

```json
{
  "npmCommand": ["mise", "exec", "node@20", "--", "npm"]
}
```

**对依赖安装的影响**：
- pi 识别 `--` 后面的命令名（`npm` / `pnpm` / `bun`）决定走哪条 `getNpmInstallArgs` 分支
- git 包 install 命令也会用这个包装器（但**不带 `--legacy-peer-deps`**）
- bun binary 模式下 `virtualModules` 仍然生效（不走 npm）

证据：`package-manager.ts`（`getNpmCommand` + `getPackageManagerName`）。

---

## 版本冲突与 hoisting（B 档）

installRoot 共享 `node_modules/` 意味着多个 pi package 依赖同一个第三方包时，按 npm hoisting 规则：

**场景**：扩展 A `dependencies.lodash: ^4.17.0`，扩展 B `dependencies.lodash: ^3.10.0`

**结果**：
- npm 选一个版本 hoist 到 `node_modules/lodash/`（通常是版本号更高的）
- 另一个版本被嵌套到 `node_modules/<需要旧版本的包>/node_modules/lodash/`
- jiti 从扩展文件所在目录向上解析 `import "lodash"`：如果扩展在 `node_modules/A/extensions/my-ext.ts`，会先找 `node_modules/A/node_modules/lodash` → 没有再找 `node_modules/lodash`

**实用建议**：
- 第三方包写 `^x.y.z` range 而不是精确版本，让 npm hoisting 更容易选
- 高敏感场景（扩展强依赖某个 lodash v3 API）考虑 `bundledDependencies` 物理隔离
- 不要在 `dependencies` 里写 pi 内核包——会被忽略，且加剧 hoisting 复杂度

---

## 常见误期待与陷阱

1. **「pi 内核包写 `dependencies` 也行」**——错。`@earendil-works/pi-ai` / `pi-agent-core` / `pi-coding-agent` / `pi-tui` / `typebox` 必须 `peerDependencies` + `"*"`。写 dependencies 时 npm 会下载，但 pi 加载时走 alias / virtualModules 重定向，下载的包**完全被忽略**——只是占 tarball 空间。证据：`loader.ts`。
2. **「peerDependencies 列 1-2 个核心包就行」**——错。必须列完整 5 个。漏列会导致 npm 不确定该不该装，极端情况 tarball 误打包 pi 内核。证据：`docs/packages.md`。
3. **「每个 pi package 有独立 module root，依赖不会冲突」**——**半错**（这是最常见的误解）。**只有 `bundledDependencies` 物理打包的 pi package 才有真正隔离**。第三方 `dependencies` 全部装在 installRoot 共享 `node_modules/`，多个 pi package 的同名依赖会走 npm hoisting。证据：package-manager.ts（`getManagedNpmInstallPath`）；packages.md（"separate module roots" 紧接 bundledDependencies 段落）。
4. **「`typebox` 写错了，应该是 `@sinclair/typebox`」**——半错。pi 的 alias 表同时支持 `typebox` 和 `@sinclair/typebox`（loader.ts 的 alias 表），写哪个都行。但**官方 docs (`packages.md`) 写的就是 `typebox`**——和 pi 内部约定一致，建议跟随官方写法。
5. **「pi 会自动 TypeScript 编译我的扩展」**——对，但有限制。jiti 在加载时即时编译 `.ts`，**不预编译**。扩展 import 的必须是真实 npm 包（不能是 tsconfig 的 path mapping）。证据：`loader.ts`。
6. **「`bundledDependencies` 拼写无所谓」**——⚠️ 部分对。npm 同时接受两种拼写：`bundledDependencies`（双 d，字符串数组）和 `bundleDependencies`（单 d，同义）——这是 npm 官方行为，pi 源码不参与判断。**两种都合法，选一个用即可**。但 pi 官方文档（`packages.md`）和示例统一用 `bundledDependencies`（双 d），建议跟随官方写法保持一致。真正要避免的是拼写错误（如 `bundledDependency`、`bunddependencys` 等 npm 不认识的变体）。
7. **「git 包和 npm 包 install 命令一样」**——错。npm 包走 `getNpmInstallArgs`（带 `--legacy-peer-deps`），git 包走 `getGitDependencyInstallArgs`（默认 `install --omit=dev`，**不带 `--legacy-peer-deps`**）。git 包的 peerDependencies 必须格外严格，否则 npm 会去下载 pi 内核包污染 node_modules。证据：package-manager.ts（`getNpmInstallArgs`）。
8. **「`pi install` 会跑 `npm install` 安装我 dependencies 里的所有包」**——对 user/project scope 的 npm 包是对的。但 **local source 不跑 npm install**——pi 直接读磁盘，扩展的 node_modules 自己管。证据：`package-manager.ts`。
9. **「扩展 import axios 后 pi 会自动装 axios」**——错。pi 装的是 `package.json` 里**声明**的 dependencies。代码里 `import "axios"` 但 package.json 没写 → 加载时 `Cannot find module 'axios'`。
10. **「`bundledDependencies` 不用同时在 `dependencies` 里声明」**——错。npm 要求 `bundledDependencies` 里的包**必须也在 `dependencies` 里有版本声明**，否则 `npm pack` 报错。证据：npm 官方文档。
11. **「`peerDependencies` 写精确版本更稳」**——错。pi 内核包在 `peerDependencies` 里**必须写 `"*"`**——pi 用 alias 重定向，根本不会真的查找 peer 版本。写 `^1.0.0` 反而可能让某些包管理器（pnpm strict 模式）报版本不匹配。
12. **「不同 pi package 共享 axios 是 bug」**——错。这是 installRoot 共享 `node_modules/` 的设计结果，**官方支持的特性**。如果真需要隔离，用 `bundledDependencies` 物理打包。

---

## 变体与延伸

- **打包发布全流程**（写 `package.json` / `pi` 字段 / 组织目录）→ [I01](I01-pi-package.md)
- **分发到团队 / 社区**（npm publish / git push / `.pi/settings.json` 共享）→ [I02](I02-distribute-extension.md)
- **扩展开发基础**（ExtensionAPI / registerTool / 事件）→ [E02](E02-extension-basics.md)
- **SDK 内联加载**（程序化加载，npm/git 源走临时 install）→ [A06](A06-load-extensions.md)
- **沙箱运行不可信扩展** → [I04](I04-sandbox.md)
