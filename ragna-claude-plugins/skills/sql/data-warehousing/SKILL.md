---
name: data-warehousing
description: Design data warehouse schemas including dimensional modeling, slowly changing dimensions, ETL patterns, and analytics queries
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Data Warehousing Skill

This skill provides data warehouse design patterns including dimensional modeling, slowly changing dimensions, ETL patterns, and analytics query optimization.

## Purpose

Design and implement data warehouse solutions for:
- **Dimensional Modeling** - Star and snowflake schemas
- **Slowly Changing Dimensions** - Historical data tracking
- **ETL Patterns** - Efficient data loading
- **Analytics Queries** - Aggregations, cubes, rollups

## Instructions

When this skill is invoked:

1. **Understand the analytics requirements:**
   - What questions need answering?
   - What dimensions and measures?
   - Historical tracking needs?

2. **Design dimensional model:**
   - Identify facts and dimensions
   - Choose schema type (star, snowflake)
   - Define grain

3. **Implement SCD strategy:**
   - Determine change tracking needs
   - Choose appropriate SCD type

4. **Design ETL patterns:**
   - Incremental loading
   - Bulk operations
   - Error handling

## Dimensional Modeling

### Star Schema

```sql
-- Fact table: Sales transactions (center of star)
CREATE TABLE fact_sales (
    sale_key BIGSERIAL PRIMARY KEY,
    date_key INT NOT NULL REFERENCES dim_date(date_key),
    product_key INT NOT NULL REFERENCES dim_product(product_key),
    customer_key INT NOT NULL REFERENCES dim_customer(customer_key),
    store_key INT NOT NULL REFERENCES dim_store(store_key),
    promotion_key INT REFERENCES dim_promotion(promotion_key),

    -- Degenerate dimension (no separate table)
    order_number VARCHAR(50) NOT NULL,
    line_number INT NOT NULL,

    -- Measures
    quantity INT NOT NULL,
    unit_price NUMERIC(10,2) NOT NULL,
    discount_amount NUMERIC(10,2) DEFAULT 0,
    net_amount NUMERIC(12,2) NOT NULL,
    cost_amount NUMERIC(10,2),
    profit_amount NUMERIC(12,2)
);

-- Create indexes for common query patterns
CREATE INDEX idx_fact_sales_date ON fact_sales(date_key);
CREATE INDEX idx_fact_sales_product ON fact_sales(product_key);
CREATE INDEX idx_fact_sales_customer ON fact_sales(customer_key);
CREATE INDEX idx_fact_sales_composite ON fact_sales(date_key, product_key, store_key);

-- Date dimension (conformed dimension)
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,  -- YYYYMMDD format
    full_date DATE NOT NULL UNIQUE,
    day_of_week INT NOT NULL,
    day_name VARCHAR(10) NOT NULL,
    day_of_month INT NOT NULL,
    day_of_year INT NOT NULL,
    week_of_year INT NOT NULL,
    month_number INT NOT NULL,
    month_name VARCHAR(10) NOT NULL,
    quarter INT NOT NULL,
    quarter_name VARCHAR(10) NOT NULL,
    year INT NOT NULL,
    fiscal_year INT NOT NULL,
    fiscal_quarter INT NOT NULL,
    is_weekend BOOLEAN NOT NULL,
    is_holiday BOOLEAN NOT NULL DEFAULT FALSE,
    holiday_name VARCHAR(50)
);

-- Product dimension
CREATE TABLE dim_product (
    product_key SERIAL PRIMARY KEY,
    product_id VARCHAR(50) NOT NULL,  -- Business key
    product_name VARCHAR(255) NOT NULL,
    product_description TEXT,
    brand VARCHAR(100),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    unit_cost NUMERIC(10,2),
    unit_price NUMERIC(10,2),
    is_active BOOLEAN DEFAULT TRUE,

    -- SCD Type 2 columns
    effective_date DATE NOT NULL,
    expiration_date DATE DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE
);

-- Customer dimension
CREATE TABLE dim_customer (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,  -- Business key
    customer_name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    phone VARCHAR(50),
    address_line1 VARCHAR(255),
    address_line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    postal_code VARCHAR(20),
    country VARCHAR(100),
    customer_segment VARCHAR(50),
    acquisition_date DATE,

    -- SCD Type 2 columns
    effective_date DATE NOT NULL,
    expiration_date DATE DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE
);

-- Store dimension
CREATE TABLE dim_store (
    store_key SERIAL PRIMARY KEY,
    store_id VARCHAR(50) NOT NULL,
    store_name VARCHAR(255) NOT NULL,
    store_type VARCHAR(50),
    address VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    country VARCHAR(100),
    region VARCHAR(50),
    manager_name VARCHAR(255),
    open_date DATE,
    square_footage INT,
    is_active BOOLEAN DEFAULT TRUE
);
```

### Snowflake Schema

```sql
-- Normalized dimensions (snowflake branches)
CREATE TABLE dim_category (
    category_key SERIAL PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    department_key INT REFERENCES dim_department(department_key)
);

CREATE TABLE dim_department (
    department_key SERIAL PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

-- Product dimension references category (normalized)
CREATE TABLE dim_product_snowflake (
    product_key SERIAL PRIMARY KEY,
    product_id VARCHAR(50) NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    category_key INT REFERENCES dim_category(category_key),
    brand_key INT REFERENCES dim_brand(brand_key)
);

-- Trade-off: More normalized, less redundancy
-- But requires more joins for queries
```

## Slowly Changing Dimensions (SCD)

### Type 0: Retain Original

```sql
-- Never update - keep original value
CREATE TABLE dim_customer_type0 (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL UNIQUE,
    customer_name VARCHAR(255) NOT NULL,  -- Never changes
    original_signup_date DATE NOT NULL    -- Never changes
);

-- Use case: Audit fields, original values
```

### Type 1: Overwrite

```sql
-- Simply update the value - no history
CREATE TABLE dim_customer_type1 (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL UNIQUE,
    customer_name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    address VARCHAR(255)  -- Just overwrite when changed
);

-- Update procedure
UPDATE dim_customer_type1
SET email = 'new@email.com', address = '456 New St'
WHERE customer_id = 'CUST001';

-- Use case: Corrections, non-historical attributes
```

### Type 2: Add New Row (Full History)

```sql
-- Track complete history with versioned rows
CREATE TABLE dim_customer_type2 (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,  -- Business key (not unique)
    customer_name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    address VARCHAR(255),
    effective_date DATE NOT NULL,
    expiration_date DATE DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE,
    version INT DEFAULT 1
);

-- Create unique constraint on business key + dates
CREATE UNIQUE INDEX idx_customer_type2_bk ON dim_customer_type2(customer_id, effective_date);

-- Procedure to handle SCD Type 2 update
CREATE OR REPLACE PROCEDURE scd_type2_customer(
    p_customer_id VARCHAR(50),
    p_customer_name VARCHAR(255),
    p_email VARCHAR(255),
    p_address VARCHAR(255)
)
LANGUAGE plpgsql AS $$
DECLARE
    v_current_key INT;
    v_current_version INT;
    v_has_changes BOOLEAN;
BEGIN
    -- Get current record
    SELECT customer_key, version,
           (customer_name != p_customer_name OR
            email != p_email OR
            address != p_address)
    INTO v_current_key, v_current_version, v_has_changes
    FROM dim_customer_type2
    WHERE customer_id = p_customer_id AND is_current = TRUE;

    IF v_current_key IS NULL THEN
        -- New customer - insert first record
        INSERT INTO dim_customer_type2
            (customer_id, customer_name, email, address, effective_date)
        VALUES
            (p_customer_id, p_customer_name, p_email, p_address, CURRENT_DATE);
    ELSIF v_has_changes THEN
        -- Changes detected - expire current and insert new
        UPDATE dim_customer_type2
        SET expiration_date = CURRENT_DATE - 1, is_current = FALSE
        WHERE customer_key = v_current_key;

        INSERT INTO dim_customer_type2
            (customer_id, customer_name, email, address, effective_date, version)
        VALUES
            (p_customer_id, p_customer_name, p_email, p_address, CURRENT_DATE, v_current_version + 1);
    END IF;
END $$;

-- Query: Get customer at specific point in time
SELECT * FROM dim_customer_type2
WHERE customer_id = 'CUST001'
  AND '2024-01-15' BETWEEN effective_date AND expiration_date;

-- Query: Get current customer
SELECT * FROM dim_customer_type2
WHERE customer_id = 'CUST001' AND is_current = TRUE;
```

### Type 3: Add New Column

```sql
-- Track limited history (previous value only)
CREATE TABLE dim_customer_type3 (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL UNIQUE,
    customer_name VARCHAR(255) NOT NULL,
    current_address VARCHAR(255),
    previous_address VARCHAR(255),
    address_change_date DATE
);

-- Update procedure
UPDATE dim_customer_type3
SET previous_address = current_address,
    current_address = '456 New St',
    address_change_date = CURRENT_DATE
WHERE customer_id = 'CUST001';

-- Use case: When only current and previous values needed
```

### Type 6: Hybrid (1+2+3)

```sql
-- Combines Type 1, 2, and 3
CREATE TABLE dim_customer_type6 (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    customer_name VARCHAR(255) NOT NULL,

    -- Type 2 historical tracking
    historical_address VARCHAR(255),
    effective_date DATE NOT NULL,
    expiration_date DATE DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE,

    -- Type 3 quick access to current
    current_address VARCHAR(255),

    -- Type 1 overwritten field
    last_update_date DATE
);

-- Benefits:
-- - Full history (Type 2)
-- - Quick current value access (Type 3)
-- - Some fields simply overwritten (Type 1)
```

## ETL Patterns

### Bulk Insert Optimization

```sql
-- PostgreSQL: COPY for fastest bulk insert
COPY staging_sales FROM '/data/sales.csv' WITH (FORMAT CSV, HEADER);

-- Alternative: Unlogged table for staging
CREATE UNLOGGED TABLE staging_sales (...);  -- Faster, no WAL
COPY staging_sales FROM '/data/sales.csv' WITH (FORMAT CSV, HEADER);
-- Process staging data
DROP TABLE staging_sales;

-- SQL Server: BULK INSERT
BULK INSERT staging_sales
FROM 'C:\data\sales.csv'
WITH (
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    FIRSTROW = 2,
    TABLOCK
);
```

### Incremental Load Pattern

```sql
-- Track last load timestamp
CREATE TABLE etl_control (
    table_name VARCHAR(100) PRIMARY KEY,
    last_load_timestamp TIMESTAMP,
    last_load_key BIGINT,
    row_count BIGINT
);

-- Incremental load using timestamp
INSERT INTO fact_sales (date_key, product_key, customer_key, ...)
SELECT
    d.date_key,
    p.product_key,
    c.customer_key,
    ...
FROM source_transactions t
JOIN dim_date d ON DATE(t.transaction_date) = d.full_date
JOIN dim_product p ON t.product_id = p.product_id AND p.is_current = TRUE
JOIN dim_customer c ON t.customer_id = c.customer_id AND c.is_current = TRUE
WHERE t.modified_date > (
    SELECT last_load_timestamp FROM etl_control WHERE table_name = 'fact_sales'
);

-- Update control table
UPDATE etl_control
SET last_load_timestamp = CURRENT_TIMESTAMP,
    row_count = row_count + (SELECT COUNT(*) FROM just_loaded)
WHERE table_name = 'fact_sales';
```

### Change Data Capture (CDC)

```sql
-- PostgreSQL: Logical replication with pgoutput
-- SQL Server: Built-in CDC
-- MySQL: Binary log parsing

-- Example: Process CDC events
CREATE TABLE cdc_customer_changes (
    change_id SERIAL PRIMARY KEY,
    operation CHAR(1) NOT NULL,  -- I=Insert, U=Update, D=Delete
    customer_id VARCHAR(50),
    customer_name VARCHAR(255),
    email VARCHAR(255),
    address VARCHAR(255),
    change_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Apply CDC changes to dimension
CREATE OR REPLACE PROCEDURE apply_customer_cdc()
LANGUAGE plpgsql AS $$
BEGIN
    FOR change IN (SELECT * FROM cdc_customer_changes ORDER BY change_id) LOOP
        IF change.operation = 'I' OR change.operation = 'U' THEN
            CALL scd_type2_customer(
                change.customer_id,
                change.customer_name,
                change.email,
                change.address
            );
        ELSIF change.operation = 'D' THEN
            -- Soft delete: expire current record
            UPDATE dim_customer_type2
            SET expiration_date = CURRENT_DATE, is_current = FALSE
            WHERE customer_id = change.customer_id AND is_current = TRUE;
        END IF;
    END LOOP;

    -- Clear processed changes
    DELETE FROM cdc_customer_changes;
END $$;
```

### MERGE/UPSERT Pattern

```sql
-- PostgreSQL: INSERT ON CONFLICT
INSERT INTO dim_product (product_id, product_name, brand, category, unit_price)
VALUES ('PROD001', 'Widget', 'Acme', 'Gadgets', 19.99)
ON CONFLICT (product_id)
DO UPDATE SET
    product_name = EXCLUDED.product_name,
    brand = EXCLUDED.brand,
    category = EXCLUDED.category,
    unit_price = EXCLUDED.unit_price;

-- SQL Server: MERGE
MERGE INTO dim_product AS target
USING staging_products AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET
        product_name = source.product_name,
        brand = source.brand,
        category = source.category,
        unit_price = source.unit_price
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, brand, category, unit_price)
    VALUES (source.product_id, source.product_name, source.brand,
            source.category, source.unit_price);
```

## Analytics Queries

### CUBE and ROLLUP

```sql
-- ROLLUP: Hierarchical aggregation
SELECT
    COALESCE(d.year::TEXT, 'All Years') AS year,
    COALESCE(d.quarter_name, 'All Quarters') AS quarter,
    COALESCE(d.month_name, 'All Months') AS month,
    SUM(f.net_amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY ROLLUP (d.year, d.quarter_name, d.month_name)
ORDER BY d.year, d.quarter_name, d.month_name;

-- Results include:
-- Year | Quarter | Month | Sales
-- 2024 | Q1      | Jan   | 100000  (detail)
-- 2024 | Q1      | Feb   | 120000  (detail)
-- 2024 | Q1      | NULL  | 350000  (Q1 subtotal)
-- 2024 | NULL    | NULL  | 1500000 (2024 subtotal)
-- NULL | NULL    | NULL  | 3000000 (grand total)

-- CUBE: All combinations
SELECT
    COALESCE(p.category, 'All Categories') AS category,
    COALESCE(s.region, 'All Regions') AS region,
    SUM(f.net_amount) AS total_sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_store s ON f.store_key = s.store_key
GROUP BY CUBE (p.category, s.region);

-- Results include:
-- Category    | Region | Sales
-- Electronics | North  | 50000   (detail)
-- Electronics | South  | 45000   (detail)
-- Electronics | NULL   | 95000   (category subtotal)
-- NULL        | North  | 120000  (region subtotal)
-- NULL        | NULL   | 200000  (grand total)
```

### GROUPING SETS

```sql
-- Custom aggregation combinations
SELECT
    d.year,
    d.quarter_name,
    p.category,
    SUM(f.net_amount) AS total_sales,
    GROUPING(d.year) AS is_year_total,
    GROUPING(d.quarter_name) AS is_quarter_total,
    GROUPING(p.category) AS is_category_total
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY GROUPING SETS (
    (d.year, d.quarter_name, p.category),  -- Full detail
    (d.year, d.quarter_name),              -- By time only
    (d.year, p.category),                  -- By year and category
    (p.category),                          -- By category only
    ()                                     -- Grand total
);
```

### Window Analytics

```sql
-- Running totals, rankings, and comparisons
SELECT
    d.full_date,
    d.month_name,
    SUM(f.net_amount) AS daily_sales,

    -- Running total for the month
    SUM(SUM(f.net_amount)) OVER (
        PARTITION BY d.year, d.month_number
        ORDER BY d.full_date
    ) AS mtd_sales,

    -- Rank within month
    RANK() OVER (
        PARTITION BY d.year, d.month_number
        ORDER BY SUM(f.net_amount) DESC
    ) AS daily_rank,

    -- Comparison to previous day
    LAG(SUM(f.net_amount)) OVER (ORDER BY d.full_date) AS prev_day_sales,

    -- Moving 7-day average
    AVG(SUM(f.net_amount)) OVER (
        ORDER BY d.full_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS avg_7day

FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
GROUP BY d.full_date, d.year, d.month_number, d.month_name
ORDER BY d.full_date;
```

### Materialized Views for Aggregates

```sql
-- PostgreSQL: Create materialized view for common aggregation
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    d.year,
    d.month_number,
    d.month_name,
    p.category,
    s.region,
    SUM(f.net_amount) AS total_sales,
    SUM(f.quantity) AS total_quantity,
    COUNT(*) AS transaction_count,
    AVG(f.net_amount) AS avg_transaction
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_store s ON f.store_key = s.store_key
GROUP BY d.year, d.month_number, d.month_name, p.category, s.region;

-- Create indexes on materialized view
CREATE INDEX idx_mv_monthly_year_month ON mv_monthly_sales(year, month_number);
CREATE INDEX idx_mv_monthly_category ON mv_monthly_sales(category);

-- Refresh (complete refresh)
REFRESH MATERIALIZED VIEW mv_monthly_sales;

-- Refresh concurrently (requires unique index)
CREATE UNIQUE INDEX idx_mv_monthly_unique ON mv_monthly_sales(year, month_number, category, region);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_monthly_sales;
```

## Date Dimension Generator

```sql
-- Generate complete date dimension
INSERT INTO dim_date
SELECT
    TO_CHAR(datum, 'YYYYMMDD')::INT AS date_key,
    datum AS full_date,
    EXTRACT(ISODOW FROM datum)::INT AS day_of_week,
    TO_CHAR(datum, 'Day') AS day_name,
    EXTRACT(DAY FROM datum)::INT AS day_of_month,
    EXTRACT(DOY FROM datum)::INT AS day_of_year,
    EXTRACT(WEEK FROM datum)::INT AS week_of_year,
    EXTRACT(MONTH FROM datum)::INT AS month_number,
    TO_CHAR(datum, 'Month') AS month_name,
    EXTRACT(QUARTER FROM datum)::INT AS quarter,
    'Q' || EXTRACT(QUARTER FROM datum)::TEXT AS quarter_name,
    EXTRACT(YEAR FROM datum)::INT AS year,
    CASE
        WHEN EXTRACT(MONTH FROM datum) >= 7
        THEN EXTRACT(YEAR FROM datum)::INT + 1
        ELSE EXTRACT(YEAR FROM datum)::INT
    END AS fiscal_year,
    CASE
        WHEN EXTRACT(MONTH FROM datum) IN (7,8,9) THEN 1
        WHEN EXTRACT(MONTH FROM datum) IN (10,11,12) THEN 2
        WHEN EXTRACT(MONTH FROM datum) IN (1,2,3) THEN 3
        ELSE 4
    END AS fiscal_quarter,
    EXTRACT(ISODOW FROM datum) IN (6, 7) AS is_weekend,
    FALSE AS is_holiday,
    NULL AS holiday_name
FROM generate_series('2020-01-01'::DATE, '2030-12-31'::DATE, '1 day') AS datum;
```

## Output Format

When invoked, provide:
1. Dimensional model design (fact and dimension tables)
2. SCD strategy implementation
3. ETL patterns for data loading
4. Analytics query examples
5. Index recommendations for query performance

## Cross-References

- **SQL Schema (sql-schema skill):** Table DDL and constraints
- **Index Design (index-design skill):** Indexes for warehouse queries
- **Query Optimization (query-optimization skill):** Analytics query performance
- **Performance Tuning (performance-tuning skill):** Warehouse performance
