# SQL Reference & Data Types

jprm-BBDD supports standard ANSI SQL syntax for Data Definition Language (DDL) and Data Manipulation Language (DML), powered by a vectorized execution engine.

---

## Supported Data Types

| SQL Type | Physical Storage | Description |
| :--- | :--- | :--- |
| `INT`, `INTEGER` | 64-bit Signed Integer | Standard numeric identifiers and counts |
| `BIGINT` | 64-bit Signed Integer | Large integers |
| `FLOAT`, `REAL` | 32-bit Floating Point | Single precision floating point |
| `DOUBLE`, `DOUBLE PRECISION` | 64-bit Floating Point | Double precision coordinates, metrics |
| `VARCHAR`, `TEXT`, `STRING` | Variable-length UTF-8 | Textual strings |
| `BOOLEAN`, `BOOL` | 1-bit Boolean | `TRUE` / `FALSE` |
| `TIMESTAMP` | 64-bit Nanosecond Timestamp | ISO-8601 timestamps and calendar dates |

---

## Data Definition Language (DDL)

### Create Table
```sql
CREATE TABLE customers (
    id INT,
    name VARCHAR,
    email VARCHAR,
    balance DOUBLE,
    is_active BOOLEAN,
    created_at TIMESTAMP
);
```

### Drop Table
```sql
DROP TABLE customers;
```

### Show Tables
Lists all persistent tables and active external federations in the database catalog:
```sql
SHOW TABLES;
```

---

## Data Manipulation Language (DML)

### Insert Values
```sql
INSERT INTO customers VALUES 
    (1, 'Alice Corp', 'alice@corp.com', 4500.50, TRUE, '2026-01-15T08:00:00Z'),
    (2, 'Beta Ltd', 'beta@corp.com', 1200.00, FALSE, '2026-02-01T12:30:00Z');
```

### Select & Filtering
```sql
SELECT id, name, balance 
FROM customers 
WHERE balance > 1000.0 AND is_active = TRUE;
```

### Vectorized Aggregations & Grouping
```sql
SELECT 
    is_active, 
    COUNT(*) AS total_clients, 
    SUM(balance) AS total_balance, 
    AVG(balance) AS avg_balance,
    MIN(balance) AS min_balance,
    MAX(balance) AS max_balance
FROM customers
GROUP BY is_active
ORDER BY total_balance DESC;
```

### Sorting & Pagination
```sql
SELECT * FROM customers 
ORDER BY balance DESC 
LIMIT 50 OFFSET 100;
```

---

## Analytical System Introspection (`EXPLAIN`)

jprm-BBDD provides transparent plan analysis showing execution split between the tabular SIMD engine and the graph engine:

```sql
EXPLAIN SELECT * FROM customers WHERE balance > 2000;
```
Response includes execution duration in microseconds, rows evaluated, and hardware SIMD vectorization confirmation.
