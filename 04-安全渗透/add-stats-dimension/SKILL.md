---
name: add-stats-dimension
description: Checklist for adding or changing a statistics table/dimension in the collector. Use whenever touching SQLite schema, traffic write paths, retention, ClickHouse parity, or stats query APIs — several places must change together or data silently drifts.
---

# Add / Change a Stats Dimension

The stats pipeline has one write implementation but several coupled registries. Missing any step below produces silent data loss or drift — none of it fails loudly. Work through the checklist in order.

## 1. Schema (single source of truth)

- Add the `CREATE TABLE` / `CREATE INDEX` statements to `apps/collector/src/database/schema.ts` (`SCHEMA` + `INDEXES`). Nothing else creates tables.
- Existing databases only get `IF NOT EXISTS` DDL from schema.ts. Column additions and index drops need an explicit migration in `apps/collector/src/modules/db/db.ts` `init()` (see the `ALTER TABLE ... ADD COLUMN` and `DROP INDEX IF EXISTS` precedents there).
- Do NOT add expression indexes on `(total_download + total_upload)` — hot queries filter by `backend_id` first, so the planner never uses them (verified with EXPLAIN QUERY PLAN); they only amplify writes.

## 2. Write path (exactly one)

- All traffic writes go through `batchUpdateTrafficStats` in `apps/collector/src/database/repositories/traffic-writer.repository.ts`. `updateTrafficStats` is a thin wrapper over it — never add a second write implementation.
- Add your aggregation map in the batch loop and the upsert in the appropriate transaction group. The whole flush is one outer transaction; keep it that way (a partial commit + buffer retry double-counts).
- CSV-style association columns (`ips`, `rules`, `chains`, `domains`) must dedupe with the delimiter-aware pattern — bare `INSTR(list, value)` treats `a.com` as present when `aa.com` is:

```sql
INSTR(',' || col || ',', ',' || @value || ',') > 0
```

- Respect `reduceWrites` semantics: detail tables are skipped when ClickHouse is the read source (`shouldReduceSqliteWrites` in `modules/stats/stats-write-mode.ts`). Decide whether your table is "core" (always written) or "detail" (skippable) and place it in the matching branch.

## 3. Rule naming contract (if the dimension involves rules)

- Rule stats key on the **top-level policy group** (last chain hop) for multi-hop chains — `buildRuleName` in `src/shared/utils/rule-name.ts` is the only place this logic lives. The frontend matches these names against gateway rule targets (`rule.proxy`), so changing the key breaks the flow graph and rule matching (this regressed once in v1.3.9; see `docs/dev/deep-review-2026-07.md`).

## 4. Retention

- Register cleanup in `apps/collector/src/modules/cleanup/cleanup.service.ts` + `config.repository.ts`. Minute-level tables follow `connectionLogsDays` (default 7d); hourly tables follow `hourlyStatsDays` (default 30d). `hourly_dim_stats` backs all >6h range queries — never tie it to the minute cutoff.
- Cumulative `*_stats` tables currently have no retention; if your table is a cross-product (rule×ip style), consider a `last_seen`-based cap from day one.

## 5. Backend deletion

- Add the table to the explicit delete list in `deleteBackendData` (`apps/collector/src/database/repositories/backend.repository.ts`). `PRAGMA foreign_keys` is OFF, so the declared `ON DELETE CASCADE` never fires — forgetting this leaves orphan rows forever.

## 6. ClickHouse parity (if the data is queryable over ranges)

- Writer: `apps/collector/src/modules/clickhouse/clickhouse.writer.ts`
- Reader: `apps/collector/src/modules/clickhouse/clickhouse.reader.ts` — the SQLite and CH implementations of the same query must return the same shape and semantics (time format, windows). Routing lives in `modules/stats/stats.service.ts` (`*WithRouting` methods).

## 7. Types and API

- Shared response types go in `packages/shared` (both apps consume them; never fork the type locally).
- Time keys are UTC ISO slices: minute `YYYY-MM-DDTHH:MM:00`, hour `YYYY-MM-DDTHH:00:00` (`toMinuteKey`/`toHourKey` in `base.repository.ts`). Don't introduce local-time keys.

## 8. Tests

- Add a Vitest case in `apps/collector` exercising the batch write + read-back (see `traffic-writer.test.ts` for patterns, `src/__tests__/helpers.ts` for DB setup), then run the `verify-changes` skill.
