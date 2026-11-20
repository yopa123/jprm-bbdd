# Backup & Disaster Recovery

jprm-BBDD features non-blocking, atomic hot backups. Backups capture consistent state snapshots across both tabular columnar partitions and the topological graph database without locking concurrent read operations.

---

## 1. Creating a Hot Backup

Execute the `BACKUP DATABASE` command specifying the absolute or relative target directory:

```sql
BACKUP DATABASE TO '/var/backups/jprm-bbdd/snapshot_2026_03_01';
```

---

## 2. Backup Contents & Verification

The operation writes copies of all active storage components and emits a verification manifest:

```
snapshot_2026_03_01/
├── backup_manifest.json    # Cryptographic SHA-256 manifest & metadata
├── graph.redb              # Graph & security state
└── tables/                 # Columnar Parquet table files
    ├── customers.parquet
    └── telemetry.parquet
```

### Verification Manifest (`backup_manifest.json`)
The manifest includes timestamp metadata and cryptographic digests for auditing:

```json
{
  "timestamp": "2026-03-01T14:32:00Z",
  "version": "1.0.0-alpha",
  "files": [
    {
      "path": "tables/customers.parquet",
      "size_bytes": 1428570,
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "path": "graph.redb",
      "size_bytes": 8388608,
      "sha256": "4b227777d4dd1fc61c6f884f48641d02b4d121d3fd328cb08b5531fcacdabf8a"
    }
  ]
}
```

---

## 3. Restoring from Backup

To restore an instance:
1. Stop the `jprm-bbdd-server` service.
2. Replace the contents of `data_dir` with the snapshot directory.
3. Restart `jprm-bbdd-server`. All registered tables and graph connections will be mounted automatically upon startup.
