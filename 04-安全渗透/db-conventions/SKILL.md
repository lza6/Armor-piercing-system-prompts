---
name: db-conventions
description: Data-layer invariants for the collector — SQLite patterns, transactions, time keys, backendId isolation, ClickHouse read/write topology. Use when writing queries, touching repositories, or debugging data drift.
---

# Data Layer Conventions (apps/collector)

## Invariants (breaking any of these has bitten us before)

1. **One write path.** All traffic stats go through `batchUpdateTrafficStats` (`database/repositories/traffic-writer.repository.ts`); the whole flush is a single outer transaction (inner `db.transaction()` calls degrade to savepoints). Never commit table groups independently — a mid-flush failure plus buffer retry double-counts.
2. **Rule name = policy group.** For multi-hop chains the aggregation key is the last chain hop (the user's policy group), produced only by `buildRuleName` (`shared/utils/rule-name.ts`). Rule type + payload (`RuleSet(foo)`) is the key only for single-hop DIRECT/REJECT chains. The web flow graph and gateway-rule matching depend on this.
3. **Delimiter-aware CSV dedup.** Comma-list columns dedupe with `INSTR(',' || col || ',', ',' || @v || ',')` — bare `INSTR` substring-matches (`a.com` inside `aa.com`) and silently drops values. Lists are capped at `LENGTH > 4000`.
4. **UTC time keys.** Minute `YYYY-MM-DDTHH:MM:00`, hour `YYYY-MM-DDTHH:00:00`, via `toMinuteKey`/`toHourKey` (`base.repository.ts`). No local-time truncation.
5. **backendId isolation everywhere.** Every stats table, query, and cache key is scoped by `backend_id`. `PRAGMA foreign_keys` is OFF — deleting a backend relies on the explicit table list in `deleteBackendData`; new tables must be added there.
6. **Range routing.** Range queries route by span in `resolveFactTable` (`base.repository.ts`): ≤6h → `minute_dim_stats`, >6h → `hourly_dim_stats`. Retention must respect this: hourly dim tables follow `hourlyStatsDays`, minute tables follow `connectionLogsDays`.

## SQLite usage

- better-sqlite3, synchronous — long scans and big transactions block the event loop shared with WS ingest. Cap query result sets (`LIMIT`), use `Map` merges instead of nested `findIndex`, and prefer prepared statements cached on the instance for hot paths.
- WAL mode with tuned pragmas is set in `modules/db/db.ts` `init()` (env-tunable: `SQLITE_CACHE_MB`, `SQLITE_WAL_AUTOCHECKPOINT_PAGES`, `SQLITE_BUSY_TIMEOUT_MS`).
- Migrations are ad-hoc in `init()` — idempotent (`IF NOT EXISTS` / try-catch ALTER / app_config flags for one-time data migrations, see `database/rule-name-cleanup.ts` as the reference pattern).

## ClickHouse topology

Read/write behavior is controlled by env flags — keep them coherent:

| Flag | Meaning |
|---|---|
| `CH_WRITE_ENABLED` | Write traffic batches to ClickHouse |
| `STATS_QUERY_SOURCE` | `sqlite` (default) / `clickhouse` / `auto` — read routing |
| `CH_ONLY_MODE=1` | Skip SQLite stats writes entirely (requires healthy CH writer) |
| `CH_STRICT_STATS=1` | Fail instead of silently falling back to SQLite reads |

Guard rails already enforced in code (`modules/stats/stats-write-mode.ts`): reduced SQLite writes require reads to actually prefer ClickHouse — don't bypass `shouldReduceSqliteWrites`/`shouldSkipSqliteStatsWrites` with ad-hoc env checks. When adding a range query, implement BOTH the SQLite repository method and the `clickhouse.reader.ts` counterpart with identical semantics, and route via a `*WithRouting` method in `stats.service.ts`.

## Partial-failure contract

`BatchBuffer.flush` returns per-group outcomes (`detailOk`/`aggOk`); the realtime store is only cleared for the groups that durably persisted. When touching flush/clear logic, preserve this contract — it is the data-loss protection added in v1.3.9.
