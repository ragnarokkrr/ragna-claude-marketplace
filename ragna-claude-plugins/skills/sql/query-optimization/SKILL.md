---
name: query-optimization
description: Analyze and optimize SQL queries with execution plan analysis, index selection strategies, and query hints across PostgreSQL, MySQL, SQL Server, and Oracle
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Query Optimization Skill

This skill analyzes SQL queries and provides optimization recommendations including execution plan analysis, index selection strategies, and platform-specific optimization techniques.

## Purpose

Optimize SQL query performance through:
- **Execution Plan Analysis** - Reading and interpreting query plans
- **Index Selection** - Identifying optimal indexes for queries
- **Query Rewriting** - Transforming queries for better performance
- **Platform-Specific Hints** - Leveraging database-specific optimizations

## Instructions

When this skill is invoked:

1. **Identify the database platform:**
   - PostgreSQL (default if not specified)
   - MySQL / MariaDB
   - SQL Server
   - Oracle

2. **Gather query context:**
   - The SQL query to optimize
   - Table sizes and row counts
   - Existing indexes
   - Performance targets (latency, throughput)
   - Current execution time

3. **Analyze execution plan:**
   - Identify expensive operations
   - Detect table scans vs index scans
   - Check cardinality estimates
   - Find implicit conversions
   - Spot missing statistics

4. **Provide optimization recommendations:**
   - Index recommendations
   - Query rewrites
   - Statistics updates
   - Platform-specific hints

## Execution Plan Analysis

### PostgreSQL

```sql
-- Get detailed execution plan
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, COSTS)
SELECT ...;

-- Key metrics to check:
-- - No Sequential Scans on large tables
-- - Index Scans used appropriately
-- - Join algorithm selection (Hash, Merge, Nested Loop)
-- - Shared buffers hit rate
-- - Actual rows vs estimated rows match
```

**Plan Analysis Checklist:**
| Indicator | Good | Bad |
|-----------|------|-----|
| Seq Scan | On small tables (<1000 rows) | On large tables |
| Index Scan | Using covering index | Key lookup required |
| Rows Estimate | Within 10x of actual | Off by 100x+ |
| Buffers Read | Low (cached) | High (disk I/O) |
| Sort | In-memory | On-disk (external sort) |

### SQL Server

```sql
-- Enable actual execution plan
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT ...;

-- Check:
-- - Index Seek vs Index Scan vs Table Scan
-- - Logical reads (should be minimal)
-- - Warnings (implicit conversions, missing indexes)
-- - Actual vs Estimated rows
```

**Key Indicators:**
```sql
-- Find queries with high logical reads
SELECT TOP 20
    qs.execution_count,
    qs.total_logical_reads / qs.execution_count AS avg_logical_reads,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.text)
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2)+1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_logical_reads / qs.execution_count DESC;
```

### MySQL

```sql
-- Analyze query performance
EXPLAIN FORMAT=JSON
SELECT ...;

-- Check execution stats
SELECT * FROM sys.statements_with_full_table_scans
WHERE query LIKE '%your_table%';
```

## Index Selection Strategies

### Covering Index Design

```sql
-- Query to optimize
SELECT customer_id, order_date, total_amount
FROM orders
WHERE status = 'completed'
    AND order_date >= '2024-01-01'
ORDER BY order_date DESC;

-- Covering index design (PostgreSQL)
CREATE INDEX idx_orders_covering
ON orders (status, order_date DESC)
INCLUDE (customer_id, total_amount)
WHERE status = 'completed';

-- SQL Server version
CREATE NONCLUSTERED INDEX idx_orders_covering
ON orders (status, order_date DESC)
INCLUDE (customer_id, total_amount)
WHERE status = 'completed';
```

### Composite Index Ordering

**Principle:** Most selective columns first for equality, range columns last.

```sql
-- Query pattern analysis
-- Query 1: WHERE status = 'X' AND category = 'Y'
-- Query 2: WHERE status = 'X'
-- Query 3: WHERE category = 'Y'

-- Optimal index (most selective first)
-- Assuming status has higher cardinality
CREATE INDEX idx_products_status_category
ON products (status, category);

-- This index helps Query 1 and Query 2
-- Query 3 needs separate index on category alone
CREATE INDEX idx_products_category
ON products (category);
```

### Partial/Filtered Indexes

```sql
-- PostgreSQL partial index for active records only
CREATE INDEX idx_products_active
ON products (category, price)
WHERE is_active = true;

-- SQL Server filtered index
CREATE INDEX idx_products_active
ON products (category, price)
WHERE is_active = 1;

-- Reduces index size dramatically if inactive records are large percentage
```

## Query Optimization Patterns

### Set-Based Operations

```sql
-- BAD: Row-by-row cursor processing
DECLARE @customer_id INT;
DECLARE customer_cursor CURSOR FOR SELECT customer_id FROM customers;
OPEN customer_cursor;
FETCH NEXT FROM customer_cursor INTO @customer_id;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE orders SET status = 'processed' WHERE customer_id = @customer_id;
    FETCH NEXT FROM customer_cursor INTO @customer_id;
END;
CLOSE customer_cursor;
DEALLOCATE customer_cursor;

-- GOOD: Set-based operation
UPDATE o
SET status = 'processed'
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
```

### Early Filtering

```sql
-- BAD: Filtering after joins
SELECT o.order_id, c.customer_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01';

-- GOOD: Filter before join (if selective)
WITH filtered_orders AS (
    SELECT order_id, customer_id
    FROM orders
    WHERE order_date >= '2024-01-01'
)
SELECT fo.order_id, c.customer_name
FROM filtered_orders fo
INNER JOIN customers c ON fo.customer_id = c.customer_id;
```

### EXISTS vs COUNT

```sql
-- BAD: Counting to check existence
IF (SELECT COUNT(*) FROM orders WHERE customer_id = 123) > 0
    -- Do something

-- GOOD: EXISTS stops at first match
IF EXISTS (SELECT 1 FROM orders WHERE customer_id = 123)
    -- Do something
```

### SARGable Predicates

```sql
-- BAD: Non-SARGable (Search ARGument able) predicate
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- GOOD: SARGable predicate (index can be used)
SELECT * FROM orders
WHERE order_date >= '2024-01-01'
    AND order_date < '2025-01-01';

-- BAD: Function on column
SELECT * FROM customers WHERE UPPER(email) = 'USER@EXAMPLE.COM';

-- GOOD: Expression index or application-side normalization
CREATE INDEX idx_customers_lower_email ON customers(LOWER(email));
SELECT * FROM customers WHERE LOWER(email) = 'user@example.com';
```

### Implicit Conversion Avoidance

```sql
-- BAD: Implicit conversion
SELECT * FROM orders WHERE order_id = '12345';  -- order_id is INT

-- GOOD: Explicit type
SELECT * FROM orders WHERE order_id = 12345;
```

## Query Hints (Use Sparingly)

### PostgreSQL

```sql
-- Force index usage (not recommended, optimizer usually knows best)
SET enable_seqscan = off;
SELECT * FROM orders WHERE status = 'completed';
SET enable_seqscan = on;

-- Better: Analyze and let optimizer decide
ANALYZE orders;
```

### SQL Server

```sql
-- Force specific index
SELECT * FROM orders WITH (INDEX(idx_orders_status))
WHERE status = 'completed';

-- Force join type
SELECT * FROM orders o
INNER MERGE JOIN customers c ON o.customer_id = c.customer_id;

-- Query hints
SELECT * FROM orders
OPTION (HASH JOIN, MAXDOP 4);
```

### MySQL

```sql
-- Force index
SELECT * FROM orders FORCE INDEX (idx_orders_status)
WHERE status = 'completed';

-- Ignore index
SELECT * FROM orders IGNORE INDEX (idx_orders_date)
WHERE order_date = '2024-01-01';
```

## Statistics Management

### PostgreSQL

```sql
-- Update statistics
ANALYZE orders;

-- Check statistics age
SELECT
    schemaname,
    tablename,
    last_analyze,
    last_autoanalyze,
    n_mod_since_analyze
FROM pg_stat_user_tables
WHERE n_mod_since_analyze > 10000
ORDER BY n_mod_since_analyze DESC;

-- Cross-column statistics for correlated columns
CREATE STATISTICS stats_orders_customer_status (dependencies)
    ON customer_id, status FROM orders;
```

### SQL Server

```sql
-- Update statistics with full scan
UPDATE STATISTICS orders WITH FULLSCAN;

-- Check statistics age
SELECT
    OBJECT_NAME(object_id) AS table_name,
    name AS stats_name,
    STATS_DATE(object_id, stats_id) AS last_updated,
    modification_counter AS rows_modified
FROM sys.stats
WHERE STATS_DATE(object_id, stats_id) < DATEADD(day, -7, GETDATE());
```

## Optimization Report Template

```markdown
# Query Optimization Analysis

## Query Details
- **Database:** [PostgreSQL/MySQL/SQL Server/Oracle]
- **Table(s):** [table names with row counts]
- **Current Execution Time:** [Xms]
- **Target Execution Time:** [<Yms]

## Execution Plan Analysis
[Paste or describe execution plan]

### Key Findings
1. **Issue 1:** [Description] - Impact: [High/Medium/Low]
2. **Issue 2:** [Description] - Impact: [High/Medium/Low]

## Recommendations

### Index Recommendations
| Priority | Index Definition | Rationale |
|----------|------------------|-----------|
| High | `CREATE INDEX idx_name ON table(col1, col2)` | [Why] |
| Medium | `CREATE INDEX idx_name ON table(col1) INCLUDE (col2)` | [Why] |

### Query Rewrites
**Before:**
```sql
[Original query]
```

**After:**
```sql
[Optimized query]
```

**Expected Improvement:** [X% reduction in execution time]

### Statistics Updates
```sql
-- Recommended statistics updates
ANALYZE table_name;
```

## Validation
- [ ] Test optimized query with EXPLAIN ANALYZE
- [ ] Verify index usage
- [ ] Measure actual performance improvement
- [ ] Check for regression in other queries
```

## Output Format

When invoked, provide:
1. Execution plan analysis with key findings
2. Specific index recommendations with DDL
3. Query rewrites (before/after)
4. Statistics update commands
5. Expected performance improvement
6. Validation checklist

## Cross-References

- **Index Design (index-design skill):** Detailed index strategies
- **Performance Tuning (performance-tuning skill):** Broader performance optimization
- **SQL Query Patterns (sql-query-patterns skill):** Query writing best practices
