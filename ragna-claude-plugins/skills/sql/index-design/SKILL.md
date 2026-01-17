---
name: index-design
description: Design optimal database indexes including B-tree, covering, filtered, composite, and full-text indexes for PostgreSQL, MySQL, SQL Server, and Oracle
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Index Design Skill

This skill designs optimal database indexes for query performance across multiple database platforms.

## Purpose

Design and document database indexes for:
- **Query Performance** - Accelerate SELECT operations
- **Constraint Enforcement** - Support UNIQUE and PRIMARY KEY
- **Join Optimization** - Efficient table joins
- **Sorting** - Avoid filesort operations

## Instructions

When this skill is invoked:

1. **Analyze query patterns:**
   - Identify frequently executed queries
   - Note WHERE clauses, JOIN conditions, ORDER BY
   - Consider selectivity of columns

2. **Determine database platform:**
   - PostgreSQL (default)
   - MySQL / MariaDB
   - SQL Server
   - Oracle

3. **Design appropriate indexes:**
   - Choose index type (B-tree, Hash, GIN, etc.)
   - Determine column order for composites
   - Consider covering indexes
   - Evaluate partial/filtered indexes

4. **Document maintenance strategy:**
   - Fragmentation monitoring
   - Rebuild schedules
   - Usage analysis

## Index Types

### B-Tree Indexes (Default)

**Best For:**
- Equality comparisons (=)
- Range queries (<, >, BETWEEN)
- Sorting (ORDER BY)
- LIKE with prefix matching ('abc%')

```sql
-- Standard B-tree index
CREATE INDEX idx_customers_email ON customers(email);

-- Descending order for recent-first queries
CREATE INDEX idx_orders_date_desc ON orders(order_date DESC);

-- Expression index
CREATE INDEX idx_customers_lower_email ON customers(LOWER(email));
```

### Composite Indexes

**Column Order Rules:**
1. Equality columns first (WHERE col = value)
2. Range columns next (WHERE col > value)
3. ORDER BY columns last
4. Within equality columns, most selective first

```sql
-- Query: WHERE status = 'active' AND category = 'electronics' ORDER BY created_at DESC
-- Optimal index:
CREATE INDEX idx_products_status_category_created
ON products(status, category, created_at DESC);

-- This index supports:
-- WHERE status = 'active'
-- WHERE status = 'active' AND category = 'electronics'
-- WHERE status = 'active' ORDER BY category
-- WHERE status = 'active' AND category = 'electronics' ORDER BY created_at DESC

-- Does NOT efficiently support:
-- WHERE category = 'electronics' (status must be first)
-- WHERE created_at > '2024-01-01' (leftmost columns not filtered)
```

### Covering Indexes

Include all columns needed by query to avoid table lookup (key lookup/bookmark lookup).

```sql
-- Query to cover
SELECT customer_id, order_date, total_amount, status
FROM orders
WHERE customer_id = 123
ORDER BY order_date DESC;

-- PostgreSQL covering index (INCLUDE clause)
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id, order_date DESC)
INCLUDE (total_amount, status);

-- SQL Server covering index
CREATE NONCLUSTERED INDEX idx_orders_customer_covering
ON orders(customer_id, order_date DESC)
INCLUDE (total_amount, status);

-- MySQL: Add columns to index keys (no INCLUDE)
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id, order_date DESC, total_amount, status);
```

### Partial/Filtered Indexes

Index only a subset of rows - smaller size, faster maintenance.

```sql
-- PostgreSQL partial index
CREATE INDEX idx_orders_pending ON orders(customer_id, created_at)
WHERE status = 'pending';

CREATE INDEX idx_products_active ON products(category, price)
WHERE is_active = true;

-- SQL Server filtered index
CREATE INDEX idx_orders_pending ON orders(customer_id, created_at)
WHERE status = 'pending';

-- Useful when:
-- - Frequent queries filter on same condition
-- - Large portion of table excluded by filter
-- - Filter condition is stable (not frequently updated)
```

### Hash Indexes

Exact equality matches only - no range queries.

```sql
-- PostgreSQL hash index
CREATE INDEX idx_sessions_token ON sessions USING HASH (session_token);

-- Good for: WHERE session_token = 'abc123'
-- Bad for: WHERE session_token LIKE 'abc%' (no range support)
```

### GIN Indexes (PostgreSQL)

Generalized Inverted Index - for composite values.

```sql
-- Full-text search
CREATE INDEX idx_products_search
ON products USING GIN (to_tsvector('english', name || ' ' || description));

-- JSONB queries
CREATE INDEX idx_users_preferences ON users USING GIN (preferences);
-- Supports: WHERE preferences @> '{"theme": "dark"}'

-- Array containment
CREATE INDEX idx_posts_tags ON posts USING GIN (tags);
-- Supports: WHERE tags @> ARRAY['sql', 'optimization']

-- Trigram search (fuzzy matching)
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_customers_name_trgm ON customers USING GIN (name gin_trgm_ops);
-- Supports: WHERE name ILIKE '%smith%'
```

### GiST Indexes (PostgreSQL)

Generalized Search Tree - for geometric and range types.

```sql
-- Range types
CREATE INDEX idx_reservations_period ON reservations USING GIST (date_range);
-- Supports: WHERE date_range && '[2024-01-01, 2024-01-31]'::daterange

-- Full-text search (alternative to GIN)
CREATE INDEX idx_articles_content ON articles USING GIST (to_tsvector('english', content));

-- Geospatial
CREATE INDEX idx_locations_point ON locations USING GIST (coordinates);
```

### Columnstore Indexes (SQL Server)

For analytical workloads - high compression, columnar storage.

```sql
-- Clustered columnstore (replaces row storage)
CREATE CLUSTERED COLUMNSTORE INDEX idx_sales_history
ON sales_history;

-- Nonclustered columnstore (additional index)
CREATE NONCLUSTERED COLUMNSTORE INDEX idx_orders_analytics
ON orders(order_date, customer_id, product_id, quantity, amount);
```

## Index Design Patterns

### Foreign Key Indexes

Always index foreign key columns for join performance.

```sql
-- orders.customer_id references customers.customer_id
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- order_items.order_id references orders.order_id
CREATE INDEX idx_order_items_order ON order_items(order_id);

-- order_items.product_id references products.product_id
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

### Multi-Column Selectivity Analysis

```sql
-- PostgreSQL: Check column selectivity
SELECT
    attname AS column_name,
    n_distinct,
    correlation
FROM pg_stats
WHERE tablename = 'orders'
    AND attname IN ('status', 'customer_id', 'order_date');

-- Higher n_distinct = more selective = better for first position
-- Example:
-- customer_id: n_distinct = 50000 (high selectivity)
-- status: n_distinct = 5 (low selectivity)
-- Order: (customer_id, status) not (status, customer_id)
```

### Query Pattern Documentation

```markdown
| Query Pattern | Frequency | Columns | Index Recommendation |
|---------------|-----------|---------|---------------------|
| Get customer orders | 1000/min | customer_id, order_date | `(customer_id, order_date DESC)` |
| Search by status | 500/min | status, created_at | `(status, created_at DESC)` |
| Product lookup | 2000/min | sku | `(sku)` UNIQUE |
| Active products | 300/min | is_active, category | `(category) WHERE is_active` |
```

## Platform-Specific Features

### PostgreSQL

```sql
-- Concurrent index creation (no table lock)
CREATE INDEX CONCURRENTLY idx_orders_date ON orders(order_date);

-- Expression indexes
CREATE INDEX idx_orders_year_month ON orders(
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
);

-- Collation-specific index
CREATE INDEX idx_names_ci ON customers(name COLLATE "en_US.UTF-8");

-- BRIN index for naturally ordered data
CREATE INDEX idx_events_timestamp ON events USING BRIN (created_at);
```

### MySQL / MariaDB

```sql
-- Invisible indexes (for testing removal)
CREATE INDEX idx_test ON orders(status) INVISIBLE;
ALTER TABLE orders ALTER INDEX idx_test VISIBLE;

-- Descending indexes (MySQL 8.0+)
CREATE INDEX idx_orders_date ON orders(order_date DESC);

-- Prefix index for long strings
CREATE INDEX idx_content_prefix ON articles(content(100));

-- Full-text index
CREATE FULLTEXT INDEX idx_products_search ON products(name, description);
```

### SQL Server

```sql
-- Online index creation
CREATE INDEX idx_orders_date ON orders(order_date)
WITH (ONLINE = ON);

-- Included columns (covering)
CREATE INDEX idx_orders_customer ON orders(customer_id)
INCLUDE (order_date, total_amount, status);

-- Compression
CREATE INDEX idx_orders_compressed ON orders(order_date)
WITH (DATA_COMPRESSION = PAGE);

-- Filtered + online
CREATE INDEX idx_active_orders ON orders(customer_id, order_date)
WHERE status = 'active'
WITH (ONLINE = ON, FILLFACTOR = 90);
```

### Oracle

```sql
-- Bitmap index (for low cardinality)
CREATE BITMAP INDEX idx_orders_status ON orders(status);

-- Function-based index
CREATE INDEX idx_orders_year ON orders(EXTRACT(YEAR FROM order_date));

-- Invisible index
CREATE INDEX idx_test ON orders(status) INVISIBLE;

-- Compressed index
CREATE INDEX idx_orders_composite ON orders(status, category, region)
COMPRESS 2;  -- Compress first 2 columns
```

## Index Maintenance

### Fragmentation Monitoring

```sql
-- PostgreSQL: Check index bloat
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- SQL Server: Check fragmentation
SELECT
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count,
    CASE
        WHEN ips.avg_fragmentation_in_percent > 30 THEN 'REBUILD'
        WHEN ips.avg_fragmentation_in_percent > 10 THEN 'REORGANIZE'
        ELSE 'OK'
    END AS recommendation
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
    AND ips.page_count > 100
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

### Index Usage Analysis

```sql
-- PostgreSQL: Find unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
    AND indexrelname NOT LIKE 'pg_%'
ORDER BY pg_relation_size(indexrelid) DESC;

-- SQL Server: Find unused indexes
SELECT
    OBJECT_NAME(i.object_id) AS table_name,
    i.name AS index_name,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE OBJECTPROPERTY(i.object_id, 'IsUserTable') = 1
    AND i.index_id > 0
    AND (ius.user_seeks = 0 OR ius.user_seeks IS NULL)
    AND (ius.user_scans = 0 OR ius.user_scans IS NULL)
ORDER BY OBJECT_NAME(i.object_id), i.name;
```

### Index Rebuild/Reorganize

```sql
-- PostgreSQL: Rebuild index
REINDEX INDEX idx_orders_date;
REINDEX TABLE orders;

-- Concurrent rebuild (no lock)
REINDEX INDEX CONCURRENTLY idx_orders_date;

-- SQL Server: Rebuild
ALTER INDEX idx_orders_date ON orders REBUILD;
ALTER INDEX ALL ON orders REBUILD;

-- SQL Server: Reorganize (less resource intensive)
ALTER INDEX idx_orders_date ON orders REORGANIZE;

-- MySQL: Rebuild
ALTER TABLE orders DROP INDEX idx_orders_date, ADD INDEX idx_orders_date(order_date);
-- Or
OPTIMIZE TABLE orders;
```

## Index Design Template

```markdown
# Index Design for [Table Name]

## Table Profile
- **Row Count:** [X rows]
- **Growth Rate:** [Y rows/day]
- **Read/Write Ratio:** [80/20]

## Query Patterns

| Query | Frequency | WHERE Columns | JOIN Columns | ORDER BY |
|-------|-----------|---------------|--------------|----------|
| Q1 | 1000/min | status, date | customer_id | date DESC |
| Q2 | 500/min | customer_id | - | - |

## Recommended Indexes

### Primary Index
```sql
-- Purpose: [Description]
CREATE INDEX idx_name ON table(col1, col2);
```

### Supporting Indexes
```sql
-- Purpose: [Description]
CREATE INDEX idx_name ON table(col1) INCLUDE (col2, col3);
```

## Index Coverage Matrix

| Index | Q1 | Q2 | Q3 | Q4 |
|-------|----|----|----|----|
| idx_1 | ✓ Seek | ✓ Seek | - | ✓ Scan |
| idx_2 | - | - | ✓ Seek | - |

## Maintenance Schedule
- **Rebuild:** Weekly (fragmentation > 30%)
- **Reorganize:** Daily (fragmentation > 10%)
- **Statistics Update:** After bulk operations
```

## Output Format

When invoked, provide:
1. Index definitions (CREATE INDEX statements)
2. Rationale for each index
3. Query coverage analysis
4. Maintenance recommendations
5. Platform-specific considerations

## Cross-References

- **Query Optimization (query-optimization skill):** Verify index usage
- **Performance Tuning (performance-tuning skill):** Broader optimization
- **SQL Schema (sql-schema skill):** Table structure and constraints
