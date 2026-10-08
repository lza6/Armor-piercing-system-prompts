---
name: ai-ide-strategy-writing
description: "AI-IDE 写策略与执行 — 用 AI 生成 Qlib 量化策略代码、Docker 容器执行策略/回测、自然语言条件选股、策略落库。在 QuantBot / Claude Code 中让 AI 写策略、生成 Qlib 策略、执行策略回测、云端保存策略时使用。触发词：AI写策略、写策略、生成策略、策略代码、AI-IDE、运行策略、执行策略、帮我写个策略、云端策略"
---

> ## ⚙️ 运行环境契约（最高优先级，先于本文其余内容执行）
>
> 本技能可能运行在 **QuantBot（QwenPaw 容器）** 或**宿主机/本地 Claude Code**。执行前先探测环境（`which docker`、API 连通性），并遵守以下映射规则：
>
> 1. **后端 API 地址**：QwenPaw / 容器网络内一律用 `http://quantmind:8000`（`quantmind` 是 docker 网络别名）；仅宿主机调试用 `http://127.0.0.1:8000`。正文中出现的 `127.0.0.1:8000`、`localhost:800x`，在 QwenPaw 环境下自动替换为 `http://quantmind:8000`。
> 2. **取数脚本执行**：凡 import 了 `pandas / duckdb / psycopg2 / numpy / sqlalchemy` 等重依赖或 `backend` 包的脚本，**必须在 quantmind 容器内执行**（QwenPaw 本地 venv 无这些依赖）：
>    ```bash
>    docker cp <脚本路径> quantmind:/tmp/<脚本名> && docker exec -w /app quantmind python3 /tmp/<脚本名> <参数>
>    ```
>    脚本源三选一：宿主机 repo `skills/<name>/scripts/`、QwenPaw 工作区 `/app/working/workspaces/default/skills/<name>/scripts/`、挂载目录 `/quantmind/skills/<name>/scripts/`。纯标准库脚本（无重依赖）可在 QwenPaw 本地直接跑。
> 3. **报告落盘**：股票报告页可见的 MD/PDF 报告，直接写 `/data/reports/trading_agents/{市场或类别}/{股票名}/`（QwenPaw 对 `/app/db` 有写权限，**直接写文件，不要 docker cp**）；过程数据 facts 写 `/data/reports/<类别>/`（`/data` 可写）。
> 4. **MD → PDF 转换（按优先级降级）**：
>    ① `docker exec -w /app quantmind python3 backend/scripts/md_to_pdf_report.py <输入.md> <输出.pdf>`（研报级排版，首选）；
>    ② docker 不可用时，**改用 QwenPaw 内置 `pdf` 技能**把 MD 转成 PDF；
>    ③ 两者都不可用则只交付 MD，并明确告知用户 PDF 未能生成及原因。
> 5. 本文中的 `~/.claude`、`cp -r ... ~/.claude/skills` 等说明仅适用于本地 Claude Code 维护者，**QuantBot 不要执行**。

# AI-IDE 写策略技能

AI-IDE 用 AI 生成 Qlib 量化策略代码，并在 **Docker 容器**中隔离执行（策略运行 / 回测）。核心入口是 `/api/v1/ai-ide/execute/*`。

## 认证

```bash
BASE=http://127.0.0.1:8000
TOKEN=$(curl -s -X POST $BASE/api/v1/auth/login -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123","tenant_id":"default"}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin).get('access_token',''))")
AUTH="Authorization: Bearer $TOKEN"
CT="Content-Type: application/json"
```

## 1. 运行策略 / 回测（核心）

AI-IDE 通过 Docker 容器执行策略代码。策略分三种模式自动识别：
1. **可执行脚本**（有 `if __name__ == '__main__'` 或顶层可执行代码）
2. **main() / run() 函数**
3. **模块型策略**（`STRATEGY_CONFIG` / `get_strategy_config()`）→ 回测中心兼容模式

### 1.1 启动执行（本地文件或临时代码）
```bash
# 从存储加载策略文件执行（云端策略）
curl -s -X POST -H "$AUTH" -H "$CT" "$BASE/api/v1/ai-ide/execute/start" \
  -d '{
    "file_id": "strategy_id_xxx",
    "model_id": "mdl_cn_ensemble_xxx",        # 可选：指定回测模型（默认模型）
    "strategy_id": "strategy_xxx",
    "run_id": "run_xxx",
    "qlib_provider_uri": "db/qlib_data",      # 多市场 Qlib 数据路径
    "qlib_region": "cn",                      # cn / us / hk / crypto / futures
    "benchmark": "SH000300"
  }'

# 直接传代码执行（未保存代码）
curl -s -X POST -H "$AUTH" -H "$CT" "$BASE/api/v1/ai-ide/execute/run-tmp" \
  -d '{
    "content": "print(1+1)",
    "filename": "tmp_strategy.py",
    "model_id": "mdl_cn_ensemble_xxx"
  }'
# 返回: {job_id, status: "started", runner_image}
```

### 1.2 流式查看日志 / 结果
```bash
# SSE 流式日志（[SYSTEM]/[RESULT]/[ERROR]/[PROCESS_FINISHED]）
curl -s -N -H "$AUTH" "$BASE/api/v1/ai-ide/execute/logs/{job_id}"
```
**日志关键标记**：
- `[SYSTEM] 使用回测中心同一回测引擎执行` — 模块型策略进入回测
- `[RESULT] annual_return: ... / sharpe_ratio: ...` — 回测结果
- `[ERROR] ...` — 执行错误
- `[PROCESS_FINISHED]` — 执行结束

### 1.3 停止执行
```bash
curl -s -X POST -H "$AUTH" "$BASE/api/v1/ai-ide/execute/stop/{job_id}"
```

### 1.4 语法检查
```bash
curl -s -X POST -H "$AUTH" -H "$CT" "$BASE/api/v1/ai-ide/execute/check-syntax" \
  -d '{"code": "class Strategy: ..."}'
```

## 2. 回测默认模型

AI-IDE 回测使用**默认模型**（`/api/v1/models/default`），回测前自动检查模型 pred 就绪：
- **单模型**：读模型目录 `pred.pkl` / `pred.parquet`；缺失时提示"请先对该模型执行推理生成预测"
- **融合模型**（`ensemble_config.json`）：自动用子模型 pred 融合生成 `pred.pkl`（截面排名百分位加权）
- **向量化极速回测**：纯 TopK 策略（topk=n_drop、无加权/无止损/无自定义类）自动走向量化引擎（秒级），复杂策略保真走 step 模式

## 3. 策略代码规范

模块型策略（回测中心兼容模式）需定义：
- **`STRATEGY_CONFIG`**：dict，含 `class` + `kwargs`（signal/topk/n_drop/rebalance_days 等）
- **或 `get_strategy_config()`**：返回上述 dict
- 支持 `POOL_FILE` 顶部变量指定股票池文件

**策略类参考**（`backend/services/engine/qlib_app/utils/extended_strategies.py` + `recording_strategy.py`）：
| 类 | 用途 |
|---|---|
| `RedisTopkStrategy` | Top-K 选股（等权） |
| `RedisMomentumStrategy` | 动量策略（momentum_period 累计收益 + 模型分融合） |
| `RedisRiskGuardTopkStrategy` | 大盘风控 Top-K（f_* 过滤 + 动态降仓） |
| `RedisWeightStrategy` | 分数加权（max_weight/min_score） |
| `RedisStopLossStrategy` | 止损止盈 |
| `RedisLongShortTopkStrategy` | 多空 Top-K |
| `RedisVolatilityWeightedStrategy` | 波动率加权 |
| `RedisRecordingStrategy` | 通用（f_* 基本面过滤 + 动态仓位） |

## 4. 策略云端落库

```bash
# 保存策略
curl -s -X POST -H "$AUTH" -H "$CT" "$BASE/api/v1/strategies" \
  -d '{
    "name": "AI生成-低估值策略",
    "description": "PE<15 且 ROE>10",
    "strategy_type": "CustomStrategy",
    "params": {"topk": 20, "n_drop": 5},
    "code": "STRATEGY_CONFIG = {...}",
    "market": "CN"
  }'

# 策略列表 / 模板
curl -s -H "$AUTH" "$BASE/api/v1/strategies"
curl -s -H "$AUTH" "$BASE/api/v1/strategies/templates"
```

## 5. 实战流程（推荐）

当用户要求"写个策略 / 运行策略"时：
1. **理解需求**：确认选股条件（市值/PE/ROE/动量/行业等）和市场
2. **生成策略代码**：构造 `STRATEGY_CONFIG`（模块型）或可执行脚本
3. **保存落库**：`/strategies` POST 保存到云端
4. **执行**：`/ai-ide/execute/start`（带 model_id + qlib_provider_uri）
5. **看结果**：`/ai-ide/execute/logs/{job_id}` SSE 流式，等 `[PROCESS_FINISHED]`
6. **回测深挖**：用 [[backtest-center]] 技能对策略跑完整回测
7. **调优迭代**：根据回测结果调参数，重新生成

## 6. 相关技能

- **[[backtest-center]]** — 完整回测中心（对比/优化/深度分析）
- **[[smart-strategy-stock-picking]]** — 条件选股生成股票池
- **[[quantmind-operations]]** — 模型训练/推理/数据更新

## 7. 常见问题

| 现象 | 处理 |
|---|---|
| 模块型策略无法执行 | 确保有 `STRATEGY_CONFIG` 或 `get_strategy_config()` |
| 回测报"模型无可用预测文件" | 单模型先推理；融合模型会自动生成 pred |
| 回测慢 | 纯 TopK 策略自动向量化；复杂策略耗时正常 |
| 容器执行失败 | 查 `/execute/logs/{job_id}` 的错误详情 |
| 想换市场回测 | 传 `qlib_provider_uri` + `qlib_region`（多市场 Qlib 数据） |
