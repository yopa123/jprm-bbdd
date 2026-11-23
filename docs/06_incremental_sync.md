# Incremental Synchronization (CDC)

jprm-BBDD supports automated delta synchronization from external transactional databases using watermark columns (such as auto-incrementing sequential IDs or modification timestamps).

---

## Overview

Rather than dropping and re-importing multi-gigabyte tables, the `SYNC TABLE` command queries the highest watermark value currently recorded in the local table and fetches only modified or newly created rows from the remote source.

---

## Syntax

```sql
SYNC TABLE <local_table> 
FROM <POSTGRES | MYSQL | MSSQL | SQLITE> '<connection_string>' 
TABLE '<remote_table>' 
ON '<watermark_column>';
```

---

## Examples

### Timestamp-Based Change Tracking
Synchronizing an operational e-commerce orders table where the `updated_at` column reflects the last modification:

```sql
SYNC TABLE local_orders 
FROM POSTGRES 'postgres://readonly:secret@main-db:5432/ecommerce' 
TABLE 'orders' 
ON 'updated_at';
```

**Internal Operation:**
1. jprm-BBDD evaluates `SELECT MAX(updated_at) FROM local_orders;`.
2. Queries the remote database: `SELECT * FROM orders WHERE updated_at > :max_val;`.
3. Streams the delta into local columnar partitions without downtime.

### Sequence/ID-Based Append Sync
For immutable append-only ledgers, audit logs, or event streams:

```sql
SYNC TABLE event_logs 
FROM MYSQL 'mysql://app:secret@log-db:3306/audit' 
TABLE 'system_events' 
ON 'id';
```
