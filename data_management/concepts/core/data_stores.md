# Data Stores — OLTP, OLAP, NoSQL & Object Storage

**Scope**: Types of data stores (OLTP, OLAP, NoSQL, object storage), when to use which, and how they fit into data platforms. For deep RDBMS design and operations, see [Database](database.md).

**Purpose**: Use this when choosing storage for an application or workload, when explaining OLTP vs OLAP, or when integrating relational, NoSQL, and object storage in a data platform.

---

## 1. OLTP vs OLAP

### OLTP (Online Transaction Processing)

- **Purpose**: Day-to-day transactional workloads; high-volume, low-latency reads and writes.
- **Characteristics**: ACID transactions, row-oriented or row-friendly, normalized or lightly denormalized, indexed for point lookups and small range scans.
- **Typical stores**: RDBMS (SQL Server, PostgreSQL, MySQL), transactional NoSQL (e.g. document DB with transactions).
- **Use when**: Applications need consistent, real-time reads/writes (orders, inventory, user sessions, configuration).

### OLAP (Online Analytical Processing)

- **Purpose**: Analytics, reporting, aggregations over large datasets; read-heavy, batch or interactive.
- **Characteristics**: Column-oriented or column-friendly, denormalized/star schema, optimized for full scans and aggregations; often eventual consistency or batch-refreshed.
- **Typical stores**: Data warehouses (Synapse, Snowflake, BigQuery, Redshift), columnar stores, data lakehouses (Delta, Iceberg) with SQL layers.
- **Use when**: BI, reporting, dashboards, ad-hoc analytics, ML feature backfills.

### Hybrid and Patterns

- **HTAP (Hybrid Transactional/Analytical)**: Single store or tier that supports both; often via separate engines (e.g. row store + column store) or careful schema.
- **Operational reporting**: Read replicas or CDC from OLTP into OLAP so reporting doesn’t load the transaction system.
- **Medallion / lakehouse**: Raw (object) → cleaned (Silver) → analytical (Gold); OLTP systems feed ingestion; OLAP and BI read from Gold.

---

## 2. NoSQL Store Types

### Document Stores

- **Model**: Documents (JSON, BSON) with optional schema; nested structures; often indexed by document id and selected fields.
- **Examples**: MongoDB, Cosmos DB (API for MongoDB), DynamoDB (document-style).
- **Use when**: Flexible or evolving schema, document-centric APIs, catalogs, content, event payloads.

### Key-Value Stores

- **Model**: Key → value; minimal structure; very fast lookups by key.
- **Examples**: Redis, DynamoDB, Riak, Azure Table Storage.
- **Use when**: Caching, sessions, feature flags, simple lookups by id, queues.

### Wide-Column (Column Family) Stores

- **Model**: Rows keyed by row key; columns grouped into families; sparse; good for high write throughput and range scans by row key.
- **Examples**: Cassandra, HBase, Cosmos DB (Cassandra API).
- **Use when**: Time-series, event logs, high write volume, access by partition/row key.

### Graph Stores

- **Model**: Nodes and edges; traversals and pattern matching.
- **Examples**: Neo4j, Cosmos DB (Gremlin), Neptune.
- **Use when**: Relationships and traversal are central (social, recommendations, fraud, knowledge graphs).

### When to Choose NoSQL vs RDBMS

- **RDBMS**: Strong consistency, complex queries, joins, and integrity constraints matter. See [Database](database.md).
- **NoSQL**: Scale-out writes, flexible or sparse schema, single-key or partition-key access, or graph/traversal workloads. Use the type (document, key-value, wide-column, graph) that matches access patterns.

---

## 3. Object Storage (Blob / S3-Style)

- **Model**: Flat namespace (bucket/container + key); immutable blobs; no indexing; cheap, durable, good for large files and streams.
- **Examples**: Azure Blob / Data Lake Storage, S3, GCS; used as the base layer for data lakes and lakehouses.
- **Use when**: Data lakes, raw ingest, archives, ML artifacts, parquet/Delta/ORC files for analytical workloads. Often queried via SQL engines (Spark, Synapse, Databricks SQL) over Delta/Iceberg.

---

## 4. Coverage at a Glance

| Workload / need        | Typical store type     | See also        |
|------------------------|------------------------|-----------------|
| Transactions, CRUD     | OLTP RDBMS or doc/kv   | [Database](database.md) |
| Reporting, BI, analytics | OLAP, warehouse, lakehouse | [Analytics Engineering](analytics_eng.md), [Fabric](../platform/fabric.md), [Databricks](../platform/databricks.md) |
| Flexible schema, docs  | Document store         | —               |
| Cache, sessions        | Key-value              | —               |
| Events, time-series    | Wide-column or append  | [Streaming](../platform/streaming.md) |
| Relationships, graph   | Graph store            | —               |
| Raw data, lake, ML     | Object storage + engine| [Data Engineering](engineering.md), [concepts/platform/](../platform/) |

---

> **Note**: For RDBMS design, indexing, transactions, and operations, see [Database](database.md). For pipeline and lake patterns, see [Data Engineering](engineering.md). For governance and quality across stores, see [Data Governance](governance.md) and [Data Operations](operations.md). For Fabric, Databricks, Azure: [concepts/platform/](../platform/).
