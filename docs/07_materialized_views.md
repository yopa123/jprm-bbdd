# Materialized Views

Materialized Views in jprm-BBDD precompute and persist the results of resource-intensive analytical queries into local columnar storage. Subsequent queries against the view execute with sub-millisecond latency.

---

## 1. Creating a Materialized View

### Static View
Precomputed once at creation time:
```sql
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT 
    DATE_TRUNC('day', order_date) AS sale_day,
    region,
    COUNT(*) AS total_transactions,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY DATE_TRUNC('day', order_date), region;
```

---

## 2. Automated Scheduled Refresh

jprm-BBDD features an internal background scheduler capable of refreshing views automatically without relying on external cron jobs or orchestrators:

```sql
-- Refresh every 30 seconds
CREATE MATERIALIZED VIEW mv_live_metrics 
REFRESH EVERY 30 SECONDS AS
SELECT 
    device_type,
    AVG(cpu_usage) AS avg_cpu,
    MAX(memory_usage) AS peak_mem
FROM telemetry
GROUP BY device_type;

-- Refresh every 10 minutes
CREATE MATERIALIZED VIEW mv_regional_summary 
REFRESH EVERY 10 MINUTES AS
SELECT branch_id, SUM(total) AS revenue 
FROM branch_sales 
GROUP BY branch_id;
```

---

## 3. Manual Refresh

To trigger an immediate recalculation of a view outside of its scheduled window:

```sql
REFRESH MATERIALIZED VIEW mv_daily_sales;
```

---

## 4. Dropping a Materialized View

```sql
DROP MATERIALIZED VIEW mv_daily_sales;
```

---

## Performance Considerations

- **Atomic Swap**: During background refresh operations, queries targeting the view continue reading the previous consistent dataset until the new computation finishes. Once complete, an atomic reference swap makes the fresh data available with zero reader lock contention.
- **Catalog Visibility**: Materialized views appear alongside base tables in `SHOW TABLES` and can be queried using standard `SELECT` statements.
