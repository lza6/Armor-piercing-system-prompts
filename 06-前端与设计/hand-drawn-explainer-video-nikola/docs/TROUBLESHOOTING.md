# 常见问题

## 逐笔预检找不到 Python

先运行 `python vendor/srt-whiteboard-animation/scripts/prepare_env.py`。若使用自建环境，将解释器显式传给 `scripts/stroke_story_preflight.py --backend-python <路径>`。

如果 pip 报代理、证书或“找不到版本”，先确认 Python 3.10+，再检查本机/公司网络的 pip 代理和镜像设置。也可以在可联网环境中按 `requirements-media.txt` 创建虚拟环境；不要把 `.venv` 提交到仓库。

如果日志先连续出现 `ProxyError`，最后才显示 `No matching distribution found`，优先判断为包源/代理不可达，不要据此随意降低依赖版本。修复网络或切换到可信镜像后重新运行准备脚本；已创建但依赖不全的 `.venv` 可以复用，脚本会继续补齐缺失包。

## 手移动太快

降低 `hand-follow` 只能让手更平滑，不会降低落墨速度。增加区域时长、拆分语义区域或简化源图线条；确保主要轮廓、补色和至少 0.5 秒完成停留都能看清。

## 后一区域提前露出

检查区域框、跨岛连通线和 `protectedRegions`。双岛之间保留 6%–10% 留白，禁止背景线跨岛。

## 最后一帧漏色或突然放大

比较幕尾 0.3–0.5 秒和批准源图。只有确认漏色时才启用源图淡入；正常情况下使用 `--source-overlay never`。

## 程序动画缺少 GSAP

进入示例工程运行 `npm install`；`predev`、`precheck` 和 `prerender` 会把 npm 包中的浏览器文件复制到本地 `assets/`。不要把来历不明的压缩脚本提交到仓库。

## HyperFrames 找不到 FFmpeg 或 FFmpeg 无法启动

`setup_check.py` 只把当前命令行可发现的 FFmpeg/FFprobe 视为就绪。若机器已有未加入 PATH 的可运行版本，可在当前任务的本地运行配置中填写绝对路径，或按 HyperFrames 支持的 `HYPERFRAMES_FFMPEG_PATH`、`HYPERFRAMES_FFPROBE_PATH` 临时环境变量传入。不要为了一个项目永久改写系统 PATH。

“找到文件”不等于“编码器可运行”：Windows 返回 `STATUS_DLL_NOT_FOUND` 时通常是该 FFmpeg 构建缺少配套 DLL；精简版 FFmpeg 也可能不能读取 WAV 或编码 AAC。换成完整、可独立运行的 64 位发行版，并分别执行版本检查和短音频/短视频测试后再渲染。

## 有图但无法完成视频

保存源图、旁白、SRT、时间轴和标注，说明缺少的是 FFmpeg、浏览器、账号还是模型。不能把静态图或 SVG 淡入假称为逐笔成片。
