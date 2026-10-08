# 运行预检与故障恢复

完整制作第一次进入配音或长渲染前读取本页；同项目仅改文案/颜色且环境未变时复用已有检查。提示词模式不执行环境检查。

## 用执行结果选环境

不按默认 `python`、包可被搜索到、文件夹存在来判断能力。先选已验证的独立环境；运行库升级、模型路径变动或环境被清理后重新实测。机器路径放在本地配置（Windows 可放 `%LOCALAPPDATA%/CodexVideo/runtime.json`），不要写进可分发技能或工程 ZIP。配置不是成功证明。

`scripts/preflight.py` 只做本地检查，不读密钥、不调用配音、不安装依赖。配置字段：`node`、`hyperframes_cli`（CLI 的 JS 入口绝对路径）、`chrome`、`ffmpeg`、`ffprobe`、`media_python`；独立语音复核另加 `asr_python`、`asr_model`（实际模型快照目录）。按当前机器发现文件并填入，不复制别人的路径。

逐笔故事模式另外运行 `scripts/stroke_story_preflight.py`。它检查仓库内置的 `vendor/srt-whiteboard-animation`、隔离 Python、`skeleton`/`contour-wipe` 参数，并在临时目录完成一个无网络的小型 MP4 渲染与解码。它不会安装依赖或调用收费接口；具体命令见 [stroke-story-workflow.md](stroke-story-workflow.md)。

```text
python <skill>/scripts/preflight.py --config <local-runtime.json> --mode render --report <project>/preflight-render.json
python <skill>/scripts/preflight.py --config <local-runtime.json> --mode asr --audio <existing-local-audio> --report <project>/preflight-asr.json
```

渲染检查会真实编码三帧 H.264、探测生成文件、通过当前 CLI 依赖启动 Chrome 并截图。ASR 检查会先核实非空模型权重、配置和词表，再在独立子进程中实际加载模型并推理；提供已有音频时只用前五秒。不能把导入成功或空模型缓存当成可运行。

本机本次验证通过的是 Python 3.12 独立环境、faster-whisper 1.2.1 / CTranslate2 4.8.1；完整依赖组合见 `scripts/asr-requirements-tested.txt`。这是可复现组合，不是所有机器都必须升级到该版本。建立独立环境时依照安装技能，不改系统 PATH 或其他项目环境。先用已有音频测试，不为预检重新收费配音。

## 选择当前发行版支持的检查工具

优先运行当前 HyperFrames CLI 的 `check` / `inspect` / `snapshot`。部分技能附带的动画映射脚本直接依赖开发包 `@hyperframes/producer`，CLI 发行版不一定提供这个独立包；不能把“CLI 能运行”推导成“所有附带脚本依赖齐全”。

需要细粒度动画映射且工程是单页、内联 SVG/HTML/GSAP 时，可使用本技能的兼容入口（参数均取本地配置）：

```text
node <skill>/scripts/animation-map.mjs <project> --cli <hyperframes-cli-entry> --browser <chrome-executable> --width 1080 --height 1920 --frames 2 --out <project>/animation-map
```

它沿用原映射逻辑，通过 CLI 所在依赖树发现 Puppeteer，不要求单独安装开发包，也不硬编码 npm 缓存目录。此入口不模拟 HyperFrames 嵌套合成/视频同步，遇到外部子合成或视频会明确拒绝；改用官方 CLI 检查与实际 MP4 验证，不把缩减版映射冒充完整引擎检查。

## 按失败类型恢复

- DLL 初始化失败、访问违规：记录解释器和依赖版本，将该组合标记为本次不可用。切换经预检的隔离环境；无任何变化不再重跑同一命令。仅凭另一个环境成功，不能断言具体哪个 DLL 或单个版本就是根因。
- 模型只有目录、权重缺失：不启动加载。选择已完整可用的模型；确需下载时先核实体积、磁盘和网络，不为了可选加强检查自动下载数 GB。
- 检查脚本缺开发依赖：使用上述兼容入口或当前 CLI 的公开能力；不临时修改全局插件文件。
- 浏览器检查超时：保存阶段和日志，先查看进程是否仍有进展；确认停滞后终止本次子进程，允许切换 `--no-browser-gpu` 后做短片/截图测试。软件模式不是普遍更快，不强制套用。
- UTF-8 中文文件被系统 GBK 解码：文件读写显式指定 UTF-8，必要时对子进程设置 `PYTHONUTF8=1`，不修改系统区域设置。
- 布局错误：修画面，保留已通过的音轨。固定眉题、角标尽量放在场景外共享层；转场标题叠化与正常场景文本冲突分别判断，禁止批量忽略碰撞。

预检失败只阻止依赖该能力的阶段。原生有效词时间戳可用于字幕对齐；独立 ASR 是额外复核，失败不等于配音失败，但必须披露复核缺口，不能改用静音分析后宣称已逐词验收。没有工具可以证明语气自然或英文读音完全正确时，保留待试听状态。

## 长渲染之前与之后

顺序为：确定最终静态构图 → 代表场景、长字幕和转场检查 → 完整工程检查通过 → 长渲染。不要一边仍改布局、一边先启动全片渲染。用 3–5 秒内部样段覆盖人物动作与至少一次转场，确认有画、有声后继续全片；不重复向用户索要已确认风格。

最终仍需实际 MP4 全解码、逐场景/转场抽检与结尾核对。修复工具、代码返回 0、HTML 通过检查，都不等于成片已经验收。保存实际工具路径、版本、预检结果和已成功阶段，下一次从记录继续，但在环境改变后重新验证。
