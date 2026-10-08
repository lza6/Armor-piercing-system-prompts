# A 类静态 Token 指标

- 工具: count_tokens.py v0.1  配置: `benchmarks/naval/token-config.json` (sha256:b83ec08c8fae168a)
- 静态加载模型见 `scripts/count_tokens.py` docstring；不代表宿主实际计费。

## tokenizer = `cl100k_base`

| 版本 | 入口数 | 发现负载(常驻) | 单任务负载 min/median/max | 语料总量 |
|---|---|---|---|---|
| A-original-pack | 19 | 3897 | 2070/2207/2345 | 113417 |
| B-single | 1 | 340 | 4568/4730/4844 | 52599 |
| C-compact-pack | 7 | 1575 | 路由 4795/4957/5071；晋级 2157/2247/2345 | 68799 |

## tokenizer = `o200k_base`

| 版本 | 入口数 | 发现负载(常驻) | 单任务负载 min/median/max | 语料总量 |
|---|---|---|---|---|
| A-original-pack | 19 | 2682 | 1531/1616/1704 | 82859 |
| B-single | 1 | 222 | 3407/3503/3594 | 38008 |
| C-compact-pack | 7 | 1092 | 路由 3576/3672/3763；晋级 1571/1633/1674 | 50053 |
