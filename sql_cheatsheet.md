# sql_cheatsheet

# QUERYING DATA

---

SELECT * FROM table;                          → All columns
SELECT col1, col2 FROM table;                 → Specific columns
SELECT DISTINCT col FROM table;               → Unique values only
SELECT * FROM table LIMIT 10;                 → First 10 rows
SELECT * FROM table ORDER BY col ASC;         → Sort ascending
SELECT * FROM table ORDER BY col DESC;        → Sort descending

---

## FILTERING

SELECT * FROM table WHERE col = 'value';
SELECT * FROM table WHERE col != 'value';
SELECT * FROM table WHERE col > 100;
SELECT * FROM table WHERE col BETWEEN 1 AND 100;
SELECT * FROM table WHERE col IN ('a', 'b', 'c');
SELECT * FROM table WHERE col IS NULL;
SELECT * FROM table WHERE col IS NOT NULL;
SELECT * FROM table WHERE col LIKE 'Jo%';     → Starts with "Jo"
SELECT * FROM table WHERE col LIKE '%son';    → Ends with "son"
SELECT * FROM table WHERE col LIKE '%ohn%';   → Contains "ohn"

---

## AGGREGATIONS

SELECT COUNT(*) FROM table;                   → Row count
SELECT COUNT(DISTINCT col) FROM table;        → Unique count
SELECT SUM(col) FROM table;
SELECT AVG(col) FROM table;
SELECT MIN(col), MAX(col) FROM table;
SELECT col, COUNT(*) FROM table GROUP BY col;
SELECT col, COUNT(*) FROM table
GROUP BY col HAVING COUNT(*) > 5;           → Filter after GROUP BY

---

## JOINS

- - INNER JOIN (only matching rows)
SELECT a.col, b.col FROM table_a a
INNER JOIN table_b b ON [a.id](http://a.id/) = b.a_id;
- - LEFT JOIN (all from left, matching from right)
SELECT a.col, b.col FROM table_a a
LEFT JOIN table_b b ON [a.id](http://a.id/) = b.a_id;
- - RIGHT JOIN (all from right)
SELECT a.col, b.col FROM table_a a
RIGHT JOIN table_b b ON [a.id](http://a.id/) = b.a_id;
- - FULL OUTER JOIN (all rows from both)
SELECT a.col, b.col FROM table_a a
FULL OUTER JOIN table_b b ON [a.id](http://a.id/) = b.a_id;

---

## MODIFYING DATA

INSERT INTO table (col1, col2) VALUES ('val1', 'val2');
UPDATE table SET col1 = 'new' WHERE col2 = 'target';
DELETE FROM table WHERE col = 'value';
TRUNCATE TABLE table;                         → Delete all rows (fast)

---

## TABLE MANAGEMENT

CREATE TABLE users (
id   SERIAL PRIMARY KEY,
name VARCHAR(100) NOT NULL,
age  INT,
created_at TIMESTAMP DEFAULT NOW()
);

ALTER TABLE users ADD COLUMN email VARCHAR(255);
ALTER TABLE users DROP COLUMN email;
ALTER TABLE users RENAME COLUMN name TO full_name;
DROP TABLE users;                             → Delete entire table
DROP TABLE IF EXISTS users;                   → Safe drop

---

## INDEXES & PERFORMANCE

CREATE INDEX idx_name ON table(col);          → Speed up lookups
DROP INDEX idx_name;
EXPLAIN SELECT * FROM table WHERE col = 'x'; → Show query plan

---

## USEFUL FUNCTIONS

NOW()                    → Current timestamp
CURRENT_DATE             → Today's date
UPPER(col) / LOWER(col)  → Case conversion
LENGTH(col)              → String length
COALESCE(col, 'default') → Return first non-NULL value
CAST(col AS INTEGER)     → Type casting
CONCAT(col1, ' ', col2)  → Concatenate strings
ROUND(col, 2)            → Round to decimal places
DATE_TRUNC('month', col) → Truncate to month (PostgreSQL)

---

## SUBQUERIES & CTEs

- - Subquery
SELECT * FROM table WHERE id IN (SELECT id FROM other_table);
- - CTE (Common Table Expression)
WITH ranked AS (
SELECT *, ROW_NUMBER() OVER (ORDER BY col DESC) AS rn
FROM table
)
SELECT * FROM ranked WHERE rn <= 10;

---

## TRANSACTIONS

BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;                  → Save changes
ROLLBACK;                → Undo changes