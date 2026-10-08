# Understanding Database Indexing: A Practical Guide

Database indexes are data structures that improve the speed of data retrieval operations on a database table at the cost of additional storage space and slower writes.

## How Indexes Work

An index creates a separate data structure (typically a B-tree or B+ tree) that maintains a sorted reference to the data in your table. Without an index, the database must perform a full table scan, reading every row to find matches.

### B-Tree Index Structure

A B-tree index organizes data into a balanced tree where:

- The root node points to intermediate nodes
- Intermediate nodes point to leaf nodes
- Leaf nodes contain the indexed values and pointers to the actual rows
- All leaf nodes are at the same depth, ensuring consistent lookup time

```
        [50]
       /    \
    [20,35]  [65,80]
    / | \    / | \
  [10][25][40][55][70][90]
   |   |   |   |   |   |
  rows rows rows rows rows rows
```

Lookup complexity: **O(log n)** vs **O(n)** for full scan.

## Types of Indexes

### Single-Column Index

```sql
CREATE INDEX idx_users_email ON users (email);

-- Speeds up:
SELECT * FROM users WHERE email = 'alice@example.com';
```

### Composite (Multi-Column) Index

```sql
CREATE INDEX idx_orders_user_date ON orders (user_id, created_at);

-- Speeds up (left-prefix rule):
SELECT * FROM orders WHERE user_id = 42;
SELECT * FROM orders WHERE user_id = 42 AND created_at > '2024-01-01';

-- Does NOT speed up:
SELECT * FROM orders WHERE created_at > '2024-01-01'; -- missing left prefix
```

### Unique Index

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);
-- Enforces uniqueness AND speeds up lookups
```

### Partial (Filtered) Index

```sql
-- PostgreSQL
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';

-- Only indexes pending orders, much smaller than full index
```

### Covering Index

```sql
CREATE INDEX idx_users_covering ON users (email) INCLUDE (name, created_at);

-- This query is answered entirely from the index (no table lookup):
SELECT name, created_at FROM users WHERE email = 'alice@example.com';
```

### Full-Text Index

```sql
-- PostgreSQL
CREATE INDEX idx_articles_search ON articles
USING GIN (to_tsvector('english', title || ' ' || body));

SELECT * FROM articles
WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('database & indexing');
```

## Index Selection Guidelines

| Scenario | Index Type | Example |
|----------|-----------|---------|
| Exact lookups | B-tree | WHERE email = ? |
| Range queries | B-tree | WHERE created_at > ? |
| Text search | GIN/GiST | WHERE body @@ ? |
| JSON fields | GIN | WHERE data @> '{"key": "val"}' |
| Geospatial | GiST/SP-GiST | WHERE ST_DWithin(point, ...) |
| Low-cardinality | Bitmap (auto) | WHERE status IN ('active', 'pending') |

## Query Plan Analysis

Use `EXPLAIN ANALYZE` to verify index usage:

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';

-- Good: Index Scan
-- Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=128)
--   Index Cond: (email = 'alice@example.com'::text)
--   Actual time: 0.023..0.024 rows=1 loops=1
-- Planning Time: 0.089 ms
-- Execution Time: 0.045 ms

-- Bad: Sequential Scan (index not used)
-- Seq Scan on users  (cost=0.00..124.50 rows=1 width=128)
--   Filter: (email = 'alice@example.com'::text)
--   Rows Removed by Filter: 4999
--   Actual time: 2.145..2.145 rows=1 loops=1
-- Planning Time: 0.065 ms
-- Execution Time: 2.178 ms
```

## Common Indexing Mistakes

### 1. Over-Indexing

Every index slows down INSERT, UPDATE, and DELETE operations because the index must be updated too.

| Table Size | Indexes | INSERT Time | Index Overhead |
|-----------|---------|-------------|----------------|
| 1M rows | 2 | 0.3ms | +0.1ms |
| 1M rows | 5 | 0.3ms | +0.4ms |
| 1M rows | 10 | 0.3ms | +1.2ms |
| 1M rows | 20 | 0.3ms | +3.5ms |

### 2. Indexing Low-Cardinality Columns

```sql
-- Bad: boolean column has only 2 values
CREATE INDEX idx_users_active ON users (is_active);
-- The optimizer will likely ignore this for large tables
```

### 3. Not Using Composite Index Order Correctly

The left-prefix rule means the order matters:

```sql
-- Index: (a, b, c)
WHERE a = 1                  -- Uses index
WHERE a = 1 AND b = 2       -- Uses index
WHERE a = 1 AND b = 2 AND c = 3  -- Uses index
WHERE b = 2                  -- Does NOT use index
WHERE b = 2 AND c = 3       -- Does NOT use index
WHERE a = 1 AND c = 3       -- Partially uses index (only 'a')
```

### 4. Functions on Indexed Columns

```sql
-- Bad: function prevents index usage
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Fix: create an expression index
CREATE INDEX idx_users_email_lower ON users (LOWER(email));
```

## Monitoring Index Health

```sql
-- PostgreSQL: Find unused indexes
SELECT
  schemaname || '.' || relname AS table,
  indexrelname AS index,
  pg_size_pretty(pg_relation_size(i.indexrelid)) AS size,
  idx_scan AS scans
FROM pg_stat_user_indexes i
JOIN pg_index USING (indexrelid)
WHERE idx_scan = 0
AND NOT indisunique
ORDER BY pg_relation_size(i.indexrelid) DESC;
```

```sql
-- Find missing indexes (slow queries without index)
SELECT
  query,
  calls,
  mean_exec_time,
  total_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY total_exec_time DESC
LIMIT 20;
```

## Summary

- Indexes trade write speed and storage for faster reads
- B-tree indexes handle most use cases (equality and range queries)
- Composite indexes follow the left-prefix rule
- Use `EXPLAIN ANALYZE` to verify index usage
- Monitor for unused indexes and remove them
- Only index columns that appear in WHERE, JOIN, or ORDER BY clauses

## References

- [PostgreSQL Index Documentation](https://www.postgresql.org/docs/current/indexes.html)
- [Use The Index, Luke](https://use-the-index-luke.com/)
- [MySQL Index Best Practices](https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html)
