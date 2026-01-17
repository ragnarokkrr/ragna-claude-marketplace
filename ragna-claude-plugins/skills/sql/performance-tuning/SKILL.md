---
name: performance-tuning
description: Optimize database performance including query plan caching, parameter sniffing, partitioning, compression, materialized views, and resource management
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Performance Tuning Skill

This skill provides advanced database performance optimization techniques including caching, partitioning, compression, and resource management.

## Purpose

Optimize database performance through:
- **Query Plan Caching** - Efficient plan reuse
- **Parameter Sniffing** - Handling variable query patterns
- **Partitioning** - Large table management
- **Compression** - Storage and I/O optimization
- **Resource Management** - CPU, memory, I/O tuning

## Instructions

When this skill is invoked:

1. **Identify performance bottleneck:**
   - Query execution time
   - Resource contention
   - Storage/I/O issues
   - Memory pressure

2. **Determine database platform:**
   - PostgreSQL (default)
   - MySQL / MariaDB
   - SQL Server
   - Oracle

3. **Apply appropriate optimization:**
   - Choose technique based on bottleneck
   - Consider trade-offs
   - Plan validation

4. **Provide monitoring guidance**

## Query Plan Caching

### PostgreSQL Prepared Statements

```sql
-- Prepared statement for plan caching
PREPARE get_customer_orders(INT) AS
SELECT o.order_id, o.order_date, o.total_amount
FROM orders o
WHERE o.customer_id = $1
ORDER BY o.order_date DESC
LIMIT 10;

-- Execute with different parameters
EXECUTE get_customer_orders(123);
EXECUTE get_customer_orders(456);

-- Check cached plans
SELECT * FROM pg_prepared_statements;

-- Deallocate when done
DEALLOCATE get_customer_orders;
```

### SQL Server Plan Cache Management

```sql
-- View plan cache
SELECT
    cp.objtype,
    cp.cacheobjtype,
    cp.size_in_bytes,
    cp.usecounts,
    cp.refcounts,
    qt.text AS query_text,
    qp.query_plan
FROM sys.dm_exec_cached_plans cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) qt
CROSS APPLY sys.dm_exec_query_plan(cp.plan_handle) qp
WHERE qt.text LIKE '%orders%'
ORDER BY cp.usecounts DESC;

-- Clear specific plan
DBCC FREEPROCCACHE(plan_handle);

-- Clear all plans (use carefully!)
DBCC FREEPROCCACHE;

-- Force parameterization
ALTER DATABASE MyDB SET PARAMETERIZATION FORCED;
```

### Query Store (SQL Server)

```sql
-- Enable Query Store
ALTER DATABASE MyDB SET QUERY_STORE = ON;
ALTER DATABASE MyDB SET QUERY_STORE (
    OPERATION_MODE = READ_WRITE,
    MAX_STORAGE_SIZE_MB = 1000,
    DATA_FLUSH_INTERVAL_SECONDS = 900,
    INTERVAL_LENGTH_MINUTES = 60,
    QUERY_CAPTURE_MODE = AUTO
);

-- Find regressed queries
SELECT
    q.query_id,
    qt.query_sql_text,
    rs.avg_duration,
    rs.count_executions
FROM sys.query_store_query q
JOIN sys.query_store_query_text qt ON q.query_text_id = qt.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
ORDER BY rs.avg_duration DESC;

-- Force specific plan
EXEC sp_query_store_force_plan @query_id = 1, @plan_id = 1;
```

## Parameter Sniffing

### Detecting Parameter Sniffing Issues

```sql
-- SQL Server: Query with variable execution times
-- Good for parameter A, slow for parameter B
SELECT * FROM orders WHERE customer_id = @customer_id;

-- Check for sniffing
SELECT
    qs.execution_count,
    qs.total_elapsed_time / qs.execution_count AS avg_duration,
    qs.min_elapsed_time,
    qs.max_elapsed_time,
    qs.plan_generation_num,
    qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
WHERE qt.text LIKE '%customer_id%'
ORDER BY qs.max_elapsed_time DESC;
```

### Solutions for Parameter Sniffing

```sql
-- Solution 1: OPTIMIZE FOR hint
SELECT * FROM orders WHERE customer_id = @customer_id
OPTION (OPTIMIZE FOR (@customer_id = 1));  -- Optimize for typical value

-- Solution 2: OPTIMIZE FOR UNKNOWN
SELECT * FROM orders WHERE customer_id = @customer_id
OPTION (OPTIMIZE FOR UNKNOWN);  -- Generic plan

-- Solution 3: RECOMPILE
SELECT * FROM orders WHERE customer_id = @customer_id
OPTION (RECOMPILE);  -- Fresh plan each time

-- Solution 4: Local variables (prevents sniffing)
DECLARE @local_customer_id INT = @customer_id;
SELECT * FROM orders WHERE customer_id = @local_customer_id;

-- Solution 5: Plan guides
EXEC sp_create_plan_guide
    @name = 'Guide_CustomerOrders',
    @stmt = 'SELECT * FROM orders WHERE customer_id = @customer_id',
    @type = 'SQL',
    @hints = 'OPTION (OPTIMIZE FOR UNKNOWN)';
```

## Table Partitioning

### Range Partitioning (by Date)

```sql
-- PostgreSQL: Declarative partitioning
CREATE TABLE orders_partitioned (
    order_id BIGSERIAL,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMERIC(12,2),
    status VARCHAR(20),
    PRIMARY KEY (order_id, order_date)  -- Partition key must be in PK
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE orders_2024_q3 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

CREATE TABLE orders_2024_q4 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2024-10-01') TO ('2025-01-01');

-- Create default partition for unmatched data
CREATE TABLE orders_default PARTITION OF orders_partitioned DEFAULT;

-- Create indexes on partitions (inherits to all partitions)
CREATE INDEX idx_orders_part_customer ON orders_partitioned(customer_id);
```

### List Partitioning (by Category)

```sql
-- PostgreSQL: List partitioning
CREATE TABLE products_partitioned (
    product_id SERIAL,
    product_name VARCHAR(255),
    category VARCHAR(50) NOT NULL,
    price NUMERIC(10,2),
    PRIMARY KEY (product_id, category)
) PARTITION BY LIST (category);

CREATE TABLE products_electronics PARTITION OF products_partitioned
    FOR VALUES IN ('Electronics', 'Computers', 'Phones');

CREATE TABLE products_clothing PARTITION OF products_partitioned
    FOR VALUES IN ('Clothing', 'Shoes', 'Accessories');

CREATE TABLE products_home PARTITION OF products_partitioned
    FOR VALUES IN ('Furniture', 'Kitchen', 'Garden');
```

### Hash Partitioning (for even distribution)

```sql
-- PostgreSQL: Hash partitioning
CREATE TABLE events_partitioned (
    event_id BIGSERIAL,
    user_id INT NOT NULL,
    event_type VARCHAR(50),
    created_at TIMESTAMP,
    PRIMARY KEY (event_id, user_id)
) PARTITION BY HASH (user_id);

-- Create 8 hash partitions
CREATE TABLE events_p0 PARTITION OF events_partitioned
    FOR VALUES WITH (MODULUS 8, REMAINDER 0);
CREATE TABLE events_p1 PARTITION OF events_partitioned
    FOR VALUES WITH (MODULUS 8, REMAINDER 1);
-- ... continue for p2 through p7
```

### Partition Maintenance

```sql
-- Add new partition
CREATE TABLE orders_2025_q1 PARTITION OF orders_partitioned
    FOR VALUES FROM ('2025-01-01') TO ('2025-04-01');

-- Detach partition for archival
ALTER TABLE orders_partitioned DETACH PARTITION orders_2023_q1;

-- Drop old partition
DROP TABLE orders_2022_q1;

-- Partition switching (SQL Server) - instant data movement
ALTER TABLE staging_orders
SWITCH TO orders_partitioned PARTITION 5;
```

### Partition Pruning Verification

```sql
-- PostgreSQL: Check partition pruning
EXPLAIN (ANALYZE, COSTS)
SELECT * FROM orders_partitioned
WHERE order_date BETWEEN '2024-01-01' AND '2024-03-31';

-- Should show only orders_2024_q1 being scanned
-- Not scanning other partitions = successful pruning
```

## Data Compression

### PostgreSQL Compression

```sql
-- TOAST compression (automatic for large values)
-- Use bytea for binary data with LZ compression

-- Extension for table compression
CREATE EXTENSION cstore_fdw;  -- Columnar storage

-- Native BRIN index (block range) for sorted data
CREATE INDEX idx_events_brin ON events USING BRIN (created_at);
```

### SQL Server Compression

```sql
-- Row compression (good for fixed-length types)
ALTER TABLE orders
REBUILD WITH (DATA_COMPRESSION = ROW);

-- Page compression (better ratio, more CPU)
ALTER TABLE orders
REBUILD WITH (DATA_COMPRESSION = PAGE);

-- Columnstore compression (best for analytics)
CREATE CLUSTERED COLUMNSTORE INDEX idx_orders_cs ON orders_archive;

-- Check compression savings
EXEC sp_estimate_data_compression_savings
    @schema_name = 'dbo',
    @object_name = 'orders',
    @index_id = NULL,
    @partition_number = NULL,
    @data_compression = 'PAGE';

-- Compression on specific partitions
ALTER TABLE orders
REBUILD PARTITION = 1 WITH (DATA_COMPRESSION = PAGE);
```

### MySQL Compression

```sql
-- InnoDB page compression
ALTER TABLE orders ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;

-- Transparent page compression (MySQL 5.7+)
ALTER TABLE orders COMPRESSION='lz4';

-- Check compression ratio
SELECT
    table_name,
    data_length,
    data_free,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS total_mb
FROM information_schema.tables
WHERE table_schema = 'mydb' AND table_name = 'orders';
```

## Materialized Views

### PostgreSQL Materialized Views

```sql
-- Create materialized view for expensive aggregation
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT
    date_trunc('month', order_date) AS month,
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_sales,
    AVG(total_amount) AS avg_order_value
FROM orders
WHERE status = 'completed'
GROUP BY date_trunc('month', order_date), customer_id;

-- Create index on materialized view
CREATE INDEX idx_mv_sales_month ON mv_sales_summary(month);
CREATE INDEX idx_mv_sales_customer ON mv_sales_summary(customer_id);

-- Refresh options
-- Full refresh (blocks reads)
REFRESH MATERIALIZED VIEW mv_sales_summary;

-- Concurrent refresh (requires unique index)
CREATE UNIQUE INDEX idx_mv_sales_unique
ON mv_sales_summary(month, customer_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary;

-- Automated refresh with pg_cron
CREATE EXTENSION pg_cron;
SELECT cron.schedule('0 * * * *',
    'REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary');
```

### SQL Server Indexed Views

```sql
-- Create indexed view (auto-maintained)
CREATE VIEW dbo.vw_sales_summary
WITH SCHEMABINDING  -- Required for indexed view
AS
SELECT
    CAST(order_date AS DATE) AS order_date,
    customer_id,
    COUNT_BIG(*) AS order_count,
    SUM(total_amount) AS total_sales
FROM dbo.orders
GROUP BY CAST(order_date AS DATE), customer_id;

-- Create clustered index (materializes the view)
CREATE UNIQUE CLUSTERED INDEX idx_vw_sales
ON dbo.vw_sales_summary(order_date, customer_id);

-- Query optimizer automatically uses indexed view
-- Or force usage with NOEXPAND
SELECT * FROM dbo.vw_sales_summary WITH (NOEXPAND)
WHERE order_date >= '2024-01-01';
```

## Resource Management

### PostgreSQL Configuration

```sql
-- Memory settings (postgresql.conf or ALTER SYSTEM)
-- shared_buffers: 25% of RAM
ALTER SYSTEM SET shared_buffers = '4GB';

-- work_mem: Memory per operation (sorts, hashes)
ALTER SYSTEM SET work_mem = '256MB';

-- maintenance_work_mem: For VACUUM, CREATE INDEX
ALTER SYSTEM SET maintenance_work_mem = '1GB';

-- effective_cache_size: Hint for planner (50-75% of RAM)
ALTER SYSTEM SET effective_cache_size = '12GB';

-- Parallelism
ALTER SYSTEM SET max_parallel_workers_per_gather = 4;
ALTER SYSTEM SET parallel_tuple_cost = 0.001;

-- Apply changes
SELECT pg_reload_conf();
```

### SQL Server Resource Governor

```sql
-- Create resource pool with limits
CREATE RESOURCE POOL ReportingPool
WITH (
    MIN_CPU_PERCENT = 0,
    MAX_CPU_PERCENT = 30,
    MIN_MEMORY_PERCENT = 0,
    MAX_MEMORY_PERCENT = 25
);

-- Create workload group
CREATE WORKLOAD GROUP ReportingGroup
USING ReportingPool;

-- Classifier function
CREATE FUNCTION dbo.WorkloadClassifier()
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
    IF APP_NAME() LIKE 'Reporting%'
        RETURN 'ReportingGroup';
    RETURN 'default';
END;

-- Apply classifier
ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.WorkloadClassifier);
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

### Connection Pooling

```sql
-- PostgreSQL: PgBouncer configuration
-- [databases]
-- mydb = host=localhost dbname=mydb

-- [pgbouncer]
-- pool_mode = transaction
-- max_client_conn = 1000
-- default_pool_size = 20
-- reserve_pool_size = 5

-- Check pool status
SHOW POOLS;
SHOW CLIENTS;
SHOW SERVERS;
```

## Performance Monitoring

### PostgreSQL Performance Views

```sql
-- Slow queries (pg_stat_statements extension)
CREATE EXTENSION pg_stat_statements;

SELECT
    query,
    calls,
    total_exec_time / 1000 AS total_seconds,
    mean_exec_time / 1000 AS mean_seconds,
    rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Table I/O statistics
SELECT
    schemaname,
    tablename,
    heap_blks_read,
    heap_blks_hit,
    idx_blks_read,
    idx_blks_hit,
    ROUND(100.0 * heap_blks_hit / NULLIF(heap_blks_hit + heap_blks_read, 0), 2) AS heap_hit_ratio
FROM pg_statio_user_tables
ORDER BY heap_blks_read DESC;

-- Index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### SQL Server DMVs

```sql
-- Wait statistics
SELECT
    wait_type,
    waiting_tasks_count,
    wait_time_ms,
    max_wait_time_ms,
    signal_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE wait_type NOT LIKE 'SLEEP%'
    AND wait_type NOT LIKE 'CLR%'
    AND wait_type NOT LIKE 'LAZYWRITER%'
ORDER BY wait_time_ms DESC;

-- I/O statistics by file
SELECT
    DB_NAME(database_id) AS database_name,
    file_id,
    num_of_reads,
    num_of_writes,
    io_stall_read_ms,
    io_stall_write_ms,
    CAST(io_stall_read_ms / NULLIF(num_of_reads, 0) AS DECIMAL(10,2)) AS avg_read_latency_ms
FROM sys.dm_io_virtual_file_stats(NULL, NULL)
ORDER BY io_stall_read_ms + io_stall_write_ms DESC;

-- Memory grants
SELECT
    session_id,
    requested_memory_kb,
    granted_memory_kb,
    used_memory_kb,
    query_cost,
    dop
FROM sys.dm_exec_query_memory_grants
WHERE is_small = 0;
```

## Performance Tuning Checklist

```markdown
## Database Performance Checklist

### Query Level
- [ ] Execution plans analyzed (no table scans on large tables)
- [ ] Indexes covering frequent queries
- [ ] Statistics up to date
- [ ] No implicit conversions in WHERE clauses
- [ ] Parameter sniffing addressed

### Table Level
- [ ] Appropriate data types (not over-sized)
- [ ] Partitioning for large tables (>100M rows)
- [ ] Compression enabled where appropriate
- [ ] Regular maintenance (VACUUM, UPDATE STATISTICS)

### Instance Level
- [ ] Memory settings tuned (shared_buffers, work_mem)
- [ ] Connection pooling configured
- [ ] Parallelism enabled
- [ ] I/O subsystem adequate

### Monitoring
- [ ] Slow query logging enabled
- [ ] Wait statistics reviewed
- [ ] I/O latency acceptable (<10ms)
- [ ] Cache hit ratio >95%
```

## Output Format

When invoked, provide:
1. Performance bottleneck analysis
2. Specific optimization recommendations
3. Implementation scripts (DDL, configuration)
4. Expected improvement metrics
5. Monitoring queries for validation

## Cross-References

- **Query Optimization (query-optimization skill):** Individual query tuning
- **Index Design (index-design skill):** Index strategies
- **Data Warehousing (data-warehousing skill):** Analytics optimization
- **Transaction Management (transaction-management skill):** Concurrency tuning
