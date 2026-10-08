# A 类静态 Token 指标

- 工具: count_tokens.py v0.1  配置: `benchmarks/naval/token-config-v2.json` (sha256:adfa3d60ff2f0378)
- 静态加载模型见 `scripts/count_tokens.py` docstring；不代表宿主实际计费。

## tokenizer = `cl100k_base`

| 版本 | 入口数 | 发现负载(常驻) | 单任务负载 min/median/max | 语料总量 |
|---|---|---|---|---|
| atomic_pack_baseline | 19 | 3897 | 2070/2207/2345 | 254521 |
| single_compiled | 1 | 340 | 4396/4534/4650 | 48669 |
| compact_pack_compiled | 7 | 1575 | 路由 4623/4761/4877；晋级 2190/2271/2354 | 62558 |

## tokenizer = `o200k_base`

| 版本 | 入口数 | 发现负载(常驻) | 单任务负载 min/median/max | 语料总量 |
|---|---|---|---|---|
| atomic_pack_baseline | 19 | 2682 | 1531/1616/1704 | 185162 |
| single_compiled | 1 | 222 | 3290/3380/3450 | 35299 |
| compact_pack_compiled | 7 | 1092 | 路由 3459/3549/3619；晋级 1612/1669/1699 | 45468 |
