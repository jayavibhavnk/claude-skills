---
name: postgres-optimization
description: Optimize PostgreSQL queries - indexes, query plans, connection pooling, and performance tuning.
metadata:
  priority: 8
  docs:
    - "https://www.postgresql.org/docs/current/performance Tips.html"
  pathPatterns:
    - "**/postgres/**"
    - "**/postgresql/**"
  bashPatterns:
    - '\bpostgres\b'
    - '\bpostgresql\b'
  promptSignals:
    phrases:
      - "postgres"
      - "postgresql"
      - "database"
    anyOf:
      - "query"
      - "optimize"
      - "index"
      - "performance"
---

## PostgreSQL Optimization

### Understanding EXPLAIN

```sql
-- Analyze query plan
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.*, o.*
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01';

-- Key terms in EXPLAIN output
-- Seq Scan - Full table scan (often bad for large tables)
-- Index Scan - Using an index (good)
-- Bitmap Heap Scan - Index + filtering (ok)
-- Nested Loop - Row-by-row (ok for small sets)
-- Hash Join - Building hash table (good for large)
-- Merge Join - Sorted inputs (good for sorted)
```

### Index Strategies

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index (only index some rows)
CREATE INDEX idx_orders_pending
ON orders(created_at)
WHERE status = 'pending';

-- Covering index (includes all needed columns)
CREATE INDEX idx_users_name_covering
ON users(name)
INCLUDE (email, created_at);

-- Index on expression
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- GIN index for JSON
CREATE INDEX idx_products_attrs ON products USING GIN(attributes);
```

### Query Patterns

```sql
-- BAD: SELECT *
SELECT * FROM orders WHERE id = 123;

-- GOOD: Specific columns
SELECT id, status, total FROM orders WHERE id = 123;

-- BAD: Implicit type conversion
SELECT * FROM users WHERE id = '123';  -- string vs int

-- GOOD: Explicit types
SELECT * FROM users WHERE id = 123;

-- BAD: Functions on indexed columns
SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- GOOD: Range query on index
SELECT * FROM orders
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### Connection Pooling

```sql
-- pgBouncer configuration (pgbouncer.ini)
[databases]
mydb = host=localhost dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
```

### Autovacuum Tuning

```sql
-- Per-table vacuum settings
ALTER TABLE orders SET (
  autovacuum_vacuum_scale_factor = 0.01,  -- 1% of tuples
  autovacuum_analyze_scale_factor = 0.005,
  autovacuum_vacuum_cost_delay = 2ms
);

-- Monitor bloat
SELECT schemaname, tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Useful Queries

```sql
-- Find slow queries (requires pg_stat_statements)
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 20;

-- Find missing indexes on a table
SELECT
  schemaname,
  tablename,
  seq_scan,
  seq_tup_read,
  idx_scan,
  idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan * 10  -- Many more seq scans than index scans
ORDER BY seq_scan DESC;

-- Table statistics
SELECT
  relname,
  n_live_tup AS row_count,
  n_dead_tup AS dead_rows,
  last_vacuum,
  last_autovacuum,
  last_analyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

### Partitioning

```sql
-- Range partitioning by date
CREATE TABLE orders (
  id BIGSERIAL,
  created_at TIMESTAMP NOT NULL,
  total DECIMAL(10,2)
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE orders_2024_01 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- Index on partitioned table
CREATE INDEX idx_orders_date ON orders(created_at);

-- Automatic partition creation (use a function + trigger)
```

### Performance Checklist

- [ ] Use EXPLAIN ANALYZE on slow queries
- [ ] Add indexes for WHERE, JOIN, ORDER BY columns
- [ ] Avoid SELECT * - specify needed columns
- [ ] Use connection pooling for high-traffic apps
- [ ] Tune autovacuum for your workload
- [ ] Partition large tables by date/range
- [ ] Monitor and recreate bloated indexes
- [ ] Use prepared statements for repeated queries
- [ ] Keep statistics fresh (ANALYZE)
- [ ] Consider read replicas for SELECT queries
