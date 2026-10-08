# 可用的接口与目录（只读诊断视角）

所有路径都基于**数据目录** `DATA_DIR`：

```
macOS: ~/Library/Application Support/omni-studio.kunpengtalk.com/<channel>     # channel: dev | canary | stable
可用 OMNI_DATA_DIR 覆盖；OMNI_DB_PATH 单独覆盖数据库；OMNI_CONTROL_SOCKET 覆盖 socket
```

| 路径 | 内容 |
| --- | --- |
| `DATA_DIR/omni-studio.db` | SQLite（WAL）。只读打开安全：`new Database(path, {readonly:true})` |
| `DATA_DIR/logs/app.log` | **统一日志**（+ `app-<时间戳>.log` 轮转、`db-migrate-error.log`） |
| `DATA_DIR/models/` | 已下载模型（含 `.part` 等中间文件） |
| `DATA_DIR/engines/` | `mflux/`（MLX 生图）、`paddleocr/`、`whispercpp/`、`audiocpp/`、`tessdata/` |
| `DATA_DIR/images/` | 聊天图、生成图、OCR 页图、TTS 音频（媒体服务按此目录的指纹识别实例） |
| `DATA_DIR/uploads/` | 用户上传的原始文件 |
| `DATA_DIR/backups/` | `*.omnibackup` 备份归档（`omi backup inspect` 可只读预览） |
| `DATA_DIR/mlx-downloads/` | MLX 模型下载进度状态（`<id>.json`） |
| `DATA_DIR/db-backups/` | 迁移前自动备份的库（保留 3 份） |
| `DATA_DIR/omni-control.sock` | 控制通道（文件存在 ≈ 应用在运行） |

## 1. 诊断采集脚本（首选入口）

```bash
bun run --cwd apps/studio scripts/omni-diag.ts [--json] [--logs N]
```

只读；应用没运行也能用。源码 `apps/studio/scripts/omni-diag.ts`。

## 2. `omi` CLI（人/Agent 都能用）

安装：`cd apps/studio && bun link`（暴露 `~/.bun/bin/omi`）。
应用在跑 → 走控制 socket；没跑 → 只读命令直接读库/文件兜底。

| 命令 | 用途 |
| --- | --- |
| `omi status` | 服务器 / 网关状态 + 最后错误（应用没跑时会探推理端口 `/health`） |
| `omi logs [--level --source --event --search --limit -v -f --json --path --clear]` | **统一日志**（见 logs.md） |
| `omi server list\|info\|logs\|start\|stop\|restart` | 推理服务器；`logs` 打最后 200 行 |
| `omi models` / `omi model --list` / `omi model-info <name>` | 本地 + 云端模型清单 / 详情 |
| `omi cloud --list` | 云端 provider 配置 |
| `omi memory stats\|list\|search\|export` | 共享记忆（只读子集） |
| `omi backup list\|inspect` / `omi backup remote list\|test` | 备份清单 / 只读预览（不依赖应用与迁移层） |
| `omi benchmark --list` | 历史测速记录（含 error） |
| `omi install` | 检查 llama.cpp / vLLM / SGLang / MLX 依赖 |
| `omi guide [--json]` | 完整手册（数据源 `src/shared/cli-docs.ts`） |
| `omi version` / `omi update` | 版本 / 检查更新 |
| `omni doctor --json`（旧 CLI） | 旧的一体化体检：数据目录、库、引擎二进制、服务器、网关、云端配置 |

## 3. 控制 socket（外部进程的编程接口）

HTTP over unix socket，**只接受 POST**，body 是 `{"cmd": "...", "payload": {...}}`，
响应 `{"ok": true, "data": ...}` 或 `{"ok": false, "error": "..."}`。权限 0600，只在 loopback 上。

```bash
curl --unix-socket "$DATA_DIR/omni-control.sock" -X POST http://control \
  -H 'content-type: application/json' -d '{"cmd":"status"}'
```

只读命令：

| cmd | 返回 |
| --- | --- |
| `ping` | `{version, pid}`（socket 活着 = 应用在跑） |
| `status` | `{server:{status,pid,host,port,engine,logs(末 8000 字符),error?}, gateway, mode, version}` |
| `logs` | `{entries, path, info}`，payload 同 RPC `getAppLogs` 的过滤参数 |
| `logsPath` | `{dir, path, memoryEntries, files}` |
| `serverLogs` | `{logs}`（活动模型完整日志） |
| `launchCommand` | 复现服务器启动命令（`{command, engine}`） |
| `checkBinary` | 引擎二进制是否就位 |
| `downloadsList` | `{tasks}`（状态 / 进度 / error / retries） |
| `gatewayStatus` | 网关状态 + apiKey |
| `models` | `{installed[], cloud, cloudProvider, mode, activePath}` |
| `getSettings` | 全部设置（**含密钥，别外传**） |
| `cloudProviders` | 云端 provider 全表 |
| `memoryStats` / `memoryList` / `memorySearch` / `memoryPending` / `memoryExport` | 记忆 |
| `benchmarkRun` / `benchmarkRecords` | 测速 |
| 写操作 | `serverStart/Stop/Restart`、`downloadsResume/Start`、`gatewayStart/Stop/Restart`、`logsClear`、`serverClearLogs`、`setActiveModel`、`updateSettings`、`activate`、`navigate`、`memory*` 写口 |

## 4. RPC（主进程 ↔ webview）

契约在 `apps/studio/src/bun/rpc/index.ts` 的 `AppRPC`（约 380 个请求方法，扁平命名）。
**外部进程无法直接调用**（它是 webview 的传输层）—— 外部诊断走控制 socket / CLI。
排查代码时按前缀找方法名：`getServerStatus` `listServedModels` `getServedModelLogs`
`getAppLogs` `listImageRecords` `listVideoRecords` `listVoiceRecords` `getOcrStatus`
`listDownloads` `listNotifications` `kbIndexStats` `memoryEvents` `backupInspect` 等。
新增的日志相关方法：`getAppLogs` / `getAppLogInfo` / `clearAppLogs` / `writeAppLog`。

## 5. 媒体服务（图片 / 音频预览）

固定端口 `127.0.0.1:19782`（`src/shared/server-info.ts` 的常量，所有进程共用）。

| 路由 | 内容 |
| --- | --- |
| `GET /__omni/media-id` | `{id, pid}`，`id` = 图片目录路径的 sha1 前 12 位。**用来判断端口占用者是谁** |
| `/prompt-library/<rel>` | 提示词库素材（本地仓库 → 本地缓存） |
| `/artifact/<id>/...` | Agent 产出物预览 |
| `/workspace/<rootId>/...` | 工作区文件预览 |
| `/<rel>` | `DATA_DIR/images` 下的聊天图 / 生成图 / OCR 页图 / TTS 音频 |

状态三态：`serving`（本实例）/ `shared`（同数据的另一实例，无害）/ `blocked`（**另一个数据目录**
—— 预览会失败或串数据）。`blocked` 时顶栏告警 + 通知，并每 5 秒重试接管。

## 6. 数据库（只读）

> 用 `readonly: true` 打开；WAL 下并发读安全。表结构随版本演进，**查询失败就用
> `PRAGMA table_info(<表>)` 先看列**，别假设列名（`documents` 没有 `name`，用 `path`）。

| 表 | 排障用途 |
| --- | --- |
| `settings` | 全部设置（key/value）；`MODEL_DOWNLOADS` 是下载任务 JSON |
| `conversations` / `messages` | 会话与消息（`app` 列区分 chat / agent / voicecall） |
| `agent_events` | `kind='error'` 的行 = Agent 错误事件；`tool_start/tool_end` 看工具轨迹 |
| `agent_todos` / `agent_artifacts` / `agent_permissions` | 待办 / 产出物 / 权限规则 |
| `image_records` | 生图记录：`status done\|failed`、`backend`、`model`、`prompt`、`error` |
| `video_records` | 生视频：`status processing\|done\|failed`、`task_id`、`video_path`、`error` |
| `voice_records` | TTS/ASR/克隆：`kind`、`audio_path`（失败行通常不写，看 audio_path 是否为空） |
| `documents` / `pages` | 文档解析：`status`、逐页 `pages.error` |
| `benchmark_records` | 测速：`status done\|cancelled\|error`、`error` |
| `automation_runs` | 自动化：`status failed`、`error`、`started_at/finished_at` |
| `kb_*`（bases / docs / chunks / ingest_jobs / events） | 知识库摄取与索引 |
| `memories` / `memory_events` / `memory_metrics` | 记忆与检索指标 |
| `skills` / `skill_targets` / `skill_audit_log` | 技能与同步目标、审计（上限 500 行） |
| `mcp_servers` / `cloud_providers` | MCP 与云端 provider 配置 |

常用查询：

```sql
-- 最近失败的生图
select id, backend, model, prompt, error, datetime(created_at/1000,'unixepoch','localtime') at
from image_records where status='failed' order by id desc limit 5;

-- Agent 最近错误
select id, conversation_id, tool_name, output, datetime(created_at/1000,'unixepoch','localtime') at
from agent_events where kind='error' order by id desc limit 5;

-- 下载任务状态
select value from settings where key='MODEL_DOWNLOADS';
```

## 7. 日志文件与健康端点

| 入口 | 说明 |
| --- | --- |
| `GET http://<SERVER_HOST>:<SERVER_PORT>/health` | 推理服务器健康检查（llama/vLLM/SGLang；MLX 用 `/v1/models`） |
| `GET http://<GATEWAY_HOST>:<GATEWAY_PORT>/health` | 网关健康（`{status, gateway, auth, upstream, cloud}`） |
| `DATA_DIR/logs/app.log` | 统一日志 |
| `omni doctor --json` | 旧 CLI 体检（含 exit code：有失败项返回 1） |
