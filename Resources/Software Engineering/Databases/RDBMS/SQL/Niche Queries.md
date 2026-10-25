# Niche Queries

---

# Recursive Queries (CTEs)

A **recursive CTE** (Common Table Expression) lets a query reference itself, making it possible to traverse hierarchical or graph-like data entirely in SQL — no application-side looping required.

Supported in: PostgreSQL, MySQL 8+, SQL Server, SQLite 3.35+, MariaDB 10.2+.

## Syntax

```sql
WITH RECURSIVE cte_name AS (
    -- 1. Anchor member: the base/starting rows
    SELECT ...
    FROM   some_table
    WHERE  <base condition>

    UNION ALL

    -- 2. Recursive member: joins CTE back to itself
    SELECT ...
    FROM   some_table
    JOIN   cte_name ON <join condition>
)
SELECT * FROM cte_name;
```

- **Anchor member** — executed once, seeds the result set.
- **Recursive member** — executed repeatedly, each time joining against the *previous iteration's* output, until it produces zero rows.
- `UNION ALL` is the standard connector (use `UNION` only if you need duplicate elimination per step — it's slower).
- Most databases impose a **max recursion depth** (e.g. SQL Server default: 100; PostgreSQL default: unlimited but configurable via `max_execution_depth`).

## Example Use Case — Employee Org Chart

### Tables

```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(100),
    manager_id INT REFERENCES employees(id)  -- NULL for the CEO
);

INSERT INTO employees VALUES
(1, 'Alice',   NULL),   -- CEO
(2, 'Bob',     1),      -- reports to Alice
(3, 'Carol',   1),      -- reports to Alice
(4, 'Dave',    2),      -- reports to Bob
(5, 'Eve',     2),      -- reports to Bob
(6, 'Frank',   3);      -- reports to Carol
```

### Goal: find all reports under Bob (id = 2), at any depth

```sql
WITH RECURSIVE org_chart AS (
    -- Anchor: start at Bob
    SELECT id, name, manager_id, 1 AS depth
    FROM   employees
    WHERE  id = 2

    UNION ALL

    -- Recursive: find everyone whose manager is already in the CTE
    SELECT e.id, e.name, e.manager_id, oc.depth + 1
    FROM   employees e
    JOIN   org_chart  oc ON e.manager_id = oc.id
)
SELECT id, name, depth
FROM   org_chart
ORDER  BY depth, name;
```

### Result

| id | name  | depth |
|----|-------|-------|
| 2  | Bob   | 1     |
| 4  | Dave  | 2     |
| 5  | Eve   | 2     |

## Other Common Use Cases

- **Bill of materials** — explode a product into its nested components.
- **Folder/file trees** — walk a directory hierarchy stored in a parent-child table.
- **Graph reachability** — find all nodes reachable from a starting node (add a `visited` guard to prevent cycles).
- **Generating sequences** — produce a date range or integer series without a helper table.

## Performance vs Server-Side Logic

| Dimension | Recursive CTE | Server-side loop (app code / stored proc) |
|---|---|---|
| **Round trips** | 1 query | 1 per depth level (N+1 risk) |
| **Network overhead** | Minimal — data stays in DB | High if each level fetches rows over the wire |
| **Query planner** | Can optimize the full traversal | Planner sees only one level at a time |
| **Depth limit** | Hard cap set by DB config | Only stack/memory limits |
| **Cycle safety** | Must be handled manually (track visited IDs) | Easier to short-circuit in code |
| **Debuggability** | Query plans can be opaque for deep recursion | Step-through debugging available |
| **Portability** | Standard SQL (ISO/IEC 9075), widely supported | Language/ORM specific |

**Rule of thumb:** prefer a recursive CTE when the hierarchy is shallow-to-medium depth (< a few hundred levels) and the traversal logic is simple. Switch to server-side logic (or a dedicated graph DB) when you need cycle detection, complex branching logic per node, or the depth is unbounded and unpredictable.

### Tip: guard against infinite loops

If your data could contain cycles (e.g. a graph rather than a pure tree), track visited IDs in an array:

```sql
WITH RECURSIVE graph_walk AS (
    SELECT id, ARRAY[id] AS visited
    FROM   nodes
    WHERE  id = <start>

    UNION ALL

    SELECT n.id, gw.visited || n.id
    FROM   nodes        n
    JOIN   edges        e  ON e.to_id   = n.id
    JOIN   graph_walk   gw ON e.from_id = gw.id
    WHERE  NOT n.id = ANY(gw.visited)   -- cycle guard
)
SELECT * FROM graph_walk;
```

---

# Window Functions

> **Stub** — fill in details when you read about this.

Perform calculations across a set of rows related to the current row, without collapsing them into a single group. Uses `OVER (PARTITION BY ... ORDER BY ...)`.

Key functions: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, `SUM/AVG OVER`.

---

# PIVOT / UNPIVOT

> **Stub** — fill in details when you read about this.

Transform rows into columns (PIVOT) or columns into rows (UNPIVOT). Native syntax varies by DB (SQL Server has `PIVOT`; PostgreSQL uses `crosstab` or conditional aggregation).

---

# LATERAL Joins

> **Stub** — fill in details when you read about this.

Allows a subquery in the `FROM` clause to reference columns from preceding tables in the same `FROM` clause — like a correlated subquery but usable as a table. PostgreSQL/MySQL: `LATERAL`; SQL Server: `CROSS APPLY` / `OUTER APPLY`.

---

# MERGE (Upsert)

> **Stub** — fill in details when you read about this.

Atomically `INSERT`, `UPDATE`, or `DELETE` rows in a target table based on a match against a source. Useful for upsert patterns. Syntax: `MERGE INTO target USING source ON condition WHEN MATCHED ... WHEN NOT MATCHED ...`.

---

# Grouping Sets / ROLLUP / CUBE

> **Stub** — fill in details when you read about this.

Extensions of `GROUP BY` that produce multiple levels of aggregation in one query. `ROLLUP` gives subtotals along a hierarchy; `CUBE` gives all possible combinations; `GROUPING SETS` lets you specify exactly which groupings you want.
