# 场景：分发扩展到团队 / 社区 (I02)

## 这是什么

**分发** = 把已打包的 Pi Package 通过 **npm / git / 私有 registry / 团队共享 settings** 等渠道送到消费者手上，让团队成员或社区用户通过 `pi install` 一键加载。

**不是什么**：
- 不是「**打包**」——打包（写 `package.json` / `pi` 字段 / 组织目录结构）是 [I01](I01-pi-package.md) 的事。I02 假设你已经有了能用的 Pi Package
- 不是「**SDK 内联加载**」——SDK 通过 `additionalExtensionPaths` 加载不走 `pi install`，是 [A06](A06-load-extensions.md) 的事
- 不是「**subagent 委托**」——subagent 是运行时任务委托，和分发无关（[I05](I05-subagent.md)）

---

## 什么时候要分发 / 不用会怎样

| 触发场景 | 用 I02 | 不用的话 |
|---------|--------|---------|
| 团队成员要复用你的扩展 | 写到 `.pi/settings.json` 提交 git，成员 clone 即得 | 每个人手动 `cp` 目录、版本漂移 |
| 公开发布到 pi 社区 | 发 npm + `keywords: ["pi-package"]` 进画廊 | 用户搜不到，只能靠 README 链接 |
| 私有团队 / 企业内部 | 发私有 npm / git URL（SSH key 鉴权）| 共享 zip 不安全、不版本化 |
| 单项目一次性扩展 | **不分发**，直接用 `.pi/extensions/` | 打成 package 是过度工程 |

**铁律**：如果你**只有一两个扩展且不跨项目复用**，不需要 I02——直接放 `.pi/extensions/` 约定目录即可（详见 [I01](I01-pi-package.md)）。I02 是为**复用、共享、版本管理**而生。

---

## I02 vs I01 vs A06 角色边界

读者经常混淆这三个场景。下表说清各自的边界：

| 场景 | 定位 | 典型动作 | 输出物 |
|------|------|---------|--------|
| [I01](I01-pi-package.md) | **打包** | 写 `package.json` / `pi` 字段 / 组织目录 | 一个**可分发的包** |
| **I02（本文档）** | **分发** | `npm publish` / `git push` / 共享 `.pi/settings.json` | **消费者能 install 的源** |
| [A06](A06-load-extensions.md) | **SDK 加载** | `additionalExtensionPaths` API 调用 | **运行时加载的扩展实例** |
| [I03](I03-extension-deps.md) | 依赖管理 | 处理 `dependencies` / `peerDependencies` / `bundledDependencies` | 扩展的**依赖图** |

**一句话**：I01 是「**做包**」、I02 是「**卖/送/装包**」、A06 是「**SDK 直接用**」。

---

## 涉及 SDK

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| `npm publish` / `git push` | 把包推到分发渠道（**通用 npm/git 知识，pi 不参与**） | npm / git 官方文档 |
| `package.json` `files` 字段 | 声明 tarball 包含哪些文件（**漏了 pi 资源就分发不到**） | npm 官方文档 |
| `pi install <source>` | 消费者侧加载包到 user / project | [sdk_doc/20-pi-package.md](../sdk_doc/20-pi-package.md) |
| `pi install -l <source>` | 写到 `.pi/settings.json`（**需要 project trust**） | 同上 |
| `pi list` / `pi remove` / `pi update` | 消费者侧管理已安装的包 | 同上 |
| `.pi/settings.json` `packages` 字段 | 团队共享的项目级依赖声明 | 同上 |
| `settings.json` `npmCommand` 字段 | **包管理器定制**（mise / asdf / pnpm / bun）。pi 内部透传 npm 调用，默认用 `npm`；用版本管理器或非 npm 包管理器时必须显式指定，否则 install 会报"找不到 npm"。例：`"npmCommand": ["mise", "exec", "node@20", "--", "npm"]` | `docs/packages.md`「npm」段 |

> 本文聚焦**分发链路**（作者 → 渠道 → 消费者）。命令矩阵的完整说明见 [I01](I01-pi-package.md)。

---

## 三种分发渠道对比

| 维度 | npm | git | 本地路径 |
|------|-----|-----|---------|
| **格式** | `npm:@scope/pkg@1.2.3` | `git:host/path@ref` / 完整 URL | `/abs/path` / `./rel/path` |
| **版本锁定** | 精确版本被 pin，range 不 pin | ref 是 pin（tag/branch/commit） | **无锁定**（直接引用，不复制） |
| **私有性** | 私 scope + registry 鉴权 | SSH key / 私有仓库权限 | 物理路径隔离 |
| **发现机制** | pi.dev 包画廊（`keywords: ["pi-package"]`）| 仓库 URL 直接分享 | 不发布，无画廊 |
| **版本管理** | semver，`npm version` 命令 | git tag / branch | 无内置版本 |
| **CI 友好** | ✓（设置 `NPM_TOKEN`） | ✓（设置 `GIT_SSH_COMMAND`） | ✗（依赖机器） |
| **典型场景** | 公开发布 / 跨团队复用 | 私有仓库 / 企业内部 | 开发调试 / 临时试用 |

---

## 分发渠道一：npm（推荐）

### 作者侧：完整发布流程

#### 1. 准备 `package.json`

完整字段说明见 [I01](I01-pi-package.md)。这里只强调**分发相关**的关键字段：

```json
{
  "name": "@myteam/pi-code-reviewer",
  "version": "1.2.0",
  "description": "Automated code review extension for pi-agent",
  "keywords": ["pi-package", "code-review"],
  "files": [
    "extensions/",
    "skills/",
    "prompts/",
    "themes/"
  ],
  "pi": {
    "extensions": ["./extensions/*.ts"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"],
    "image": "https://raw.githubusercontent.com/myteam/pi-code-reviewer/main/screenshot.png"
  },
  "peerDependencies": {
    "@earendil-works/pi-ai": "*",
    "@earendil-works/pi-agent-core": "*",
    "@earendil-works/pi-coding-agent": "*",
    "@earendil-works/pi-tui": "*",
    "typebox": "*"
  },
  "dependencies": {
    "diff": "^5.0.0"
  }
}
```

**关键字段为什么这么写**：

| 字段 | 为什么 / 不写会怎样 |
|------|-------------------|
| `name` | scoped 名（`@myteam/...`）可发私有 registry；无 scope 默认发公共 npm |
| `version` | semver。消费者 `pi install npm:pkg@1.2.0` 锁定，`pi install npm:pkg` 不锁 |
| `keywords: ["pi-package"]` | **必须包含** `"pi-package"` 才能在 [pi.dev 包画廊](https://pi.dev/packages) 被发现。`pi` 不强制校验，但画廊用这个 keyword 过滤 |
| `files` | **npm tarball 的 whitelist**。漏写 `extensions/` 等目录，tarball 里就没有这些文件，消费者 install 后扩展加载不到 |
| `pi.image` / `pi.video` | **画廊端元数据，pi CLI 不会读取这两个字段**（`PiManifest` 接口只认 `extensions` / `skills` / `prompts` / `themes` 四个字段）。仅供 [pi.dev 包画廊](https://pi.dev/packages) 消费：`image` 支持 PNG/JPEG/GIF/WebP；`video` 仅 MP4 + 桌面端 hover 自动播放；两者同设时 **video 优先**。证据：`pi-manifest.ts`（`PiManifest` 接口 + `RESOURCE_FIELDS`）。 |
| `peerDependencies` | **5 个 pi 核心包必须列在这里 + `"*"` 范围**，不能放 `dependencies`。详见 [I01](I01-pi-package.md#peerdependencies-5-个核心包不要打包) / [I03](I03-extension-deps.md) |

#### 2. 验证 tarball 内容（重要）

发布前**必须**本地模拟 install 验证 `files` 字段没漏资源：

```bash
# 模拟打包，看 tarball 里到底有什么
npm pack

# 解开 tarball 检查
tar -tzf myteam-pi-code-reviewer-1.2.0.tgz | grep -E "extensions|skills|prompts|themes"

# 如果某个目录没出现，就是 files 漏了，或 .npmignore 排除了
```

**常见坑**：`.npmignore` 和 `.gitignore` 的**交互规则**——`files` 字段是**白名单**（只列要打包的文件），`.npmignore` 是**排除清单**，两者**叠加生效**（白名单 ∩ 非排除）。`.npmignore` 存在时它**取代的是 `.gitignore`**（而非 `files`），所以「`.npmignore` 覆盖 `files`」的说法是错的。最简单的做法是**只用 `files` 字段**控制打包内容，删掉 `.npmignore`，避免两套规则混用漏掉 pi 资源。

#### 3. 登录 + 发布

```bash
# 首次需要登录（会打开浏览器）
npm login

# 公共包
npm publish

# scoped 包默认发私有的，发公共必须加 --access public
npm publish --access public
```

#### 4. 验证发布成功

```bash
# 看 npm registry 上能不能查到
npm view @myteam/pi-code-reviewer version

# 用 pi 试用一下
pi -e npm:@myteam/pi-code-reviewer@1.2.0
```

⚠️ **`pi -e` 仍然会下载到磁盘**（temporary scope，落在 `~/.pi/agent/tmp/extensions/` 下），只是不写 settings。详见 [I01 陷阱 #2](I01-pi-package.md#常见误期待与陷阱)。

### 消费者侧：发现 + 安装

```bash
# 在 pi.dev/packages 画廊搜索 keywords: pi-package
# 或直接 install（已知包名）

# 用户级安装（写到 ~/.pi/agent/settings.json）
pi install npm:@myteam/pi-code-reviewer@1.2.0

# 项目级安装（写到 .pi/settings.json，需要 project trust）
pi install -l npm:@myteam/pi-code-reviewer@1.2.0

# 不带版本号 = 不 pin，pi update --extensions 会升级
pi install npm:@myteam/pi-code-reviewer
```

**`pi install -l` 需要 project trust**：未 trust 项目会报 `Project is not trusted. Use --approve to modify local package config.`。一次性覆盖用 `-a`（`--approve` 的短形式），反向用 `-na`（`--no-approve`）本次忽略项目文件。证据：`package-manager-cli.ts`（`--approve` / `-a` → `projectTrustOverride = true`；`--no-approve` / `-na` → `projectTrustOverride = false`）。

---

## 分发渠道二：git（私有仓库首选）

### 作者侧

```bash
# 1. 推到仓库
git init && git add . && git commit -m "initial"
git remote add origin git@github.com:myteam/pi-code-reviewer.git
git push -u origin main

# 2. 打 tag（消费者用 tag 锁定版本）
git tag v1.2.0
git push origin v1.2.0
```

**`package.json` 仍要写**——git 包 install 时如果存在 `package.json`，pi 会自动跑 `npm install` 装依赖。

### 消费者侧：两种 URL 格式（★ 必须分清楚）

```bash
# 短格式：必须有 git: 前缀
pi install git:github.com/myteam/pi-code-reviewer@v1.2.0       # host/path 简写（parseGitUrl 自动补成 https://）
pi install git:git@github.com:myteam/pi-code-reviewer@v1.2.0   # SSH scp-like 简写

# 完整 URL：不需要 git: 前缀（必须是协议 URL）
pi install https://github.com/myteam/pi-code-reviewer@v1.2.0
pi install ssh://git@github.com/myteam/pi-code-reviewer@v1.2.0
pi install git://github.com/myteam/pi-code-reviewer@v1.2.0
```

**SSH 短格式 `git@github.com:user/repo` 必须配 `git:` 前缀**——不写前缀会被当成 local 路径。证据：`utils/git.ts`（`parseGitUrl`）。

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

`pi update --extensions` 在 git 包上**仅当 HEAD 与目标 ref 不一致时**才会执行 `git reset --hard <ref>` + `git clean -fdx`，**本地修改会被清掉**；HEAD 已对齐时只走 `repairMissingGitDependencies`（补缺失依赖，不 reset/clean）。证据：`package-manager.ts` 中 `ensureGitRef`（fetch 后比较 `rev-parse HEAD` 与 `rev-parse <ref>^{commit}`，相等则 `repairMissingGitDependencies` 返回，不等才写 marker → `reset --hard` → `cleanAndInstallGitDependencies`）；`clean -fdx` 在 `cleanAndInstallGitDependencies` 内（用于"重置/重装"路径）。

---

## 分发渠道三：本地路径（开发调试）

不算正式分发，主要用于开发调试。详见 [I01](I01-pi-package.md#local-source)。

```bash
# 绝对路径
pi install /absolute/path/to/my-ext

# 相对路径（相对 settings 文件所在目录解析）
pi install ./relative/path/to/my-ext

# 用 -e 临时加载（不写 settings，但仍然会读磁盘）
pi -e ./my-ext
```

**特点**：不复制文件、不锁版本、安装快但**依赖原路径存在**。

---

## 私有 npm registry / GitHub Packages（B 档）

企业内部不想发公共 npm 时，可选：

### 私有 npm registry（Verdaccio / Nexus / Artifactory）

在项目根 `.npmrc` 配置 scope 走私有 registry：

```ini
# .npmrc
@myteam:registry=https://npm.internal.myteam.com
//npm.internal.myteam.com/:always-auth=true
//npm.internal.myteam.com/:_authToken=${NPM_TOKEN}
```

之后 `npm publish` 自动走私有 registry，消费者 `pi install npm:@myteam/pkg` 也会走私有 registry（pi 内部就是 `npm install`）。

### GitHub Packages

```ini
# .npmrc
@myteam:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

**消费者侧也需要配置 `.npmrc`**——pi 透传 `npm install`，npm 找不到 registry 就会失败。

---

## 团队共享 `.pi/settings.json` 完整流程

这是**最常见的分发模式**——团队成员通过 git 拉项目代码 + `.pi/settings.json`，pi 启动时自动 install。

### 作者侧（技术负责人）

```bash
# 1. 在项目根创建 .pi/settings.json，声明项目依赖的 pi packages
mkdir -p .pi
cat > .pi/settings.json <<EOF
{
  "packages": [
    "npm:@myteam/pi-code-reviewer@1.2.0",
    {
      "source": "git:github.com/myteam/pi-linter@v0.4.0",
      "extensions": ["extensions/*.ts"]
    }
  ]
}
EOF

# 2. 提交到 git
git add .pi/settings.json
git commit -m "Add project pi packages"
git push
```

### 消费者侧（团队成员）

```bash
# 1. clone 项目
git clone git@github.com:myteam/project.git
cd project

# 2. 启动 pi
pi
```

**pi 启动时发生什么**：

1. **检测到 `.pi/settings.json` 存在** → 触发 project trust 询问（证据：`trust-manager.ts` 中 `TRUST_REQUIRING_PROJECT_CONFIG_RESOURCES` 含 `"settings.json"`）
2. **用户确认 trust** → pi 加载 project settings
3. **`packageManager.resolve()`** 发现 settings 里的 package 没安装 → 自动调 `installParsedSource` 真下载（证据：`package-manager.ts` 中 `resolvePackageSources` 的 `if (!onMissing) await this.installParsedSource(parsed, scope); return true;`）
4. **下载完成** → 资源加载到运行时

**如果用户拒绝 trust**：`.pi/settings.json` 里的 packages **完全被忽略**——不会下载、不会加载。这是安全设计。

### 跨设备同步行为

- **不同设备 trust 状态独立**：trust 信息存在 `~/.pi/agent/trust.json`，不在项目里。每台设备/每个用户首次都要单独 trust
- **scope 去重规则**：同一个包同时出现在 user 和 project settings 时，**project 胜出**（证据：`package-manager.ts` 中 `dedupePackages` / `getPackageIdentity`，project 先入列、user 后入列，命中同身份 key 时 user 被丢）。身份 key：
  - npm → `npm:<name>`（不含版本）
  - git → `git:<host>/<path>`（**忽略 ref**）
  - local → 解析后的绝对路径
- **例外：`autoload: false` 当 delta 用**。如果 project 条目用 object 形式且 `"autoload": false`，则**不替换** user 那条，而是把 user 条目作为增量叠加保留——这是让项目「关掉某包的自动加载但不覆盖用户已装的过滤配置」的关键机制。证据：`package-manager.ts` 中 `dedupePackages`（`existing.scope === "project" && entry.scope === "user"` 且 `existing.pkg.autoload === false` 时 `result.push(entry)`）。

---

## 消费者侧管理已装包资源（B 档）

包装好之后，消费者除了 install / update / remove，还有两类高频管理操作：

### `pi config`：启用 / 禁用单个资源

装完一个包，想关掉它的某个 extension / skill / prompt / theme，**不用改 settings 文件**——用 `pi config`：

```bash
pi config        # 默认从全局 settings（~/.pi/agent/settings.json）开始
pi config -l     # 从 project overrides（.pi/settings.json）开始，已继承的全局资源显示为灰色
```

进入交互界面后可逐项 enable/disable，Tab 在全局 / 项目模式间切换。详见 `docs/packages.md`「Enable and Disable Resources」。

### settings object form：过滤包内资源（`!` / `+` / `-`）

包内资源太多、只想加载一部分时，用 `packages` 的 object 形式写过滤规则（在 manifest 基础上**收窄**，不能扩大）：

```json
{
  "packages": [
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

- 省略某个 key → 加载该类型的全部
- `[]` → 该类型一个都不加载
- `!pattern` → 排除匹配项
- `+path` → 强制包含精确路径（相对包根）
- `-path` → 强制排除精确路径

详见 `docs/packages.md`「Package Filtering」。

---

## 版本管理策略（B 档）

### semver 三段版本号

| 阶段 | 版本号 | 升级规则 | 消费者写法 |
|------|--------|---------|-----------|
| **修 bug** | `1.2.0` → `1.2.1` | 不破坏 API | `^1.2.0` 自动升级 |
| **加功能** | `1.2.1` → `1.3.0` | 不破坏 API，加新能力 | `^1.2.0` 自动升级 |
| **破坏性变更** | `1.3.0` → `2.0.0` | 破坏 API | 消费者**必须手动**升级 |
| **预发布** | `2.0.0-beta.1` | 不稳定 | 默认不装，`pi install npm:pkg@2.0.0-beta.1` 显式 |

### pinned vs range（消费者侧）

```bash
# pinned（精确版本）—— pi update --extensions 跳过，不会切版本
pi install npm:@foo/bar@1.2.0

# range—— pi update --extensions 会查最新
pi install npm:@foo/bar           # 不写版本
pi install npm:@foo/bar@^1.2.0    # 写 range（★ pi 会把 ^1.2.0 当 range，不当 pin）
```

**证据**：`package-manager.ts` 中 `isExactNpmVersion(version)`——**只有精确版本号是 pin**，range（`^` / `~` / `>=`）不是 pin。

### `pi update` 子目标矩阵

`pi update` 支持多种子目标，互斥（`--all` 不能和其他目标同用）：

| 写法 | 更新对象 | 备注 |
|------|---------|------|
| `pi update` | pi 自身 | 无目标时的默认行为 |
| `pi update --self` | pi 自身 | 显式 |
| `pi update --extensions` | 所有已装包（pinned npm 跳过） | 见上文 pinned vs range |
| `pi update --models` | 刷新模型目录 | |
| `pi update --all` | pi + 所有已装包 | 等于 `--self` + `--extensions` |
| `pi update --extension <source>` | 仅更新指定的一个包 | |
| `pi update <source>` | 仅更新指定的一个包（位置参数） | |
| `pi update pi` | pi 自身（`pi` 作为 self 别名） | |

证据：`package-manager-cli.ts`（`parsePackageCommand` 中 `--self` / `--extensions` / `--models` / `--all` / `--extension <source>` / 位置参数分支 + 互斥校验）；`docs/packages.md`「Update」段。

### git tag / branch

git 包用 ref 锁定：

| ref 类型 | 行为 | 用法 |
|---------|------|------|
| **tag**（`v1.2.0`） | 推荐，不可变 | `pi install git:host/repo@v1.2.0` |
| **branch**（`main`） | 会随 upstream 变化 | `pi install git:host/repo@main` |
| **commit hash** | 最精确 | `pi install git:host/repo@abc1234` |

⚠️ **branch ref 不会自动跟随 upstream**——`pi update --extensions` 只会 `git reset` 到**配置时的 ref**。要切 ref 必须重 `pi install`。

---

## 安全警告（**必读**）

> Pi packages 以**完整系统权限**运行。Extension 可以执行任意代码，skill 可以引导模型执行任意操作（包括运行可执行文件）。
>
> **安装第三方 package 前必须审查源码**。`pi install npm:@stranger/awesome-ext` 等价于 `npm install` + 让陌生人代码接管你的 Agent。

证据：`docs/packages.md`。

### 审查建议

```bash
# 查看已 install 包的源码（user scope，注意多一层 node_modules/）
ls ~/.pi/agent/npm/node_modules/<pkg-name>/
cat ~/.pi/agent/npm/node_modules/<pkg-name>/extensions/*.ts

# git 包源码位置
ls ~/.pi/agent/git/<host>/<path>/

# 项目级（同样多一层 node_modules/）
ls .pi/npm/node_modules/<pkg-name>/
ls .pi/git/<host>/<path>/
```

**不可信扩展必须靠操作系统级隔离**——pi **没有内置 sandbox**，也没有 settings 级别的 sandbox 字段（内置工具、扩展、shell 命令都以 pi 进程权限运行）。官方建议用容器 / VM / microVM / 远程沙箱（详见 `docs/security.md`「No Built-in Sandbox」+ `docs/containerization.md`：把整个 `pi` 跑在 Docker 里，或用 Gondolin micro-VM 路由工具执行，只挂载必要工作区路径）。

---

## CLI 的环境变量

CI / 自动化场景必备：

| 变量 | 作用 | 典型场景 |
|------|------|---------|
| `PI_OFFLINE=1` | 禁用网络检查，全部用本地缓存 | 离线环境 / 加速启动 |
| `GIT_TERMINAL_PROMPT=0` | 禁用 git 凭据提示（**失败而非挂起**）。pi 内部已为 `ls-remote` 等**远程查询**自动设置此变量，但 clone/fetch 阶段继承自进程环境——CI 显式 export 是为了覆盖这两个阶段 | CI 必备 |
| `GIT_SSH_COMMAND` | 自定义 SSH 命令 | CI 用 deploy key / 设超时 |

CI 示例：

```bash
# GitHub Actions 例子
export GIT_TERMINAL_PROMPT=0
export GIT_SSH_COMMAND="ssh -o BatchMode=yes -o StrictHostKeyChecking=accept-new"
pi install -l --approve git:github.com/myteam/pi-tools@v1.0.0
```

`--approve` 用于 CI 跳过 project trust 询问（直接信任）。

---

## CI 自动 publish（C 档）

略——属于通用 npm / git 知识，参考：
- npm: `npm publish` + `NPM_TOKEN` secret
- GitHub Actions: `actions/setup-node` + `npm publish`
- 详见 npm 官方文档「GitHub Actions 中发布 npm 包」

pi 不参与 publish 流程，只是消费者侧的 install 工具。

---

## 常见误期待与陷阱

1. **「`files` 字段可以省略，npm 默认全打包」**——错。npm 默认遵循 `.gitignore` / `.npmignore`，**很容易把 pi 资源漏掉**。必须显式写 `"files": ["extensions/", "skills/", "prompts/", "themes/"]`，发布前 `npm pack` 验证。
2. **「`.npmignore` 和 `files` 可以同时用」**——可以，但要理清关系。`files` 是**白名单**，`.npmignore` 是**排除清单**，两者**叠加生效**（白名单 ∩ 非排除）；`.npmignore` 存在时它**取代的是 `.gitignore`**（而非 `files`）。最简单的做法是**只用 `files` 字段**控制打包内容，删 `.npmignore`。
3. **「`pi -e <source>` 不下载」**——错。`-e`（`--extension`）只是不写 settings，**仍然会 `npm install` / `git clone` 到磁盘**（temporary scope，而非 user/project 的正式安装目录）。两条路径结构不对称：npm 落在 `~/.pi/agent/tmp/extensions/npm/<hash>/`，git 落在 `~/.pi/agent/tmp/extensions/git-<host>/<hash>/<path>/`（git 前缀带 host，且多一层 `<path>`）。要真正不下载只能用 local 路径。证据：`-e` 解析于 `parseArgs`（`args.ts`）；temporary scope 由 `resolvePackageSources` 按需 `installParsedSource`，目录由 `getTemporaryDir`（`package-manager.ts`，npm 前缀 `npm`、git 前缀 `` `git-${source.host}` ``，再拼 `source.path` 作为 suffix）+ `getExtensionTempFolder`（`tmp/extensions`）。
4. **「`pi install -l` 默认能写项目配置」**——错。需要项目已 trust。未 trust 时报 `Project is not trusted. Use --approve to modify local package config.`。CLI 实际支持两种覆盖：`-a` / `--approve`（本次命令信任项目，写 project settings）和 `-na` / `--no-approve`（本次命令忽略项目文件）。证据：`package-manager-cli.ts`（`--approve` / `-a` → `projectTrustOverride = true`；`--no-approve` / `-na` → `projectTrustOverride = false`）；错误文本在同文件「Project is not trusted. Use --approve to modify local package config.」。
5. **「团队成员 clone 项目就能用 pi packages」**——**部分错**。pi 启动时检测到 `.pi/settings.json` 会先**询问 trust**，用户拒绝 trust 时所有 packages 都不加载（安全设计）。证据：`trust-manager.ts`（`hasTrustRequiringProjectResources` / `TRUST_REQUIRING_PROJECT_CONFIG_RESOURCES`）。
6. **「`pi update <pkg>` 能升级 pinned 包」**——错，要分两种 pin 看。**pinned npm（精确版本）确实被跳过**，不会升级。但**pinned git（带 ref）不会被跳过**——`updateConfiguredSources` 会把 git 包无条件入列，调 `updateGit` 把现有 clone **协调（fetch + reset）到配置时的 ref**；它只是不主动移动到更新的 ref。所以要换版本，pinned npm 要重 `pi install <source>@<new>`，pinned git 要重 `pi install git:...@<new-ref>`。证据：`updateConfiguredSources`（npm 跳 `!pinned`，git 无条件入列）。
7. **「`git@github.com:user/repo` 不写 `git:` 前缀也行」**——错。`parseSource` 先查 `isLocalPath()`，再 fallback 到 `parseGitUrl`；`parseGitUrl` 在**无 `git:` 前缀时只接受协议 URL**（正则 `/^(https?\|ssh\|git):\/\//i`），scp-like 短格式（`git@host:path`）和纯 host/path（`github.com/user/repo`）都返回 null → 最终走 `{type: "local"}` 分支被当成本地路径。所以 scp-like 短格式必须带 `git:` 前缀走 shorthand 分支。证据：`utils/git.ts`（`parseGitUrl` 正则）；`package-manager.ts`（`parseSource` 先 `isLocalPath` 后 `parseGitUrl`，都失败 fallback `{type: "local"}`）。
8. **「`keywords: ["pi-package"]` 是 pi 强制校验的」**——半错。pi 不强制校验，但 pi.dev 包画廊用这个 keyword 发现包——不发 npm 只本地用可以不写。
9. **「peerDependencies 列 1-2 个核心包就行」**——错。必须列完整 5 个：`pi-ai` / `pi-agent-core` / `pi-coding-agent` / `pi-tui` / `typebox`，全部 `"*"` 范围。漏列会让 tarball 把 pi 内核包一起打包，而 pi install 时**强制跳过 peer 解析**（npm `--legacy-peer-deps` / bun `--omit=peer` / pnpm `--config.auto-install-peers=false`），且每个包用**独立 module root** 加载——结果消费者侧形成两份 pi 内核实例，扩展调用的 API 和 pi 主进程的 API 不一致，行为分裂。证据：`package-manager.ts`（`getNpmInstallArgs` 分包管理器分支）；`docs/packages.md`（"Pi loads packages with separate module roots"）。
10. **「同时装在 user 和 project 会出现两份」**——错。project 胜出，user 那条被去重忽略（按身份 key，不按 ref）。证据：`package-manager.ts`（`dedupePackages` / `getPackageIdentity`）。
11. **「git ref 会自动跟随 upstream 更新」**——错。ref 是 pin，`pi update --extensions` 只 `git reset` 到**配置的 ref**，不会移动到新 tag。要切 ref 必须重 install。
12. **「scope 名（`@myteam/...`）只是装饰」**——错。scope 决定 npm registry 路由（`.npmrc` 里 `@myteam:registry=...`），决定公共/私有、鉴权方式。

---

## 变体与延伸

- **打包发布全流程**（写 `package.json` / `pi` 字段 / 组织目录）→ [I01](I01-pi-package.md)
- **扩展引用第三方 npm 依赖**（axios、lodash、其他 pi package 等）→ [I03](I03-extension-deps.md)
- **SDK 内联加载**（不走 CLI，程序化加载）→ [A06](A06-load-extensions.md)
- **Subagent 进程级委托**（与分发无关，但容易混淆）→ [I05](I05-subagent.md)
