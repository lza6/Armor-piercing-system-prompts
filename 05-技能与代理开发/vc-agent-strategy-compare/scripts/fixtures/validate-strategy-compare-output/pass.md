# Strategy Comparison

## Strategies Considered

- sequential — one agent, lowest cost
- parallel-subagents — N agents fan-out
- workflow — staged pipeline
- agent-team — named teammates with shared task list

## 7-Signal Table

| Signal | Score |
| --- | --- |
| File independence | 3 |
| Investigation directions | 2 |
| Integration boundary clarity | 3 |
| Blast radius spread | 2 |
| Time sensitivity | 1 |
| Cost tolerance | 2 |
| Coordination overhead | 1 |

## Cost Guards

- If recommended agent count would exceed > 30, downgrade and warn.
- Never auto-spawn more than > 100 agents; require explicit confirmation.
