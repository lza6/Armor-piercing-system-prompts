# How I Cut Our PostgreSQL Query Times by 90%

*Published April 8, 2026 by Anika Patel*

Last month our main dashboard was taking 12 seconds to load. Users were complaining. Our PM was panicking. After two weeks of focused optimization work, the same dashboard now loads in under 800ms. Here is everything I did.

## The Setup

We run a SaaS analytics platform. The core of it is a PostgreSQL 16 database holding about 400 million event rows across 23 tenants. Each tenant generates between 5 and 80 million rows per month. The schema was designed three years ago when we had two tenants and a dream.

The main offender was the events table:

```sql
CREATE TABLE events (
    id          BIGSERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL,
    event_type  VARCHAR(64) NOT NULL,
    payload     JSONB NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    user_id     INTEGER,
    session_id  UUID,
    processed   BOOLEAN DEFAULT false
);

CREATE INDEX idx_events_tenant ON events (tenant_id);
CREATE INDEX idx_events_created ON events (created_at);
```

Two indexes. For 400 million rows. That was problem number one.

## Step 1: Understanding the Queries

Before touching anything, I turned on `pg_stat_statements` and let it collect data for 48 hours:

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

After two days, I pulled the top offenders:

```sql
SELECT
    calls,
    round(total_exec_time::numeric, 2) AS total_ms,
    round(mean_exec_time::numeric, 2) AS mean_ms,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

The top three queries accounted for 74% of total database time. They all followed the same pattern: filter by tenant, filter by date range, aggregate by event type. The dashboard was running these on every page load.

## Step 2: Composite Indexes

The existing indexes were useless for the actual query patterns. PostgreSQL was doing sequential scans on the full table because no single index covered both `tenant_id` and `created_at`.

```sql
-- Drop the old, unhelpful indexes
DROP INDEX idx_events_tenant;
DROP INDEX idx_events_created;

-- Create composite indexes matching actual query patterns
CREATE INDEX CONCURRENTLY idx_events_tenant_created
    ON events (tenant_id, created_at DESC);

CREATE INDEX CONCURRENTLY idx_events_tenant_type_created
    ON events (tenant_id, event_type, created_at DESC);
```

The `CONCURRENTLY` keyword is critical in production. Without it, PostgreSQL takes an exclusive lock on the table and blocks all writes. With it, the index build takes longer but the table stays fully operational.

After this change alone, the dashboard query dropped from 12 seconds to about 3.5 seconds. Progress, but not enough.

## Step 3: Partitioning

With 400 million rows, even a good index has to traverse a large B-tree. PostgreSQL supports declarative partitioning natively, and our queries always include a date range, so time-based partitioning was the obvious choice.

```sql
-- Create the partitioned table
CREATE TABLE events_partitioned (
    id          BIGSERIAL,
    tenant_id   INTEGER NOT NULL,
    event_type  VARCHAR(64) NOT NULL,
    payload     JSONB NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    user_id     INTEGER,
    session_id  UUID,
    processed   BOOLEAN DEFAULT false
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE events_2026_01 PARTITION OF events_partitioned
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE events_2026_02 PARTITION OF events_partitioned
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
CREATE TABLE events_2026_03 PARTITION OF events_partitioned
    FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
CREATE TABLE events_2026_04 PARTITION OF events_partitioned
    FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
```

I wrote a script to automate creating future partitions. It runs on the first of each month via pg_cron:

```sql
SELECT cron.schedule(
    'create-monthly-partition',
    '0 0 1 * *',
    $$SELECT create_next_month_partition('events_partitioned')$$
);
```

Migrating data from the old table into the partitioned one took careful planning. We used `pg_partman` to handle the initial backfill during a maintenance window.

After partitioning, the dashboard queries were down to about 1.2 seconds. Most queries only touch one or two monthly partitions instead of the entire 400M row table.

## Step 4: Materialized Views for Aggregations

The dashboard always shows the same aggregations: events per type per day, broken down by tenant. These do not need to be real-time. A five-minute lag is acceptable.

```sql
CREATE MATERIALIZED VIEW mv_daily_event_counts AS
SELECT
    tenant_id,
    event_type,
    date_trunc('day', created_at) AS day,
    count(*) AS event_count,
    count(DISTINCT user_id) AS unique_users,
    count(DISTINCT session_id) AS unique_sessions
FROM events_partitioned
GROUP BY tenant_id, event_type, date_trunc('day', created_at);

CREATE UNIQUE INDEX idx_mv_daily_tenant_type_day
    ON mv_daily_event_counts (tenant_id, event_type, day);
```

Refreshing the materialized view concurrently:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_event_counts;
```

This runs every five minutes via pg_cron. The dashboard now reads from the materialized view instead of aggregating raw events. Query time dropped to under 50ms for the aggregation panels.

## Step 5: Connection Pooling

Our application was opening a new database connection for each request. PostgreSQL forks a new process per connection, and we were hitting 300+ simultaneous connections during peak hours. Each one consumes memory and competes for CPU.

We deployed PgBouncer in transaction pooling mode:

```ini
[databases]
analytics = host=127.0.0.1 port=5432 dbname=analytics

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = md5
pool_mode = transaction
max_client_conn = 500
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3
```

This reduced active PostgreSQL connections from 300+ to a stable 25. Memory usage on the database server dropped by 40%.

## Step 6: Query Rewrites

The application code had some genuinely bad query patterns. The worst was a correlated subquery that ran once per row:

```sql
-- Before: correlated subquery (terrible)
SELECT e.*, (
    SELECT count(*)
    FROM events e2
    WHERE e2.tenant_id = e.tenant_id
      AND e2.user_id = e.user_id
      AND e2.created_at < e.created_at
) AS prior_event_count
FROM events e
WHERE e.tenant_id = 5
  AND e.created_at >= '2026-03-01';
```

Rewritten as a window function:

```sql
-- After: window function (fast)
SELECT
    e.*,
    count(*) OVER (
        PARTITION BY e.tenant_id, e.user_id
        ORDER BY e.created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
    ) AS prior_event_count
FROM events_partitioned e
WHERE e.tenant_id = 5
  AND e.created_at >= '2026-03-01';
```

This single rewrite turned a 45-second query into a 200ms query.

## The Results

| Metric               | Before   | After    | Improvement |
|----------------------|----------|----------|-------------|
| Dashboard load time  | 12.1s    | 780ms    | 93%         |
| P95 query latency    | 8.4s     | 320ms    | 96%         |
| DB connections (peak)| 312      | 25       | 92%         |
| DB memory usage      | 14.2 GB  | 8.6 GB   | 39%         |
| Aggregation queries  | 3.2s     | 48ms     | 98%         |

## Lessons Learned

1. **Measure before optimizing.** Without `pg_stat_statements`, I would have guessed wrong about which queries to fix first.

2. **Indexes must match query patterns.** A single-column index on `tenant_id` is useless when every query also filters by `created_at`. Composite indexes in the right order matter enormously.

3. **Partitioning is not free.** It adds complexity to migrations, backups, and schema changes. But for tables above 100M rows with natural time boundaries, the payoff is massive.

4. **Materialized views are underrated.** If your users can tolerate a few minutes of staleness, materialized views eliminate expensive repeated aggregations entirely.

5. **Connection pooling should be the default.** There is almost no reason to let your application open raw connections to PostgreSQL in production.

6. **Read your query plans.** `EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)` is your best friend. Learn to read the output. It tells you exactly where time is being spent.

## Tools I Used

- [pg_stat_statements](https://www.postgresql.org/docs/16/pgstatstatements.html) for query profiling
- [PgBouncer](https://www.pgbouncer.org/) for connection pooling
- [pg_partman](https://github.com/pgpartman/pg_partman) for partition management
- [pg_cron](https://github.com/citusdata/pg_cron) for scheduled maintenance
- [EXPLAIN.dalibo.com](https://explain.dalibo.com/) for visualizing query plans

## What I Would Do Differently

If I were starting this database from scratch, I would partition from day one, set up PgBouncer immediately, and design indexes based on actual query patterns rather than guessing. The cost of retrofitting all of this was much higher than getting it right initially.

The next frontier is read replicas. We are already bottlenecked on write throughput during peak ingestion hours, and streaming replication with read-only replicas for the dashboard would separate read and write workloads cleanly.

*Have questions? Find me on [Mastodon](https://mastodon.example.com/@anikapatel) or [send me an email](mailto:anika@example.com).*
