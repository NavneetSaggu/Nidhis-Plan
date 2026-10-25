## Data Models

Data models are layered: each layer **hides the complexity of the layers below it** by providing a clean abstraction. The choice of data model affects how you think about the problem — and how easy it is to extend the application.
Eg: Consider how data is represented at different layers in a system

| Layer                | Representation |
| -------------------- | -------------- |
| Application          | Json / XML     |
| Database             | Data Structure |
| Infra / Kernel Level | Encoded Bytes  |


---

## Relational Model vs. Document Model

### Why Relational Dominated
- Born from business data processing (1970s), but expanded well beyond original scope.
- Most of the web today still runs on relational databases: social networking, ecommerce, SaaS, games, publishing.

### Rise of NoSQL
Driving forces behind NoSQL adoption:
- Need for **greater scalability** — very large datasets, high write throughput
- Preference for **free and open-source** software
- Need for **specialized query operations** not well supported by relational
- Frustration with the **restrictiveness** of relational schemas — desire for a more dynamic/expressive data model

> Likely future: **polyglot persistence** — relational used alongside non-relational stores depending on use case.

### The Object-Relational Mismatch (Impedance Mismatch)
- Most app code is **object-oriented**; SQL is relational — there's an awkward **translation layer** between the two. ORM frameworks reduce boilerplate but don't eliminate the disconnect. (For example, you'll need joins to fetch a single object's data if you have it normalized)
- Example: a LinkedIn-style résumé — positions, education, contact info are **one-to-many** from user → items.
  - **Traditional SQL**: separate tables (positions, education) with foreign key references back to users table.
  - **JSON representation**: the entire profile is self-contained — **better locality**, all relevant info in one place.

![[Pasted image 20260526071015.png]]

- JSON is supported natively by document databases (MongoDB, CouchDB, Espresso) and also by relational DBs (PostgreSQL, MySQL, SQL Server via JSON/XML columns) but often json cannot be queried like native column types can by in SQL.

### Many-to-One and Many-to-Many Relationships

When storing region or industry as an ID (rather than free text):
- **Consistent style and spelling** across records
- **Avoiding ambiguity** (e.g. multiple cities with the same name)
- **Ease of updating** — name stored in one place
- **Localization support** — can be translated
- **Better search** — linked to related concepts

> Storing IDs rather than strings is the **normalisation** principle: human-meaningful info lives in one place; everything else references it by ID.

![[Pasted image 20260526071038.png]]

- In **relational databases**, joins are easy — normal to refer to rows in other tables by ID.
- In **document databases**, joins are often weak or absent — fine for one-to-many tree structures since they are already 4represented in jsons, but problematic for many-to-many.
- If joins aren't supported, the application must emulate them — making multiple queries and joining in code. This is **slower** and **shifts complexity to the application**.

> Even if an application initially fits well in a join-free document model, **data tends to become more interconnected as features are added**.

Example of features that introduce many-to-many relationships in a résumé app:
- **Organisations and schools as entities** — rather than free text, so they can have their own pages with logo, description, etc.
- **Recommendations** — the recommendation needs a reference to the author's profile.

![[Pasted image 20260526071100.png]]

### Are Document Databases Repeating History?

The **hierarchical model** (IMS, 1970s) represented all data as a tree of records nested within records — very similar to JSON. Like document databases:
- Worked well for one-to-many relationships
- Made many-to-many relationships difficult
- Didn't support joins

Two solutions proposed at the time:

**Network model (CODASYL)**
- Every record could have multiple parents (not just one, like hierarchical)
- Links between records were **pointers** (not foreign keys) stored on disk
- The only way to access a record was to follow a **path from a root record** along chains of links — called an *access path*
- Querying required hand-coded traversal paths — very inflexible; adding a new query meant rewriting the access path logic

**Relational model**
- **Lays all data out in the open**: a table is simply a collection of rows — no nested structure, no access paths
- The **query optimizer automatically decides** which parts of the query to execute in which order, and which indexes to use — effectively the access path, but chosen automatically
- You only need to **build a query optimizer once**, and all applications benefit
- Adding a new feature = declare a new index; no need to rewrite queries

> Document databases reverted to the hierarchical model for storing nested records within their parent. But for many-to-one and many-to-many relationships, both relational and document databases use a **foreign key / document reference** — fundamentally not that different.

### Relational vs. Document Today — Which is Better?

**Arguments for the document model:**
- **Schema flexibility** (schema-on-read, not enforced by the DB)
- **Better locality** — data for a record is in one place, no multi-table joins needed
- **Closer to application data structures** — less impedance mismatch

**Arguments for the relational model:**
- Better support for **joins**
- Better support for **many-to-many relationships**
- Better **consistency guarantees**

**When to use document model:**
- Data has a **document-like structure** — a tree of one-to-many relationships, typically the entire tree is loaded at once
- The pieces of the document don't need to be cross-referenced from other parts of the app

**Limitations of document model:**
- Cannot refer directly to a **nested item within a document** — must use an access path (e.g. "second item in the list of positions for user 251")
- If data is **deeply nested**, this becomes cumbersome
- **Poor support for many-to-many relationships** — possible to denormalise, but then the app must do extra work to keep data consistent
- **Locality advantage only applies** if you need large parts of the document at the same time — loading an entire document to read one field is wasteful

**On updates:** the entire document usually needs to be **rewritten** on any update (unless the modification doesn't change the encoded size). Keep documents **fairly small** and avoid writes that increase document size.

### Schema-on-Read vs. Schema-on-Write

| | Schema-on-Write (relational) | Schema-on-Read (document) |
|---|---|---|
| Schema enforced | At write time, by the DB | At read time, by the application |
| Analogy | Static (compile-time) type checking | Dynamic (runtime) type checking |
| Changing format | Requires `ALTER TABLE` migration | Just start writing new documents; handle old format in app code |

> Schema-on-read is advantageous when: there are many different types of objects (not practical to put each in its own table), or the structure of data is determined by an external system and may change at any time.

**Schema migration note:** `ALTER TABLE` has a bad reputation for being slow, but this is **not entirely deserved** — most relational databases execute it in milliseconds. MySQL is the notable exception — it copies the entire table, which can take minutes or hours on large tables.

### Data Locality

- A document is stored as a **single continuous string** (JSON, XML, or binary variant like MongoDB's BSON).
- If the app needs to access the **entire document**, there's a **performance advantage** to locality vs. data split across multiple tables requiring multiple index lookups.
- The locality advantage **only applies if you need large parts of the document at once**.
- On updates, the entire document usually needs to be **rewritten** — only modifications that don't change the encoded size can be performed in place.

---

## Query Languages for Data

### Declarative vs. Imperative

| | Declarative (SQL, CSS) | Imperative (most APIs) |
|---|---|---|
| You specify | *What* you want | *How* to compute it |
| Implementation details | Hidden by the engine | Exposed to caller |
| Performance improvements | Engine can introduce transparently | Caller must change code |
| Parallel execution | Lends itself naturally | Harder — order of operations matters |

> Just as declarative **CSS styling** is better than manipulating styles imperatively in JavaScript, declarative **SQL** turned out to be much better than imperative query APIs.

### MapReduce
- **Neither fully declarative nor fully imperative** — a low-level programming model for distributed execution across many machines.
- Supported by MongoDB (and others) as an alternative query mechanism.
- `map` and `reduce` functions **must be pure** — they can only use the data passed to them, no side effects.
- The map function **emits key-value pairs**; pairs are **grouped by key**; the reduce function is called once per key.
- There is **nothing in SQL that constrains it to a single machine**, and MapReduce doesn't have a monopoly on distributed query execution — many distributed SQL implementations don't use MapReduce at all.
- Trend: MongoDB added an **aggregation pipeline** — a declarative alternative to MapReduce with a query optimizer.

---

## Graph-Like Data Models *(brief — niche)*

Useful when **many-to-many relationships are very common** throughout the data. A graph consists of **vertices** (nodes) and **edges** (relationships).

Graphs are not limited to homogeneous data — can store completely different types of objects in a single datastore (e.g. people, locations, events all as vertices, connected by typed edges).

### Property Graph model
- Each **vertex** has: unique ID, set of outgoing edges, set of incoming edges, collection of key-value properties
- Each **edge** has: unique ID, tail vertex, head vertex, a label describing the relationship, key-value properties
- Any vertex can have an edge connecting it to any other vertex — **no schema restricting** which things can be connected
- Can traverse graphs efficiently in both directions using indexes on `tail_vertex` and `head_vertex`

### Query Languages
- **Cypher** — declarative query language for property graphs, created for Neo4j. Expresses graph traversals naturally without specifying execution details.
- **SPARQL** — query language for triple-stores (RDF data model). All information stored as three-part statements: `(subject, predicate, object)`.
- **Datalog** — older language, foundation for many modern query languages. Rules can be applied recursively.
- **Graph queries in SQL** — possible but awkward; requires recursive CTEs (`WITH RECURSIVE`), which are verbose and clunky compared to Cypher.

---

## Summary

| Model | Best for | Watch out for |
|---|---|---|
| **Relational** | Many-to-many relationships, joins, strong consistency | Impedance mismatch with OO code; rigid schema |
| **Document** | Self-contained hierarchical data (one-to-many trees), flexible schema | Weak join support; locality only helps if reading whole document |
| **Graph** | Highly interconnected data with many types of relationships | Niche — overkill if your data isn't truly graph-shaped |
