# 安装

## 最轻量方式

把仓库目录放到 Codex 的 Skills 目录，并确保文件夹名为 `hand-drawn-explainer-video-nikola`。提示词模式不需要媒体依赖；运行脚本需要 Python 3.10 或更高版本。Windows 若同时安装多个版本，可把下列 `python` 换成 `py -3.12` 或已确认的解释器绝对路径。

```powershell
git clone https://github.com/hi-nikola/hand-drawn-explainer-video-nikola.git "$env:USERPROFILE/.codex/skills/hand-drawn-explainer-video-nikola"
python "$env:USERPROFILE/.codex/skills/hand-drawn-explainer-video-nikola/scripts/setup_check.py"
```

安装检查应至少显示 `skill_files.ok: true` 和 `capabilities.prompt_only: true`。如果目录里只有 `.git`、ZIP 解压报截断、根目录找不到 `SKILL.md`，说明下载并未完成，不能视为已安装；删除或改名这个不完整目录后重新克隆。检查报告中的 `false` 表示相应成片能力还需要配置，不代表提示词模式失效。

也可以克隆到任意目录，再运行仓库中的 `scripts/install.ps1`。安装器不会覆盖已有目录。

## 启用逐笔故事渲染

逐笔后端已经打包，不需要另装 `srt-whiteboard-animation` Skill。首次运行：

```powershell
python vendor/srt-whiteboard-animation/scripts/prepare_env.py
python scripts/stroke_story_preflight.py --report preflight-stroke-story.json
```

脚本在后端目录创建隔离的 `.venv`，不会写入系统 Python。若公司网络不能访问 Python 包源，可手动创建虚拟环境并安装 `requirements-media.txt`，然后向预检传入 `--backend-python`。

核心逐笔渲染不需要 Informative Drawings、Anime2Sketch、PyTorch 等本地神经网络模型或模型权重。它使用仓库内置的分区遮罩、OpenCV 图像处理和 skeleton 路径追踪。已有合格源图或线稿时，不需要 GPU；只有“从文稿生成新插画”时才可能调用外部图像生成服务。

## 完整 MP4 与程序动画

- 最终合成需要 FFmpeg 和 FFprobe；安装后确认两者可从命令行直接调用。
- 程序动画需要 Node.js 20+、Chromium/Chrome/Edge 和 HyperFrames。示例工程会通过 npm 安装 GSAP，不在仓库中复制其发行文件。
- 云配音和图像/视频生成是可选能力，需要各自账号、网络与可能产生的费用。也可以直接提供旁白音频和源图。

首次安装 Python 包、npm 包和 HyperFrames 需要能够访问相应软件包源。公司代理或镜像环境下如安装失败，先处理代理、证书和镜像配置，不要把下载失败误判为 Skill 缺文件。FFmpeg 请从 [FFmpeg 官方下载页](https://ffmpeg.org/download.html) 或操作系统可信的软件包管理器安装；安装后重新运行 `scripts/setup_check.py`，不要把某个编辑器私有目录中的残缺 FFmpeg 当作系统依赖。

默认成片音色为火山引擎刘飞 `zh_male_liufei_uranus_bigtts`。没有授权密钥或合格现成音轨时，Skill 仍能做路线判断、提示词、分镜和部分画面工作，但不能把电脑系统朗读或 edge-tts 自动顶替为正式旁白。

## OpenAI Skill 上传

OpenAI Skills API 支持上传一个目录或单个 ZIP。打包时 ZIP 根目录应直接包含 `SKILL.md`，不要多套一层文件夹。具体接口和限制以 [OpenAI 官方 Skills 文档](https://developers.openai.com/api/reference/python/resources/skills/methods/create) 为准。

安装完成后重新启动或刷新支持 Skills 的客户端，并用 README 中的触发提示词测试。

## 新环境验收

1. 运行 `scripts/setup_check.py`，先确认 Skill 文件完整，再根据目标路线补环境。
2. 只需要提示词时，无需安装媒体依赖。
3. 需要逐笔视频时，创建后端隔离环境并运行 `stroke_story_preflight.py`；预检必须实际生成并解码一个小型 MP4。
4. 需要程序动画时，确认 Node.js、浏览器、FFmpeg/FFprobe 可用，再进入示例工程执行 `npm install` 和 `npm run check`。
5. 需要正式配音时，先用刘飞音色执行 Dry Run；未授权时不发请求、不自动换低质量声音。
6. 最终只有真实 MP4 完整解码并通过音轨、字幕、场景和末帧检查，才能称为完成。
