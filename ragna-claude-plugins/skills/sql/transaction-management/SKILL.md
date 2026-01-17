---
name: transaction-management
description: Design transaction strategies including isolation levels, deadlock prevention, locking patterns, and optimistic concurrency for PostgreSQL, MySQL, SQL Server, and Oracle
allowed-tools: ["Read", "Edit", "Write"]
---

# SQL Transaction Management Skill

This skill provides transaction design patterns including isolation levels, deadlock prevention, and locking strategies across multiple database platforms.

## Purpose

Design robust transaction strategies for:
- **Data Consistency** - ACID compliance
- **Concurrency Control** - Multiple simultaneous operations
- **Deadlock Prevention** - Avoiding resource contention
- **Performance** - Minimal locking overhead

## Instructions

When this skill is invoked:

1. **Understand the use case:**
   - Read vs write patterns
   - Concurrency requirements
   - Consistency requirements

2. **Determine database platform:**
   - PostgreSQL (MVCC-based)
   - MySQL (InnoDB)
   - SQL Server
   - Oracle

3. **Design transaction strategy:**
   - Choose isolation level
   - Design locking approach
   - Plan deadlock prevention

4. **Provide implementation patterns**

## Isolation Levels

### Overview

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| READ UNCOMMITTED | Possible | Possible | Possible | Fastest |
| READ COMMITTED | Prevented | Possible | Possible | Fast |
| REPEATABLE READ | Prevented | Prevented | Possible* | Moderate |
| SERIALIZABLE | Prevented | Prevented | Prevented | Slowest |

*PostgreSQL's REPEATABLE READ also prevents phantom reads.

### READ UNCOMMITTED

```sql
-- Allows dirty reads - rarely appropriate
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Use case: Read-only reporting where approximate data is acceptable
-- Warning: Can read uncommitted (potentially rolled back) data
```

### READ COMMITTED (Default for most databases)

```sql
-- PostgreSQL/SQL Server/Oracle default
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN;
    -- Each SELECT sees committed data as of query start
    SELECT * FROM accounts WHERE id = 1;  -- May see different data if queried again
COMMIT;

-- Use case: Most OLTP applications
-- Trade-off: Non-repeatable reads possible
```

### REPEATABLE READ

```sql
-- MySQL InnoDB default, PostgreSQL/SQL Server option
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN;
    SELECT * FROM accounts WHERE id = 1;  -- Snapshot taken
    -- Same query returns same result within transaction
    SELECT * FROM accounts WHERE id = 1;  -- Guaranteed same
COMMIT;

-- Use case: Reports requiring consistent view
-- Trade-off: Phantom reads possible (except PostgreSQL)
```

### SERIALIZABLE

```sql
-- Strongest isolation - transactions appear sequential
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

BEGIN;
    -- All reads and writes fully isolated
    SELECT SUM(balance) FROM accounts;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Use case: Financial transactions, inventory management
-- Trade-off: Highest lock contention, serialization errors
```

### PostgreSQL SNAPSHOT Isolation

```sql
-- PostgreSQL uses MVCC - REPEATABLE READ is actually snapshot isolation
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
    -- Entire transaction sees consistent snapshot
    SELECT * FROM orders;  -- Snapshot as of transaction start
    -- No phantom reads in PostgreSQL
COMMIT;
```

## Locking Strategies

### Pessimistic Locking

Lock resources before accessing them.

```sql
-- Row-level lock (SELECT FOR UPDATE)
BEGIN;
    -- Lock the row - blocks other SELECT FOR UPDATE
    SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

    -- Safe to read and modify
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Lock with NOWAIT (fail immediately if locked)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;

-- Lock with timeout (PostgreSQL)
SET lock_timeout = '5s';
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Lock with SKIP LOCKED (process unlocked rows only)
SELECT * FROM tasks WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 10;
```

### Lock Types

```sql
-- PostgreSQL lock modes
SELECT * FROM accounts FOR UPDATE;           -- Exclusive row lock
SELECT * FROM accounts FOR NO KEY UPDATE;    -- Weaker lock, allows FK checks
SELECT * FROM accounts FOR SHARE;            -- Shared lock, blocks updates
SELECT * FROM accounts FOR KEY SHARE;        -- Weakest, only blocks key updates

-- SQL Server lock hints
SELECT * FROM accounts WITH (ROWLOCK, UPDLOCK) WHERE id = 1;
SELECT * FROM accounts WITH (TABLOCKX);      -- Exclusive table lock
SELECT * FROM accounts WITH (NOLOCK);        -- No locking (dirty reads)
SELECT * FROM accounts WITH (READPAST);      -- Skip locked rows
```

### Optimistic Locking

Check for conflicts at commit time using version column.

```sql
-- Add version column
ALTER TABLE accounts ADD COLUMN version INTEGER DEFAULT 1;

-- Read with version
SELECT id, balance, version FROM accounts WHERE id = 1;
-- Returns: id=1, balance=1000, version=5

-- Update with version check
UPDATE accounts
SET balance = balance - 100, version = version + 1
WHERE id = 1 AND version = 5;

-- Check rows affected
-- If 0 rows: Concurrent modification detected - retry or fail
-- If 1 row: Success

-- Application code pattern:
-- 1. Read entity with version
-- 2. Modify in application
-- 3. UPDATE with WHERE version = read_version
-- 4. If affected_rows = 0, throw OptimisticLockException
```

### Optimistic Locking with Timestamp

```sql
-- Using timestamp instead of version
ALTER TABLE accounts ADD COLUMN updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

-- Read
SELECT id, balance, updated_at FROM accounts WHERE id = 1;
-- Returns: updated_at = '2024-01-15 10:30:00'

-- Update with timestamp check
UPDATE accounts
SET balance = balance - 100, updated_at = CURRENT_TIMESTAMP
WHERE id = 1 AND updated_at = '2024-01-15 10:30:00';
```

## Deadlock Prevention

### Access Order Strategy

Always access tables and rows in consistent order.

```sql
-- BAD: Inconsistent order causes deadlocks
-- Transaction 1:
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Lock A
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Wait for B
COMMIT;

-- Transaction 2 (simultaneous):
BEGIN;
    UPDATE accounts SET balance = balance - 50 WHERE id = 2;   -- Lock B
    UPDATE accounts SET balance = balance + 50 WHERE id = 1;   -- Wait for A (DEADLOCK!)
COMMIT;

-- GOOD: Consistent order (always lower ID first)
-- Transaction 1:
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Lock A
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Lock B
COMMIT;

-- Transaction 2:
BEGIN;
    UPDATE accounts SET balance = balance + 50 WHERE id = 1;   -- Wait for A
    UPDATE accounts SET balance = balance - 50 WHERE id = 2;   -- Then lock B
COMMIT;
```

### Lock All Resources Upfront

```sql
-- Lock all needed rows at transaction start
BEGIN;
    -- Lock both accounts immediately
    SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;

    -- Now safe to update in any order
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### Short Transactions

```sql
-- BAD: Long transaction holding locks
BEGIN;
    SELECT * FROM orders WHERE id = 1 FOR UPDATE;
    -- Long processing...
    -- HTTP call to external service (slow!)
    -- More processing...
    UPDATE orders SET status = 'completed' WHERE id = 1;
COMMIT;

-- GOOD: Minimize lock duration
-- Do processing outside transaction
-- Then quick transaction:
BEGIN;
    UPDATE orders SET status = 'completed' WHERE id = 1;
COMMIT;
```

### Deadlock Handling

```sql
-- PostgreSQL: Retry on serialization failure
DO $$
DECLARE
    retry_count INT := 0;
    max_retries INT := 3;
BEGIN
    LOOP
        BEGIN
            -- Your transaction logic
            UPDATE accounts SET balance = balance - 100 WHERE id = 1;
            UPDATE accounts SET balance = balance + 100 WHERE id = 2;
            EXIT;  -- Success, exit loop
        EXCEPTION
            WHEN serialization_failure OR deadlock_detected THEN
                IF retry_count >= max_retries THEN
                    RAISE;
                END IF;
                retry_count := retry_count + 1;
                PERFORM pg_sleep(0.1 * retry_count);  -- Backoff
        END;
    END LOOP;
END $$;
```

## Transaction Patterns

### Unit of Work Pattern

```sql
-- All operations succeed or all fail
BEGIN;
    -- Create order
    INSERT INTO orders (customer_id, total) VALUES (123, 100.00)
    RETURNING order_id INTO v_order_id;

    -- Create order items
    INSERT INTO order_items (order_id, product_id, quantity)
    VALUES (v_order_id, 456, 2);

    -- Update inventory
    UPDATE products SET stock = stock - 2 WHERE product_id = 456;

    -- All succeeded
COMMIT;

-- If any fails, all roll back
```

### Savepoints for Partial Rollback

```sql
BEGIN;
    INSERT INTO orders (customer_id, total) VALUES (123, 100.00);

    SAVEPOINT sp_inventory;

    UPDATE products SET stock = stock - 2 WHERE product_id = 456;

    -- If inventory update fails, can rollback just that part
    IF some_condition THEN
        ROLLBACK TO SAVEPOINT sp_inventory;
        -- Order still exists, but inventory not updated
    END IF;

COMMIT;
```

### Read-Only Transactions

```sql
-- PostgreSQL: Explicitly read-only
BEGIN READ ONLY;
    SELECT * FROM accounts;
    SELECT * FROM transactions;
    -- INSERT/UPDATE/DELETE not allowed
COMMIT;

-- Benefits:
-- - Optimization hints to database
-- - No write locks acquired
-- - Consistent snapshot for reporting
```

### Batch Operations

```sql
-- Process in batches to reduce lock duration
DO $$
DECLARE
    batch_size INT := 1000;
    affected INT := 1;
BEGIN
    WHILE affected > 0 LOOP
        UPDATE orders
        SET status = 'archived'
        WHERE order_id IN (
            SELECT order_id FROM orders
            WHERE status = 'completed'
                AND created_at < CURRENT_DATE - INTERVAL '1 year'
            LIMIT batch_size
            FOR UPDATE SKIP LOCKED
        );

        GET DIAGNOSTICS affected = ROW_COUNT;

        -- Commit each batch
        COMMIT;
    END LOOP;
END $$;
```

## Platform-Specific Features

### PostgreSQL Advisory Locks

```sql
-- Application-level locks (not tied to rows)
-- Lock across multiple tables/operations

-- Session-level advisory lock
SELECT pg_advisory_lock(123);  -- Blocks until acquired
-- ... do work ...
SELECT pg_advisory_unlock(123);

-- Transaction-level advisory lock (auto-released at commit)
SELECT pg_advisory_xact_lock(123);

-- Non-blocking try
SELECT pg_try_advisory_lock(123);  -- Returns true/false immediately

-- Use case: Distributed coordination, singleton jobs
```

### SQL Server Transaction Hints

```sql
-- Set lock timeout
SET LOCK_TIMEOUT 5000;  -- 5 seconds

-- Snapshot isolation (MVCC-like)
ALTER DATABASE MyDB SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

-- Read committed snapshot
ALTER DATABASE MyDB SET READ_COMMITTED_SNAPSHOT ON;
```

### MySQL InnoDB Specific

```sql
-- Gap locking in REPEATABLE READ
-- InnoDB locks gaps between index records to prevent phantom reads

-- Disable gap locking for single row
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- Only locks row with id=1

-- Enable gap locking (range query)
SELECT * FROM accounts WHERE id BETWEEN 1 AND 10 FOR UPDATE;  -- Locks gap

-- Check for deadlocks
SHOW ENGINE INNODB STATUS;
```

## Concurrency Control Patterns

### Queue Processing Pattern

```sql
-- Safe concurrent queue processing
-- Worker 1 and Worker 2 can run simultaneously

-- Each worker grabs unlocked tasks
BEGIN;
    SELECT * FROM task_queue
    WHERE status = 'pending'
    ORDER BY created_at
    FOR UPDATE SKIP LOCKED
    LIMIT 1;

    -- Process the task...

    UPDATE task_queue SET status = 'completed' WHERE id = ?;
COMMIT;
```

### Inventory Reservation Pattern

```sql
-- Prevent overselling

-- Check and reserve atomically
BEGIN;
    -- Lock the product row
    SELECT stock FROM products WHERE id = 123 FOR UPDATE;

    -- Check availability
    IF stock >= requested_quantity THEN
        UPDATE products SET stock = stock - requested_quantity WHERE id = 123;
        -- Create reservation record
        INSERT INTO reservations (product_id, quantity, expires_at)
        VALUES (123, requested_quantity, CURRENT_TIMESTAMP + INTERVAL '15 minutes');
    ELSE
        RAISE EXCEPTION 'Insufficient stock';
    END IF;
COMMIT;
```

### Account Transfer Pattern

```sql
-- Safe money transfer
CREATE OR REPLACE FUNCTION transfer_money(
    from_account INT,
    to_account INT,
    amount NUMERIC
) RETURNS VOID AS $$
BEGIN
    -- Lock accounts in consistent order (lower ID first)
    PERFORM * FROM accounts
    WHERE id IN (from_account, to_account)
    ORDER BY id
    FOR UPDATE;

    -- Verify sufficient funds
    IF (SELECT balance FROM accounts WHERE id = from_account) < amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;

    -- Perform transfer
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;

    -- Log transaction
    INSERT INTO transactions (from_account, to_account, amount, created_at)
    VALUES (from_account, to_account, amount, CURRENT_TIMESTAMP);
END;
$$ LANGUAGE plpgsql;
```

## Monitoring and Troubleshooting

### Check Current Locks

```sql
-- PostgreSQL: View current locks
SELECT
    l.locktype,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    l.pid,
    a.query,
    a.state,
    age(clock_timestamp(), a.query_start) AS query_duration
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted
    OR l.relation IS NOT NULL
ORDER BY a.query_start;

-- SQL Server: View blocking
SELECT
    blocking.session_id AS blocking_session,
    blocked.session_id AS blocked_session,
    blocked.wait_type,
    blocked.wait_time,
    blocking_text.text AS blocking_query,
    blocked_text.text AS blocked_query
FROM sys.dm_exec_requests blocked
JOIN sys.dm_exec_sessions blocking ON blocked.blocking_session_id = blocking.session_id
CROSS APPLY sys.dm_exec_sql_text(blocked.sql_handle) blocked_text
CROSS APPLY sys.dm_exec_sql_text(blocking.most_recent_sql_handle) blocking_text;
```

### Deadlock Analysis

```sql
-- PostgreSQL: Enable deadlock logging
SET deadlock_timeout = '1s';
SET log_lock_waits = on;

-- Check logs for deadlock information

-- SQL Server: Enable trace flag for deadlock logging
DBCC TRACEON(1204, 1222, -1);
```

## Output Format

When invoked, provide:
1. Recommended isolation level with rationale
2. Locking strategy (pessimistic vs optimistic)
3. Deadlock prevention approach
4. Code examples for the specific use case
5. Monitoring queries for troubleshooting

## Cross-References

- **Query Optimization (query-optimization skill):** Transaction performance
- **Performance Tuning (performance-tuning skill):** Lock contention analysis
- **SQL Query Patterns (sql-query-patterns skill):** Query patterns within transactions
