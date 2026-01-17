---
name: sql-query-patterns
description: Generate advanced SQL query patterns including CTEs, window functions, complex joins, pivoting, and temporal queries across PostgreSQL, MySQL, SQL Server, and Oracle
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Query Patterns Skill

This skill generates advanced SQL query patterns for complex data retrieval, analytics, and transformation across multiple database platforms.

## Purpose

Provide production-ready SQL patterns for:
- **Common Table Expressions (CTEs)** - Recursive and non-recursive
- **Window Functions** - Ranking, aggregation, analytics
- **Complex Joins** - Self-joins, lateral, anti-joins
- **Pivoting & Unpivoting** - Data transformation
- **Temporal Queries** - Time-based data handling

## Instructions

When this skill is invoked:

1. **Identify the use case:**
   - What data pattern is needed?
   - What is the expected output?

2. **Determine the database platform:**
   - PostgreSQL (default)
   - MySQL / MariaDB
   - SQL Server
   - Oracle

3. **Generate appropriate SQL patterns:**
   - Portable ANSI SQL when possible
   - Platform-specific optimizations when needed

4. **Provide explanation and usage guidance**

## Common Table Expressions (CTEs)

### Basic CTE Structure

```sql
WITH cte_name AS (
    SELECT columns
    FROM table
    WHERE conditions
)
SELECT *
FROM cte_name;
```

### Multiple CTEs

```sql
WITH active_customers AS (
    SELECT
        customer_id,
        customer_name,
        email,
        registration_date
    FROM customers
    WHERE is_active = true
        AND registration_date >= CURRENT_DATE - INTERVAL '1 year'
),
customer_orders AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent,
        MAX(order_date) AS last_order_date
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY customer_id
)
SELECT
    ac.customer_id,
    ac.customer_name,
    ac.email,
    COALESCE(co.order_count, 0) AS order_count,
    COALESCE(co.total_spent, 0) AS total_spent,
    co.last_order_date
FROM active_customers ac
LEFT JOIN customer_orders co ON ac.customer_id = co.customer_id
ORDER BY co.total_spent DESC NULLS LAST
LIMIT 100;
```

### Recursive CTEs - Hierarchical Data

```sql
-- Employee hierarchy with level and path
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: top-level employees
    SELECT
        employee_id,
        name,
        manager_id,
        1 AS level,
        ARRAY[employee_id] AS path,
        name::TEXT AS path_names
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1,
        eh.path || e.employee_id,
        eh.path_names || ' > ' || e.name
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
    WHERE NOT e.employee_id = ANY(eh.path)  -- Prevent cycles
)
SELECT
    employee_id,
    REPEAT('  ', level - 1) || name AS hierarchical_name,
    level,
    path_names
FROM employee_hierarchy
ORDER BY path;
```

### Recursive CTE - Bill of Materials

```sql
-- Product components with quantities
WITH RECURSIVE bom AS (
    -- Base: top-level products
    SELECT
        product_id,
        component_id,
        quantity,
        1 AS level,
        ARRAY[product_id] AS path
    FROM product_components
    WHERE product_id = 100  -- Starting product

    UNION ALL

    -- Recursive: sub-components
    SELECT
        pc.product_id,
        pc.component_id,
        pc.quantity * bom.quantity AS quantity,
        bom.level + 1,
        bom.path || pc.product_id
    FROM product_components pc
    INNER JOIN bom ON pc.product_id = bom.component_id
    WHERE NOT pc.product_id = ANY(bom.path)
)
SELECT
    component_id,
    SUM(quantity) AS total_quantity,
    MAX(level) AS depth
FROM bom
GROUP BY component_id
ORDER BY total_quantity DESC;
```

## Window Functions

### Ranking Functions

```sql
SELECT
    product_id,
    category,
    sales_amount,
    -- Row number (unique, no ties)
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales_amount DESC) AS row_num,
    -- Rank (gaps after ties)
    RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) AS rank,
    -- Dense rank (no gaps after ties)
    DENSE_RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) AS dense_rank,
    -- Percentile bucket
    NTILE(4) OVER (ORDER BY sales_amount) AS quartile
FROM product_sales;
```

### Running Totals and Moving Averages

```sql
SELECT
    order_date,
    order_id,
    amount,
    -- Running total
    SUM(amount) OVER (
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    -- Moving average (last 7 days)
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day,
    -- Previous day's amount
    LAG(amount, 1) OVER (ORDER BY order_date) AS prev_day_amount,
    -- Next day's amount
    LEAD(amount, 1) OVER (ORDER BY order_date) AS next_day_amount,
    -- First value in window
    FIRST_VALUE(amount) OVER (
        PARTITION BY EXTRACT(MONTH FROM order_date)
        ORDER BY order_date
    ) AS first_of_month,
    -- Rank within partition
    RANK() OVER (
        PARTITION BY EXTRACT(MONTH FROM order_date)
        ORDER BY amount DESC
    ) AS monthly_rank
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '1 year';
```

### Frame Clause Options

```sql
-- ROWS vs RANGE behavior
SELECT
    order_date,
    amount,
    -- ROWS: Exact row count
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS sum_last_3_rows,
    -- RANGE: Based on value (handles ties differently)
    SUM(amount) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '2 days' PRECEDING AND CURRENT ROW
    ) AS sum_last_3_days
FROM orders;

-- Frame specifications
-- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- Running total
-- ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING          -- Centered moving average
-- ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING  -- Reverse running total
```

## Complex Joins

### Self-Join for Hierarchical Data

```sql
-- Find employees and their managers
SELECT
    e.employee_id,
    e.name AS employee_name,
    e.title,
    m.name AS manager_name,
    m.title AS manager_title
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

### LATERAL Joins (Row-wise Subqueries)

```sql
-- Get top 5 orders per customer (PostgreSQL)
SELECT
    c.customer_id,
    c.customer_name,
    recent.order_id,
    recent.order_date,
    recent.total_amount
FROM customers c
CROSS JOIN LATERAL (
    SELECT order_id, order_date, total_amount
    FROM orders o
    WHERE o.customer_id = c.customer_id
    ORDER BY order_date DESC
    LIMIT 5
) recent;

-- SQL Server equivalent (CROSS APPLY)
SELECT
    c.customer_id,
    c.customer_name,
    recent.order_id,
    recent.order_date,
    recent.total_amount
FROM customers c
CROSS APPLY (
    SELECT TOP 5 order_id, order_date, total_amount
    FROM orders o
    WHERE o.customer_id = c.customer_id
    ORDER BY order_date DESC
) recent;
```

### Anti-Join (Records Not Matching)

```sql
-- Customers without orders (NOT EXISTS - most efficient)
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Alternative: LEFT JOIN with NULL check
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- NOT IN (beware of NULLs!)
SELECT c.*
FROM customers c
WHERE c.customer_id NOT IN (
    SELECT customer_id FROM orders WHERE customer_id IS NOT NULL
);
```

### Semi-Join (Records Matching)

```sql
-- Customers with orders (EXISTS)
SELECT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Using IN
SELECT c.*
FROM customers c
WHERE c.customer_id IN (SELECT customer_id FROM orders);
```

## Pivoting & Unpivoting

### Pivot with CASE (Portable)

```sql
-- Sales by product and month
SELECT
    product_name,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 1 THEN amount ELSE 0 END) AS jan,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 2 THEN amount ELSE 0 END) AS feb,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 3 THEN amount ELSE 0 END) AS mar,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 4 THEN amount ELSE 0 END) AS apr,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 5 THEN amount ELSE 0 END) AS may,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 6 THEN amount ELSE 0 END) AS jun,
    SUM(amount) AS total
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
WHERE EXTRACT(YEAR FROM oi.order_date) = 2024
GROUP BY product_name
ORDER BY total DESC;
```

### SQL Server PIVOT

```sql
SELECT *
FROM (
    SELECT product_name, MONTH(order_date) AS month, amount
    FROM order_items oi
    INNER JOIN products p ON oi.product_id = p.product_id
    WHERE YEAR(order_date) = 2024
) AS source
PIVOT (
    SUM(amount) FOR month IN ([1], [2], [3], [4], [5], [6], [7], [8], [9], [10], [11], [12])
) AS pivot_table;
```

### Unpivot Example

```sql
-- SQL Server UNPIVOT
SELECT product_name, month, amount
FROM (
    SELECT product_name, jan, feb, mar, apr, may, jun
    FROM monthly_sales
) AS source
UNPIVOT (
    amount FOR month IN (jan, feb, mar, apr, may, jun)
) AS unpivot_table;

-- PostgreSQL with VALUES
SELECT
    product_name,
    month_name,
    amount
FROM monthly_sales ms
CROSS JOIN LATERAL (
    VALUES
        ('jan', ms.jan),
        ('feb', ms.feb),
        ('mar', ms.mar),
        ('apr', ms.apr),
        ('may', ms.may),
        ('jun', ms.jun)
) AS t(month_name, amount)
WHERE amount > 0;
```

## Temporal Queries

### Slowly Changing Dimension Type 2

```sql
-- Get customer state as of specific date
SELECT
    c.customer_id,
    c.customer_name,
    c.address,
    c.status
FROM customer_history c
WHERE c.customer_id = 12345
    AND '2024-01-15' BETWEEN c.valid_from AND c.valid_to;

-- Point-in-time snapshot for all customers (PostgreSQL)
SELECT DISTINCT ON (customer_id)
    customer_id,
    customer_name,
    address,
    status,
    valid_from,
    valid_to
FROM customer_history
WHERE valid_from <= '2024-01-15'
ORDER BY customer_id, valid_from DESC;
```

### Gap and Island Detection

```sql
-- Find gaps in sequential data
WITH numbered AS (
    SELECT
        id,
        LAG(id) OVER (ORDER BY id) AS prev_id
    FROM items
)
SELECT
    prev_id + 1 AS gap_start,
    id - 1 AS gap_end
FROM numbered
WHERE id - prev_id > 1;

-- Find islands (consecutive sequences)
WITH grouped AS (
    SELECT
        order_date,
        order_date - INTERVAL '1 day' * ROW_NUMBER() OVER (ORDER BY order_date) AS grp
    FROM orders
    WHERE customer_id = 123
)
SELECT
    MIN(order_date) AS island_start,
    MAX(order_date) AS island_end,
    COUNT(*) AS consecutive_days
FROM grouped
GROUP BY grp
ORDER BY island_start;
```

### Date Series Generation

```sql
-- PostgreSQL: Generate date series
SELECT date::DATE
FROM generate_series('2024-01-01'::DATE, '2024-12-31'::DATE, '1 day') AS date;

-- SQL Server: Generate date series
WITH dates AS (
    SELECT CAST('2024-01-01' AS DATE) AS date
    UNION ALL
    SELECT DATEADD(day, 1, date)
    FROM dates
    WHERE date < '2024-12-31'
)
SELECT date FROM dates
OPTION (MAXRECURSION 366);

-- Fill missing dates with zero values
WITH date_series AS (
    SELECT date::DATE
    FROM generate_series('2024-01-01'::DATE, '2024-12-31'::DATE, '1 day') AS date
)
SELECT
    ds.date,
    COALESCE(SUM(o.amount), 0) AS daily_sales
FROM date_series ds
LEFT JOIN orders o ON DATE(o.order_date) = ds.date
GROUP BY ds.date
ORDER BY ds.date;
```

## Analytical Patterns

### Cohort Analysis

```sql
-- User retention by signup cohort
WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_activity
)
SELECT
    c.cohort_month,
    a.activity_month,
    EXTRACT(MONTH FROM AGE(a.activity_month, c.cohort_month)) AS months_since_signup,
    COUNT(DISTINCT a.user_id) AS active_users,
    COUNT(DISTINCT a.user_id)::FLOAT / COUNT(DISTINCT c.user_id) AS retention_rate
FROM cohorts c
JOIN activity a ON c.user_id = a.user_id
GROUP BY c.cohort_month, a.activity_month
ORDER BY c.cohort_month, a.activity_month;
```

### Funnel Analysis

```sql
-- Conversion funnel
WITH funnel AS (
    SELECT
        session_id,
        MAX(CASE WHEN event_type = 'page_view' THEN 1 ELSE 0 END) AS viewed,
        MAX(CASE WHEN event_type = 'add_to_cart' THEN 1 ELSE 0 END) AS added_to_cart,
        MAX(CASE WHEN event_type = 'checkout_started' THEN 1 ELSE 0 END) AS started_checkout,
        MAX(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS purchased
    FROM events
    WHERE event_date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY session_id
)
SELECT
    SUM(viewed) AS total_views,
    SUM(added_to_cart) AS added_to_cart,
    SUM(started_checkout) AS started_checkout,
    SUM(purchased) AS purchased,
    ROUND(100.0 * SUM(added_to_cart) / NULLIF(SUM(viewed), 0), 2) AS view_to_cart_rate,
    ROUND(100.0 * SUM(purchased) / NULLIF(SUM(started_checkout), 0), 2) AS checkout_conversion_rate
FROM funnel;
```

### Year-over-Year Comparison

```sql
SELECT
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(CASE WHEN EXTRACT(YEAR FROM order_date) = 2023 THEN amount ELSE 0 END) AS sales_2023,
    SUM(CASE WHEN EXTRACT(YEAR FROM order_date) = 2024 THEN amount ELSE 0 END) AS sales_2024,
    ROUND(100.0 * (
        SUM(CASE WHEN EXTRACT(YEAR FROM order_date) = 2024 THEN amount ELSE 0 END) -
        SUM(CASE WHEN EXTRACT(YEAR FROM order_date) = 2023 THEN amount ELSE 0 END)
    ) / NULLIF(SUM(CASE WHEN EXTRACT(YEAR FROM order_date) = 2023 THEN amount ELSE 0 END), 0), 2) AS yoy_growth
FROM orders
WHERE order_date >= '2023-01-01' AND order_date < '2025-01-01'
GROUP BY EXTRACT(MONTH FROM order_date)
ORDER BY month;
```

## Output Format

When invoked, provide:
1. Complete SQL query with comments
2. Explanation of the pattern and when to use it
3. Platform-specific variations if needed
4. Performance considerations
5. Example output/result structure

## Cross-References

- **Query Optimization (query-optimization skill):** Optimize generated queries
- **Index Design (index-design skill):** Index recommendations for patterns
- **Data Warehousing (data-warehousing skill):** Analytical query patterns
