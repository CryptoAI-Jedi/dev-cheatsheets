# SQL Cheatsheet

> Query language fundamentals: SELECT through transactions. Written PostgreSQL-first; dialect differences flagged where they bite. SQLite specifics are queued for their own sheet.

---

## Table of Contents
- [Querying Data](#querying-data)
- [Filtering](#filtering)
- [Aggregations](#aggregations)
- [Joins](#joins)
- [Modifying Data](#modifying-data)
- [Table Management](#table-management)
- [Indexes & Performance](#indexes--performance)
- [Useful Functions](#useful-functions)
- [Subqueries & CTEs](#subqueries--ctes)
- [Transactions](#transactions)
- [Tips & Gotchas](#tips--gotchas)

---

## QUERYING DATA

```sql
SELECT * FROM table;                          -- All columns
SELECT col1, col2 FROM table;                 -- Specific columns
SELECT DISTINCT col FROM table;               -- Unique values only
SELECT * FROM table LIMIT 10;                 -- First 10 rows
SELECT * FROM table ORDER BY col ASC;         -- Sort ascending
SELECT * FROM table ORDER BY col DESC;        -- Sort descending
```

---

## FILTERING

```sql
SELECT * FROM table WHERE col = 'value';
SELECT * FROM table WHERE col != 'value';
SELECT * FROM table WHERE col > 100;
SELECT * FROM table WHERE col BETWEEN 1 AND 100;
SELECT * FROM table WHERE col IN ('a', 'b', 'c');
SELECT * FROM table WHERE col IS NULL;        -- NOT col = NULL (see Gotchas)
SELECT * FROM table WHERE col IS NOT NULL;
SELECT * FROM table WHERE col LIKE 'Jo%';     -- Starts with "Jo"
SELECT * FROM table WHERE col LIKE '%son';    -- Ends with "son"
SELECT * FROM table WHERE col LIKE '%ohn%';   -- Contains "ohn"
```

---

## AGGREGATIONS

```sql
SELECT COUNT(*) FROM table;                   -- Row count
SELECT COUNT(DISTINCT col) FROM table;        -- Unique count
SELECT SUM(col) FROM table;
SELECT AVG(col) FROM table;
SELECT MIN(col), MAX(col) FROM table;

SELECT col, COUNT(*) FROM table GROUP BY col;

SELECT col, COUNT(*) FROM table
GROUP BY col
HAVING COUNT(*) > 5;                          -- Filter AFTER grouping (WHERE filters before)
```

---

## JOINS

```sql
-- INNER JOIN (only matching rows)
SELECT a.col, b.col
FROM table_a a
INNER JOIN table_b b ON a.id = b.a_id;

-- LEFT JOIN (all from left, matching from right; NULLs where no match)
SELECT a.col, b.col
FROM table_a a
LEFT JOIN table_b b ON a.id = b.a_id;

-- RIGHT JOIN (all from right)
SELECT a.col, b.col
FROM table_a a
RIGHT JOIN table_b b ON a.id = b.a_id;

-- FULL OUTER JOIN (all rows from both)
SELECT a.col, b.col
FROM table_a a
FULL OUTER JOIN table_b b ON a.id = b.a_id;
```

---

## MODIFYING DATA

```sql
INSERT INTO table (col1, col2) VALUES ('val1', 'val2');
UPDATE table SET col1 = 'new' WHERE col2 = 'target';   -- ALWAYS with WHERE (see Gotchas)
DELETE FROM table WHERE col = 'value';
TRUNCATE TABLE table;                         -- Delete all rows (fast; resets sequences)
```

---

## TABLE MANAGEMENT

```sql
CREATE TABLE users (
    id         SERIAL PRIMARY KEY,            -- PostgreSQL (MySQL: AUTO_INCREMENT)
    name       VARCHAR(100) NOT NULL,
    age        INT,
    created_at TIMESTAMP DEFAULT NOW()
);

ALTER TABLE users ADD COLUMN email VARCHAR(255);
ALTER TABLE users DROP COLUMN email;
ALTER TABLE users RENAME COLUMN name TO full_name;
DROP TABLE users;                             -- Delete entire table
DROP TABLE IF EXISTS users;                   -- Safe drop
```

---

## INDEXES & PERFORMANCE

```sql
CREATE INDEX idx_name ON table(col);          -- Speed up lookups on col
DROP INDEX idx_name;
EXPLAIN SELECT * FROM table WHERE col = 'x';  -- Show query plan
EXPLAIN ANALYZE SELECT ...;                   -- Plan + ACTUAL execution (runs the query!)
```

---

## USEFUL FUNCTIONS

```sql
NOW()                     -- Current timestamp
CURRENT_DATE              -- Today's date
UPPER(col), LOWER(col)    -- Case conversion
LENGTH(col)               -- String length
COALESCE(col, 'default')  -- First non-NULL value
CAST(col AS INTEGER)      -- Type casting
CONCAT(col1, ' ', col2)   -- Concatenate strings
ROUND(col, 2)             -- Round to decimal places
DATE_TRUNC('month', col)  -- Truncate to month (PostgreSQL)
```

---

## SUBQUERIES & CTEs

```sql
-- Subquery
SELECT * FROM table
WHERE id IN (SELECT id FROM other_table);

-- CTE (Common Table Expression) — readable, composable
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY col DESC) AS rn
    FROM table
)
SELECT * FROM ranked WHERE rn <= 10;
```

---

## TRANSACTIONS

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;                   -- Save changes
-- or
ROLLBACK;                 -- Undo everything since BEGIN
```

---

## TIPS & GOTCHAS

- **UPDATE/DELETE without WHERE hits every row** — the classic career moment. Habit: write the `SELECT` with the same WHERE first, check the rows, then convert to UPDATE/DELETE — inside a `BEGIN` so ROLLBACK exists.
- **`= NULL` is never true** — NULL compares as unknown; `WHERE col = NULL` returns zero rows silently. `IS NULL` / `IS NOT NULL`, always.
- **WHERE vs HAVING** — WHERE filters rows before grouping; HAVING filters groups after. Aggregate conditions (`COUNT(*) > 5`) can only live in HAVING.
- **LIKE case-sensitivity is dialect-dependent** — case-sensitive in PostgreSQL (use `ILIKE` for insensitive), insensitive by default in MySQL. Test in the DB you deploy on.
- **TRUNCATE ≠ big DELETE** — it resets sequences/auto-increment, may not fire triggers, and in some databases (MySQL) can't be rolled back. In doubt, `DELETE FROM table;` inside a transaction.
- **`EXPLAIN ANALYZE` executes the query** — on a SELECT that's fine; on UPDATE/DELETE it performs the write. Wrap in BEGIN/ROLLBACK when analyzing mutations.
- **Index the columns you filter and join on** — WHERE and ON columns are index candidates; `EXPLAIN` showing a sequential scan on a big table is the tell.
- **Dialect flags in this sheet** — `SERIAL`, `NOW()`, `DATE_TRUNC`, `FULL OUTER JOIN` are PostgreSQL-flavored; MySQL and SQLite each differ (SQLite has no RIGHT/FULL OUTER JOIN before 3.39, no ILIKE, dynamic typing). Sheet-level answer: the upcoming SQLite sheet.

---
*Last Updated: 2026-07*
