# Ingestion Engine & Zero-ETL Federation

jprm-BBDD eliminates the operational overhead of third-party ETL pipelines by providing built-in connectors to stream, convert, or federate data from external RDBMS engines and structured file formats directly into the database.

---

## 1. Bulk Ingestion (`IMPORT TABLE`)

The `IMPORT TABLE` command extracts data from an external source, infers schema types automatically, and writes the contents to local, compressed columnar Parquet storage.

### Supported External Engines
- `POSTGRES` (PostgreSQL)
- `MYSQL` (MySQL & MariaDB)
- `MSSQL` (Microsoft SQL Server)
- `SQLITE` (SQLite3 databases)
- `EXCEL` (Microsoft Excel `.xlsx`)
- `CSV` (Delimited text files)

---

### Ingestion from Relational Databases

**Importing an Entire Remote Table:**
```sql
IMPORT TABLE local_orders 
FROM POSTGRES 'postgres://user:pass@dbhost:5432/production' 
TABLE 'orders';
```

**Importing via a Custom SQL Query:**
```sql
IMPORT TABLE high_value_customers 
FROM MYSQL 'mysql://user:pass@dbhost:3306/crm' 
QUERY 'SELECT id, name, total_spent FROM users WHERE total_spent > 10000';
```

**Asynchronous Background Ingestion:**
For massive tables (10M+ rows), append `BACKGROUND` to decouple the ingestion task:
```sql
IMPORT TABLE historical_telemetry 
FROM MSSQL 'sqlserver://user:pass@dbhost:1433?database=telemetry' 
TABLE 'raw_metrics' 
BACKGROUND;
```
Progress and execution state can be monitored via `GET /status`.

---

### Ingestion from Files

**From Microsoft Excel Spreadsheet:**
```sql
IMPORT TABLE monthly_targets 
FROM EXCEL '/data/financial_plan_2026.xlsx' 
SHEET 'Q1_Targets';
```

**From Delimited CSV File:**
```sql
IMPORT TABLE country_codes 
FROM CSV '/data/iso_countries.csv';
```

---

## 2. Federated Zero-ETL (`CREATE EXTERNAL TABLE`)

When data should remain strictly in the external system without consuming local storage, `CREATE EXTERNAL TABLE` creates a live virtual link into the remote database. Queries executed against external tables are pushed down in real-time.

```sql
-- Link a remote PostgreSQL table
CREATE EXTERNAL TABLE remote_inventory 
FROM POSTGRES 'postgres://user:pass@pg-cluster:5432/wms' 
TABLE 'warehouse_stock';

-- Link via remote parameterized view
CREATE EXTERNAL TABLE remote_audit_log 
FROM MSSQL 'sqlserver://user:pass@sql-srv:1433?database=audit' 
QUERY 'SELECT id, action, operator, created_at FROM security_events';
```

Federated tables appear in `SHOW TABLES` and can be joined seamlessly with local columnar tables.
