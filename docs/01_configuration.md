# Configuration Guide (`jprm-bbdd.toml`)

jprm-BBDD can be configured using a declarative `jprm-bbdd.toml` file located in the working directory or supplied via the `--config` command-line flag.

```bash
# Launch with custom configuration file
./jprm-bbdd-server --config /etc/jprm-bbdd/jprm-bbdd.toml
```

---

## Configuration Reference

A complete `jprm-bbdd.toml` template is shown below:

```toml
[server]
# Network interface to bind (0.0.0.0 for public access, 127.0.0.1 for local only)
host = "0.0.0.0"

# Primary HTTP & Wire query port
port = 8123

# Path to persistent database directory
data_dir = "./jprm_bbdd_data"

# Optional: preload embedded demo dataset for evaluation
load_demo = false

[security]
# Enable authentication requirements
enabled = true

# Default administrator token (overridden once root user is re-keyed)
default_admin_token = "jprm-bbdd_root_secret_token_2026"

[engine]
# Query execution timeout in seconds (0 = unlimited)
query_timeout_seconds = 60

# Max memory buffer per query in Megabytes
max_memory_mb = 4096
```

---

## Command Line Arguments

Command line flags take precedence over settings defined in `jprm-bbdd.toml`:

| Flag | Description | Default |
| :--- | :--- | :--- |
| `--config <path>` | Path to custom configuration file | `jprm-bbdd.toml` |
| `--host <address>` | IP address to bind server socket | `0.0.0.0` |
| `--port <number>` | Port number for incoming traffic | `8123` |
| `--data-dir <path>` | Target storage directory | `./jprm_bbdd_data` |
| `--demo` | Preload analytical demo records | `false` |

---

## Directory Layout

Upon initialization, jprm-BBDD creates the following layout inside `data_dir`:

```
jprm_bbdd_data/
├── tables/               # Columnar Parquet table files
│   ├── users.parquet
│   └── events.parquet
└── graph.redb            # Transactional B-Tree key-value store for graphs & metadata
```

- **`tables/`**: Persisted columnar partitions. Files can be inspected or read by analytical tooling supporting Apache Parquet.
- **`graph.redb`**: ACID transactional store maintaining graph adjacency lists, user security catalogs, and materialized view metadata.
