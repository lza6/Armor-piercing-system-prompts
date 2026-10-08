# 场景：沙箱与隔离执行 (I04)

## 这是什么 / 不是什么

**这是**：pi-agent **生态中"如何把 agent 关进笼子"的场景导航**——当你要跑不可信代码、自动化任务、敏感项目时，怎么用 OS 级边界（容器/VM/扩展）限制 pi 的文件/网络/进程权限。

**不是**：
- ❌ pi **没有"内置 sandbox 配置"**。`settings.json` 里**没有** `sandbox` 字段；`packages` 数组里也**不能**写 `{ sandbox: { ... } }`。官方明确说「Pi does not include a built-in sandbox」（[security.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/security.md)）——**这是故意的设计决策**：「A partial in-process sandbox would be easy to misunderstand as a security boundary」
- ❌ pi **没有"沙箱化的扩展运行时"**。所有扩展都是普通 TypeScript 模块，**和 pi 主进程拥有完全相同的权限**（读/写文件、起子进程、发网络请求）。`ExtensionAPI` 不提供任何沙箱 API
- ❌ `pi` CLI **没有内置源码审查命令**。`pi list` 只列已配置包的 scope/source，**不接受 `--details`，也不接受 source 参数**

→ 如果你看到任何文档/教程声称「在 settings.json 配置 sandbox 字段就能限制扩展」，那是错的。

## 什么时候要管 / 不管会怎样

| 场景 | 不管会怎样 | 该怎么做 |
|------|-----------|---------|
| 跑来源不明的第三方扩展（`pi install npm:untrusted-xxx`） | 扩展能读 `~/.ssh`、`~/.aws`、`.env`，能把数据外发，能 `rm -rf` | **不要直接装**。先用 Docker/Gondolin 容器化；或**人工审源码**后再装到可信环境 |
| 在 CI / unattended automation 跑 pi | 凭证泄露、prompt injection 写出主机 | **必容器化**。容器内只放必要凭证、只挂必要路径 |
| 团队共享 `.pi/settings.json`（含扩展/skills） | 队友 pull 后自动加载项目扩展，扩展能执行任意代码 | 队友首次进入会触发 **project trust 询问**（不是沙箱，只是输入加载守卫）。仍建议容器内审 |
| 在本机跑，只是不想让 pi 误写某些文件 | 模型 prompt injection 或工具误调用可能动到 `~/.ssh` 等 | 用 `examples/extensions/sandbox/` 扩展（OS 级沙箱）或 `protected-paths.ts` 扩展 |
| 跑自己的内部代码 / 完全可信的扩展 | 没事 | **不用管**——pi 默认权限模型就是为可信本地协作设计的 |

## 涉及 SDK / 能力

| 能力 | 用途 | 详细文档 |
|------|------|---------|
| 官方"无内置 sandbox"立场 | 理解为什么 pi 不在进程内做沙箱 | [pi-coding-agent/docs/security.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/security.md) |
| Containerization 三种模式 | 容器/VM 级隔离的官方推荐路径 | [pi-coding-agent/docs/containerization.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/containerization.md) |
| `project trust` 机制 | 项目级配置/扩展的"加载前确认"守卫（**不是沙箱**） | [sdk_doc/13-settings-manager.md](../sdk_doc/13-settings-manager.md) |
| `pi install` 包管理 | 装/列/卸第三方扩展（**没有源码审查命令**） | [sdk_doc/20-pi-package.md](../sdk_doc/20-pi-package.md) |
| 扩展运行时 | 扩展能力边界（和主进程相同） | [sdk_doc/07-extensions-api.md](../sdk_doc/07-extensions-api.md) |

## 核心立场：为什么 pi 不在进程内做沙箱

引用官方 [security.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/security.md)：

> **No Built-in Sandbox**
>
> Pi does not include a built-in sandbox. Built-in tools can read files, write files, edit files, and run shell commands with the permissions of the pi process. Extensions are TypeScript modules that run with the same permissions. Package installs, shell commands, language servers, test commands, and other developer tools behave as ordinary local processes.
>
> This is intentional. Pi is designed to operate on local source trees, invoke project toolchains, and integrate with the user's existing development environment. A partial in-process sandbox would be easy to misunderstand as a security boundary while still depending on the host shell, filesystem, package managers, credentials, and extension code. Real isolation needs to come from the operating system or a virtualization/container boundary.

**翻译**：Pi 故意不内置 sandbox。内置工具能读写文件、起 shell，权限和 pi 进程一样。扩展也是普通 TS 模块，权限相同。这是**设计决策**——pi 定位是"操作本地源码树、调用项目工具链、融入既有开发环境"，进程内沙箱会**给人一种虚假的安全感**，但实际仍依赖宿主 shell/文件系统/包管理器/凭证/扩展代码。真正的隔离**只能来自操作系统或虚拟化/容器边界**。

**关键含义**：
- 你**不能**靠"配置某个字段"在 pi 内部隔离某个扩展。扩展只要能 `require('child_process').exec(...)` 就能跑任意命令
- 你**能**做的：① 把整个 pi 进程关进容器；② 用"工具路由扩展"把内置工具替换成走外部 VM 的实现；③ 用 OS 级 syscall 拦截（macOS sandbox-exec / Linux bubblewrap）限制 bash 子进程

## 三种官方推荐的隔离模式

来源：[containerization.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/containerization.md) 第 11-15 行的模式对比表。

| 模式 | 隔离对象 | 适用场景 | 关键说明 |
|------|---------|---------|---------|
| **Gondolin 扩展** | 内置工具 + `!` 命令 | 本地 micro-VM 隔离，**鉴权留在宿主** | 见 `examples/extensions/gondolin/` |
| **Plain Docker** | 整个 `pi` 进程 | 简单本地隔离 | **Provider API key 会进入容器** |
| **OpenShell** | 整个 `pi` 进程 | 策略控制沙箱，本地或远端 | 需要 OpenShell gateway；NVIDIA 产品 |

### 关键不直觉点：扩展跟 pi 走

> Extensions run wherever the `pi` process runs. If you run host `pi` with a tool-routing extension, other custom extension tools still run on the host unless they also delegate their operations.
> — containerization.md

**翻译**：扩展运行在 pi 进程所在的位置。如果你在宿主跑 pi 并用一个"工具路由扩展"，**其他扩展的工具仍然跑在宿主**——除非它们也把操作委派出去。

**实战含义**：你装了 Gondolin 想隔离内置 `bash`/`write`，但同时还装了 `my-custom-ext`（它注册了 `my_tool`）——`my_tool` 仍然在宿主起子进程，**没被 Gondolin 隔离**。要全隔离必须**整个 pi 进程进容器**（Docker / OpenShell 模式）。

## 模式一：Gondolin 扩展（本地 micro-VM）

[Gondolin](https://github.com/earendil-works/gondolin) 是 earendil-works 自家的本地 Linux micro-VM。**特点**：宿主跑 pi，但内置工具（`read`/`write`/`edit`/`bash`/`grep`/`find`/`ls`）和用户 `!` 命令全部路由进 VM。**鉴权（API key）留在宿主**——比 Docker 整体装更安全。

要求：Node.js ≥ 23.6.0、QEMU（宿主用包管理器装，如 macOS `brew install qemu`、Linux `apt install qemu-system`）。micro-VM **客户机（guest）**是 Linux，宿主 OS 可为 macOS / Linux（Windows 宿主未支持，需要 QEMU+Linux 工具链）。

```bash
# 安装扩展到全局目录
cp -R packages/coding-agent/examples/extensions/gondolin ~/.pi/agent/extensions/gondolin
cd ~/.pi/agent/extensions/gondolin
npm install --ignore-scripts

# 在要挂载的项目里启动 pi
cd /path/to/project
pi -e ~/.pi/agent/extensions/gondolin
```

VM 把宿主 `cwd` 挂载到 `/workspace`，对 `/workspace` 下文件的写入**会写回宿主**（bind mount）。

> 注意：仅内置工具被路由。你自己装的 `my-custom-ext` 注册的工具**不会**被路由（见上一节"扩展跟 pi 走"）。

## 模式二：Plain Docker（整个 pi 进容器）

最简单的本地容器边界。**注意**：Provider API key 会进入容器（不像 Gondolin 留宿主）。

`Dockerfile.pi`：

```dockerfile
FROM node:24-bookworm-slim

RUN apt-get update \
  && apt-get install -y --no-install-recommends bash ca-certificates git ripgrep \
  && rm -rf /var/lib/apt/lists/*
RUN npm install -g --ignore-scripts @earendil-works/pi-coding-agent

WORKDIR /workspace
ENTRYPOINT ["pi"]
```

构建并运行：

```bash
docker build -t pi-sandbox -f Dockerfile.pi .

docker run --rm -it \
  -e ANTHROPIC_API_KEY \
  -v "$PWD:/workspace" \
  -v pi-agent-home:/root/.pi/agent \
  pi-sandbox
```

**挂载策略**（关键决策）：

| 挂载点 | 作用 | 副作用 |
|--------|------|--------|
| `-v "$PWD:/workspace"` | 项目目录写穿到宿主 | 容器内写 `/workspace/foo.txt` = 写宿主文件 |
| `-v pi-agent-home:/root/.pi/agent`（named volume） | 容器本地保存 settings/sessions | **不会**暴露宿主 `~/.pi/agent` 凭证 |
| `-v ~/.pi/agent:/root/.pi/agent`（bind mount） | 共享宿主 settings/sessions/凭证 | **危险**：宿主 API key 暴露给容器 |

**只读保护**：要防止容器写入宿主，用 `-v "$PWD:/workspace:ro"` 只读挂载，把工作产出 copy 出来而不是写穿。

## 模式三：OpenShell（策略控制 sandbox）

[NVIDIA OpenShell](https://docs.nvidia.com/openshell/about/overview) 适合需要**策略控制**（文件系统/进程/网络/凭证/推理都可控）的场景，支持本地（Docker/Podman/VM 后端）或远端（Kubernetes gateway）。

```bash
# 注册并选择 gateway
openshell gateway add <gateway-url> --name <name>
openshell gateway select <name>

# 在 OpenShell sandbox 内启动 pi
openshell sandbox create --name pi-sandbox --from pi -- pi
```

**特点**：
- 整个 pi 进程在 sandbox 内，所有工具/`!`/扩展都在边界内
- 远端 gateway 不会 bind-mount 宿主——必须 `openshell sandbox upload` 把代码送进去
- 推理路由可让模型 API key 留在 gateway，sandbox 内代码调 `https://inference.local`

## 附加方案：`examples/extensions/sandbox/` 扩展（OS 级 syscall 拦截）

pi 仓库的 [`examples/extensions/sandbox/`](https://github.com/earendil-works/pi-coding-agent/tree/main/packages/coding-agent/examples/extensions/sandbox) 给出另一种思路：**不换运行时**，而是用 [`@anthropic-ai/sandbox-runtime`](https://www.npmjs.com/package/@anthropic-ai/sandbox-runtime) 在 bash 子进程外层套 OS 沙箱（macOS `sandbox-exec` / Linux `bubblewrap`）。

**配置文件位置**（**不在** `settings.json`！）：

- 全局：`~/.pi/agent/extensions/sandbox.json`
- 项目：`<cwd>/.pi/sandbox.json`（项目覆盖全局，深度合并）

**配置示例**：

```jsonc
// .pi/sandbox.json
{
  "enabled": true,
  "network": {
    "allowedDomains": ["github.com", "*.github.com", "registry.npmjs.org"],
    "deniedDomains": []
  },
  "filesystem": {
    "denyRead": ["~/.ssh", "~/.aws", "~/.gnupg"],
    "allowWrite": [".", "/tmp"],
    "denyWrite": [".env", ".env.*", "*.pem", "*.key"]
  }
}
```

**字段含义**（来源：`examples/extensions/sandbox/index.ts`）：

| 字段 | 类型 | 含义 |
|------|------|------|
| `enabled` | `boolean` | 是否启用沙箱（默认 true） |
| `network.allowedDomains` | `string[]` | 允许访问的域名（glob 通配符），其他全拒 |
| `network.deniedDomains` | `string[]` | 显式拒绝的域名（即使 allow 中也拒） |
| `filesystem.denyRead` | `string[]` | 禁止读取的路径（如 `~/.ssh`） |
| `filesystem.allowWrite` | `string[]` | 允许写入的路径（如 `.` 和 `/tmp`） |
| `filesystem.denyWrite` | `string[]` | 显式禁止写入的路径（优先于 allow） |
| `ignoreViolations` | `Record<string, string[]>` | （高级）忽略特定规则违规，key 为规则类别、value 为通配符列表；用于放行已知误报。传给 `SandboxManager.initialize` |
| `enableWeakerNestedSandbox` | `boolean` | （高级）启用宽松嵌套沙箱模式，用于 pi 自身已在另一层沙箱内运行的场景。传给 `SandboxManager.initialize` |

> 完整字段以 `examples/extensions/sandbox/index.ts` 的 `DEFAULT_CONFIG` + `deepMerge`（处理 `ignoreViolations` / `enableWeakerNestedSandbox`）为准；上述两类高级字段透传给 `@anthropic-ai/sandbox-runtime` 的 `SandboxManager.initialize`。

**使用**：

```bash
# 装扩展到全局
cp -R packages/coding-agent/examples/extensions/sandbox ~/.pi/agent/extensions/sandbox
cd ~/.pi/agent/extensions/sandbox && npm install

# 启动（默认启用沙箱）
pi

# 启动时禁用沙箱
pi --no-sandbox

# 在会话中查看当前沙箱配置
/sandbox
```

**关键限制**：
- **仅 macOS / Linux**（其他平台会被自动禁用并提示）
- **只覆盖 bash 工具和 `!` 命令**（不覆盖 `read`/`write`/`edit` 等其他工具）
- Linux 还需安装 `bubblewrap`、`socat`、`ripgrep`
- 是**示例扩展**，不是 pi 内核功能——生产用前自己审代码、自己 fork 维护

## 怎么审查第三方扩展（没有内置命令）

`pi` CLI **没有源码审查命令**（不要写 `pi list --details` 或 `pi inspect`——这些都不存在）。**审查 = 人工看磁盘上的源码**。

### 步骤一：找到扩展磁盘位置

不同 source 类型对应不同路径（来自 package-manager.ts 的 `getManagedNpmInstallPath` 等）：

| source | 位置（user scope） | 位置（project scope） |
|--------|-------------------|---------------------|
| `npm:pkg` | `~/.pi/agent/npm/node_modules/<pkg-name>/` | `<cwd>/.pi/npm/node_modules/<pkg-name>/` |
| `git:github.com/user/repo` | `~/.pi/agent/git/github.com/user/repo/` | `<cwd>/.pi/git/github.com/user/repo/` |
| `/absolute/path` | 原路径（不复制） | 原路径 |
| `./relative/path` | 相对 settings.json 目录解析 | 同 |

### 步骤二：用代码编辑器审查

```bash
# 示例：审查一个 user-scope git 包
cd ~/.pi/agent/git/github.com/<user>/<repo>
code .   # 或你顺手的编辑器

# 重点看：
# - extensions/*.ts —— 看注册了什么工具、订阅了什么事件
# - package.json —— 看 dependencies（有无可疑包）、postinstall 脚本
# - skills/SKILL.md —— 模型可能据此执行操作
```

### 步骤三：在容器内先跑

不放心就在 Docker 容器里装一次、跑一次：

```bash
docker run --rm -it \
  -v "$PWD:/workspace:ro" \
  -v pi-sandbox-home:/root/.pi/agent \
  --entrypoint bash pi-sandbox

# 容器内
pi install npm:<untrusted-pkg>
pi -e npm:<untrusted-pkg>
# ...观察行为，容器销毁后不影响宿主
```

## 为什么 settings.json 没有 sandbox 字段

新手常问：既然 pi 有 `packages` / `extensions` 等字段，为什么没有 `sandbox` 字段？

**答案**：sandbox 是**运行时边界**，不是**配置项**。settings.json 管的是"加载什么资源 / pi 怎么表现"，而"在什么 OS 边界内运行"是由**启动 pi 的命令**（docker run、openshell sandbox create、pi -e ./sandbox 扩展）决定的——这两层正交，**不能**在 settings.json 里写"这个扩展在 sandbox 里运行"。

`PackageSource` 类型定义（`settings-manager.ts`）只支持：

```ts
type PackageSource = string | {
  source: string;
  autoload?: boolean;       // 默认 true；false 时该包不自动加载任何资源，只有显式写了 glob 的 extensions/skills/prompts/themes 才加载
  extensions?: string[];    // glob 过滤
  skills?: string[];
  prompts?: string[];
  themes?: string[];
};
```

**没有** `sandbox` 字段。试图写 `{ "source": "...", "sandbox": {...} }` 会被 schema 忽略或拒绝——pi 不会限制该 package 的权限，仍以主进程权限运行。

> 注：`autoload: false` 是"加载控制"，不是"权限沙箱"。它只决定**装进来哪些资源**（比如只装 skills 不装 extensions，减小被注入的攻击面），一旦某个 extension 被加载进来，它仍以主进程权限运行——隔离还得靠容器/VM。

## 横向：与 project trust 的边界（常被混淆）

[security.md](https://github.com/earendil-works/pi-coding-agent/blob/main/packages/coding-agent/docs/security.md) 第 7 行明确：

> Project trust controls whether pi loads project-local settings, resources, packages, and extensions. It is not a sandbox and it does not restrict what the model can ask tools to do after you start working in a directory.

**翻译**：project trust 控制的是「是否加载项目本地 settings/资源/包/扩展」，**不是沙箱**——一旦你信任并加载，模型让工具干什么**不受 trust 限制**。

| 机制 | 解决什么 | 不解决什么 |
|------|---------|-----------|
| project trust | 防止「clone 一个 repo，pi 自动加载恶意 settings/扩展」 | 加载后的代码执行 |
| sandbox（容器/VM/扩展） | 限制已加载代码的执行边界 | 不防止你信任恶意 repo |
| **两者关系** | **互补不替代**——trust 是"输入守卫"，sandbox 是"执行边界" | — |

→ 关于 project trust 的 API，见 [sdk_doc/13-settings-manager.md §项目信任机制](../sdk_doc/13-settings-manager.md)。

## 常见误解与陷阱

1. **「我在 settings.json 写了 `sandbox: {...}` 就安全了」** ❌ 该字段不存在，写了被忽略或 schema 报错。**正确**：用 Docker / Gondolin / `examples/extensions/sandbox/` 扩展
2. **「这个扩展我装在 `npm:untrusted-xxx` 里，是不是自动隔离？」** ❌ pi 对所有 source 一视同仁，**都装进同一个 node_modules / git 目录，都以主进程权限跑**。要隔离必须**整个 pi 进程进容器**，或装个"工具路由扩展"代理该扩展的工具
3. **「Gondolin 装了 = 所有工具都被隔离」** ❌ 仅内置工具（`read`/`write`/`edit`/`bash`/`grep`/`find`/`ls`）和 `!` 命令被路由。其他扩展注册的工具**仍在宿主**（见 containerization.md）
4. **「project trust 阻止了恶意代码执行」** ❌ trust 只是"加载前确认"。一旦信任加载，扩展/skills 想干什么就干什么
5. **「`pi -e npm:xxx` 试运行所以安全」** ❌ `-e` 只是不写 settings.json，**仍然下载安装、仍然执行**。证据：[I02 CHANGELOG v2.68 P0 #1](../../CHANGELOG.md)、`resource-loader.ts` 把 `-e` 的路径标记为 `temporary: true`，但 `temporary` scope 仍然走完整安装链路
6. **「OpenShell 一定比 Docker 安全」** ⚠️ 不绝对。OpenShell 远端 gateway 不 bind-mount 宿主（更隔离），但近端 gateway 仍可能让本地文件暴露——看 gateway 配置
7. **「`examples/extensions/sandbox/` 扩展覆盖所有工具」** ❌ 只覆盖 `bash` 工具和 `user_bash` 事件。`read`/`write`/`edit`/`grep`/`find`/`ls` 不受其沙箱约束（那是 Gondolin 的覆盖范围）。源码 `examples/extensions/sandbox/index.ts` 只 override `bash` 工具的 `execute`
8. **「`pi list --details` 能看源码」** ❌ 该命令不存在。`pi list` usage 是 `pi list [--approve|--no-approve]`，无 `--details` flag、无 source 参数。审查源码要**直接看磁盘路径**（见上文「怎么审查第三方扩展」）
9. **「在 Docker 里挂 `~/.pi/agent` 就万事大吉」** ❌ 反而**把宿主 API key 暴露给容器**。除非你明确要共享凭证，否则用 named volume（`-v pi-agent-home:/root/.pi/agent`）
10. **「bind mount `$PWD:/workspace` 是只读的」** ❌ 默认读写。容器内的写入直接落宿主。要只读加 `:ro` 后缀，工作产出用 `docker cp` 拷出
11. **「macOS 用户能用 bubblewrap 沙箱」** ❌ `examples/extensions/sandbox/` 在 macOS 用 `sandbox-exec`、Linux 用 `bubblewrap`。Windows / 其他平台**直接禁用**并提示 `Sandbox not supported on ${platform}`（源码 `sandbox/index.ts`）
12. **「容器内的 settings.json 改动会写回宿主」** ⚠️ 取决于挂载。挂 named volume 不会；bind mount `~/.pi/agent` 会。CI/自动化要警惕**容器内对配置的修改影响宿主**

## 变体与延伸

- 扩展分发与安装 → 见 [场景 I01](I01-pi-package.md)、[场景 I02](I02-distribute-extension.md)
- 扩展基础开发（扩展能干什么） → 见 [场景 E02](E02-extension-basics.md)
- 拦截/确认工具调用（如 `protected-paths.ts`） → 见 [场景 D04](D04-confirm-destructive.md)
- 项目信任机制 → 见 [sdk_doc/13-settings-manager.md §项目信任机制](../sdk_doc/13-settings-manager.md)
