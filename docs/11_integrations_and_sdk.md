# Integrations & SDKs

jprm-BBDD is natively accessible over HTTP using standard JSON or ClickHouse wire-compatible headers, making it easy to integrate with any programming language without proprietary binary drivers.

---

## 1. Official Python SDK (`jprm-bbdd`)

A native, zero-dependency client library providing both synchronous and asynchronous (`asyncio` / FastAPI) connections, a fluent query builder, and automatic parameter interpolation.

### Installation
```bash
pip install jprm-bbdd
```

### Basic Query Execution
```python
from jprm-bbdd import jprm-BBDDClient

client = jprm-BBDDClient(
    host="127.0.0.1",
    port=8123,
    token="app_servicios_sec_token_2026"
)

# Direct SQL query with safe parameters
result = client.query(
    "SELECT id, name, balance FROM customers WHERE balance > :min_balance",
    min_balance=1500.0
)

for row in result:
    print(f"Customer {row['id']}: {row['name']} (Balance: ${row['balance']:.2f})")
```

### Fluent Query Builder & Spatial Filtering
```python
# Geospatial radius filter (5 km around Santiago coordinates)
nearby_stores = (
    client.table("stores")
    .select("id", "name", "lat", "lon")
    .within_radius("lat", "lon", -33.4489, -70.6693, 5000.0)
    .order_by("name")
    .limit(10)
    .execute()
)
```

### Asynchronous Client for FastAPI
```python
from jprm-bbdd import Asyncjprm-BBDDClient

client = Asyncjprm-BBDDClient(token="app_servicios_sec_token_2026")

async def get_active_orders():
    result = await client.query("SELECT * FROM orders WHERE status = 'PENDING'")
    return result.rows
```

---

## 2. Node.js / TypeScript Example

```typescript
async function queryjprm-BBDD(sql: string, token?: string) {
  const headers: Record<string, string> = {
    'Content-Type': 'text/plain',
  };
  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }

  const response = await fetch('http://localhost:8123/', {
    method: 'POST',
    headers,
    body: sql,
  });

  if (!response.ok) {
    const errorText = await response.text();
    throw new Error(`jprm-BBDD Error (${response.status}): ${errorText}`);
  }

  return response.json();
}

// Example usage
const customers = await queryjprm-BBDD('SELECT * FROM customers LIMIT 5');
console.log(customers);
```

---

## 3. PHP Example (cURL / Guzzle)

```php
<?php
function queryjprm-BBDD(string $sql, string $token): array {
    $ch = curl_init("http://127.0.0.1:8123/");
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $sql);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        "Content-Type: text/plain",
        "Authorization: Bearer " . $token
    ]);

    $response = curl_exec($ch);
    $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($status >= 400) {
        throw new Exception("jprm-BBDD Query Failed: " . $response);
    }

    return json_decode($response, true);
}

$rows = queryjprm-BBDD("SELECT * FROM customers WHERE balance > 5000", "app_token");
print_r($rows);
```

---

## 4. cURL / Shell CLI

```bash
# Query with Bearer token authentication
curl -s -X POST http://localhost:8123/ \
  -H "Authorization: Bearer app_servicios_sec_token_2026" \
  -d "SELECT id, name, balance FROM customers WHERE balance > 1000"
```
