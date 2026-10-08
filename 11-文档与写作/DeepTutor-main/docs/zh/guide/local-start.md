# 本地启动

DeepTutor 现在只有一套官方本机引导流程。

## 推荐流程

1. `git clone` 项目。
2. 激活 Python 3.11+ 环境。
3. 从源码安装：

```bash
pip install -e .
pip install ".[server]"
pip install ".[math-animator]"   # 可选
```

4. 运行：

```bash
python scripts/start_tour.py
```

引导会直接在终端里运行，并先询问你想使用 `Web` 还是 `CLI`。

两种模式最终都会写入项目根目录 `.env`。运行行为配置只从下面这些文件读取：

- `data/user/settings/main.yaml`
- `data/user/settings/agents.yaml`
- `data/user/settings/interface.json`

## 引导会完成什么

- 选择 DeepTutor 后续一直使用的前端端口和后端端口
- 安装所选档位的依赖
- 收集 LLM、Embedding、可选 Search 配置
- 在最终写入前测试 LLM 和 Embedding
- 写入 `.env`，并清理可能残留的临时草稿 `data/user/settings/env.json`

## 引导完成后启动应用

```bash
python scripts/start_web.py
```

这个启动器会读取项目根目录 `.env`，启动 FastAPI 后端，并自动把 `NEXT_PUBLIC_API_BASE` 注入前端进程。

## 仅使用 CLI

如果你只想使用终端工作流，在 `start_tour.py` 里选择 `CLI` 即可。之后常用命令仍然是：

```bash
deeptutor chat
deeptutor kb list
deeptutor serve --port 8001
```

## 说明

- `config/` 已不再参与运行时加载。
- `web/.env.local` 不再是官方本机配置流程的一部分。
- TTS 设置已不再属于官方配置面。
