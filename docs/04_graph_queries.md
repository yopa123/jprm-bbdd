# Graph Topologies & `GRAPH_PATH`

jprm-BBDD features an integrated graph topology engine backed by an ACID B-Tree index. This allows applications to maintain network edges (such as financial transactions, corporate structures, communication links, or dependency trees) and traverse them directly inside standard SQL queries without delegating to third-party graph databases.

---

## The `GRAPH_PATH` Table Function

The `GRAPH_PATH` function traverses adjacent nodes in the graph and returns an in-memory virtual relation containing a single column: `node_id` (`BIGINT`).

### Syntax
```sql
GRAPH_PATH(direction, source_node_id, relationship_type)
```

| Parameter | Type | Allowed Values / Description |
| :--- | :--- | :--- |
| `direction` | Identifier | `OUTGOING`, `INCOMING`, or `BOTH` |
| `source_node_id` | Integer | ID of the origin vertex |
| `relationship_type` | Integer | Numeric classification code representing edge type |

---

## Joining Graphs with Columnar Tables

Because `GRAPH_PATH` behaves as a first-class table source in SQL, it can be joined with persistent columnar tables to enrich relational data with topological traversals in a single round-trip:

```sql
SELECT 
    c.id, 
    c.name, 
    c.account_type, 
    c.balance,
    g.node_id AS connected_peer_id
FROM GRAPH_PATH(OUTGOING, 1052, 2) g
JOIN customers c ON g.node_id = c.id
WHERE c.balance > 5000.0;
```

---

## Traversal Directions

- **`OUTGOING`**: Finds all target nodes connected by directed edges originating from `source_node_id` (`source -> target`).
- **`INCOMING`**: Finds all source nodes with directed edges pointing toward `source_node_id` (`target -> source`).
- **`BOTH`**: Returns the union of all bidirectional connections associated with `source_node_id`.

---

## Performance Characteristics

- Adjacency lookups are executed in sub-microsecond time via low-level B-Tree key range scans.
- Results are materialized directly as zero-copy columnar arrays in memory, avoiding serialization overhead between the graph engine and the SQL query planner.
