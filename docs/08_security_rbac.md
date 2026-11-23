# Security & Role-Based Access Control (RBAC)

jprm-BBDD implements persistent Role-Based Access Control (RBAC) and Bearer token authentication to secure data access in multi-tenant environments.

---

## Authentication Mechanism

Clients authenticate by supplying a secret token via:
1. **HTTP Authorization Header**: `Authorization: Bearer <token>`
2. **ClickHouse Wire Compatibility Header**: `X-ClickHouse-Key: <token>`

---

## User Management Commands

User metadata and permission grants are stored transactions within the internal ACID security catalog.

### 1. Create User

**Standard User:**
```sql
CREATE USER analytics_svc IDENTIFIED BY 'svc_token_abc123_2026';
```

**Administrator User:**
```sql
CREATE USER sysadmin IDENTIFIED BY 'admin_token_xyz999_2026' ADMIN;
```

---

### 2. Grant Permissions

Privileges can be assigned on specific tables or globally using the wildcard (`*`):

| Privilege | Scope | Description |
| :--- | :--- | :--- |
| `SELECT` | Table / `*` | Read access to table records |
| `INSERT` | Table / `*` | Append records via `INSERT` |
| `CREATE` | Global (`*`) | Create new tables or materialized views |
| `DROP` | Table / `*` | Drop tables or views |
| `IMPORT` | Global (`*`) | Ingest data from external databases/files |

**Examples:**
```sql
-- Grant read-only access to a specific table
GRANT SELECT ON customers TO analytics_svc;

-- Grant read and insert privileges on all tables
GRANT SELECT ON * TO analytics_svc;
GRANT INSERT ON * TO analytics_svc;

-- Grant ingestion rights
GRANT IMPORT ON * TO analytics_svc;
```

---

### 3. Revoke Permissions

```sql
REVOKE INSERT ON * FROM analytics_svc;
```

---

### 4. Inspect Users & Grants

```sql
SHOW USERS;
```

---

### 5. Drop User

```sql
DROP USER analytics_svc;
```
