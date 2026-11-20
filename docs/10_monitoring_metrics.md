# Monitoring & Observability

jprm-BBDD exposes telemetry endpoints for healthchecks, operational status, and Prometheus metrics.

---

## 1. Healthcheck (`GET /ping`)

Lightweight probe for container orchestrators (Kubernetes, Nomad, Docker Compose):

```bash
curl -i http://localhost:8123/ping
```
**Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

ok
```

---

## 2. Server Status & Diagnostics (`GET /status`)

Returns operational diagnostics including uptime, memory metrics, table counts, and active background ingestion jobs:

```bash
curl -s http://localhost:8123/status
```

**JSON Output Example:**
```json
{
  "version": "1.0.0-alpha",
  "status": "online",
  "uptime_seconds": 14280,
  "tables_registered": 18,
  "materialized_views": 3,
  "active_background_jobs": 0
}
```

---

## 3. Prometheus Metrics Endpoint (`GET /metrics`)

jprm-BBDD serves metrics in Prometheus text exposition format on port `8123`:

```bash
curl -s http://localhost:8123/metrics
```

### Key Metrics Exported

| Metric | Type | Description |
| :--- | :--- | :--- |
| `jprm-bbdd_queries_total` | Counter | Cumulative query executions split by status (`success`, `error`) |
| `jprm-bbdd_query_duration_seconds` | Histogram | Latency distribution of processed queries |
| `jprm-bbdd_tables_total` | Gauge | Number of active tabular partitions |
| `jprm-bbdd_materialized_views_total` | Gauge | Count of registered materialized views |
| `jprm-bbdd_active_background_jobs` | Gauge | Count of running asynchronous ingestion/sync tasks |

### Sample Prometheus Scrape Configuration
```yaml
scrape_configs:
  - job_name: "jprm-bbdd"
    scrape_interval: 10s
    static_configs:
      - targets: ["localhost:8123"]
```
