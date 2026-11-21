---
description: AWS exam notes on database services, covering RDS, Aurora, RDS Proxy, Redshift, DynamoDB, DAX, ElastiCache, MemoryDB and the purpose-built databases.
tags:
  - AWS
  - Databases
  - Exam Prep
---

# AWS Database Services

Quick revision notes for the Solutions Architect exam. The table below gives one line per service. The sections after it cover **what the service is**, **when to pick it**, and the **key facts** that tend to show up in questions.

## At a glance

| Service | Type | One-liner |
|---|---|---|
| **RDS** | Relational (OLTP) | Managed MySQL, PostgreSQL, MariaDB, Oracle, SQL Server and Db2. AWS handles patching, backups and failover |
| **Aurora** | Relational (OLTP) | AWS-built MySQL/PostgreSQL-compatible engine with faster, self-healing storage spread across 3 AZs |
| **RDS Proxy** | Connection pooler | Pools and shares DB connections so Lambda and bursty apps don't overwhelm the database |
| **Redshift** | Data warehouse (OLAP) | Columnar, massively parallel warehouse for analytics and reporting on petabytes of data |
| **Redshift Serverless** | Data warehouse (OLAP) | Redshift without clusters to manage. Capacity scales automatically and you pay only while queries run |
| **DynamoDB** | Key-value / document (NoSQL) | Serverless NoSQL with single-digit millisecond latency at any scale |
| **DynamoDB Accelerator (DAX)** | Cache for DynamoDB | In-memory cache in front of DynamoDB. Microsecond reads with almost no code change |
| **OpenSearch Service** | Search / log analytics | Managed OpenSearch (Elasticsearch fork) for full-text search, log analytics and dashboards |
| **ElastiCache** | In-memory cache | Managed Redis OSS, Valkey or Memcached for sub-millisecond caching and session storage |
| **MemoryDB** | In-memory database | Durable, Redis/Valkey-compatible database you can use as a *primary* store, not just a cache |
| **DocumentDB** | Document | MongoDB-compatible JSON document database |
| **Keyspaces** | Wide-column | Serverless Apache Cassandra-compatible database (CQL) |
| **Neptune** | Graph | Graph database for highly connected data such as social networks, fraud rings and recommendations |
| **QLDB** | Ledger | Immutable, cryptographically verifiable journal of changes. **End of support July 2025** |
| **Timestream** | Time series | Serverless time series database for IoT, metrics and telemetry |

### Picking a database

| Need | Service |
|---|---|
| Relational / SQL, OLTP, joins, transactions | RDS, Aurora |
| Key-value / NoSQL, single-digit ms at any scale, serverless | **DynamoDB** (DAX for microsecond caching) |
| In-memory cache, session store | **ElastiCache** |
| Redis API as a durable primary database | **MemoryDB** |
| Data warehouse, analytics (OLAP), BI | **Redshift** |
| Full-text search, log analytics | **OpenSearch** |
| Document (MongoDB compatible) | **DocumentDB** |
| Graph (relationships, social networks, fraud) | **Neptune** |
| Wide-column (Apache Cassandra compatible) | **Keyspaces** |
| Time series (IoT, metrics) | **Timestream** |
| Immutable, verifiable record of changes | QLDB (legacy). For new builds use Aurora PostgreSQL with audit tables |

---

## Relational databases

### Amazon RDS (Relational Database Service)

Use RDS for **relational (SQL) database** requirements when you don't want to manage the database server yourself.

#### Why it exists

Running a database on EC2 means *you* handle OS and DB patching, backups, upgrades, replication and failover. RDS takes over that routine administration so you only manage the schema, queries and settings.

#### What you get

| Feature | Details |
|---|---|
| **Routine operations** | Automated patching, minor version upgrades, maintenance windows |
| **High availability** | Multi-AZ deployments with automatic failover |
| **Scalability** | Scale instance size (vertical), add read replicas (horizontal reads), storage auto-scaling |
| **Backup & restore** | Automated backups (retention 1–35 days) with **point-in-time restore**; manual snapshots kept until you delete them |
| **Monitoring** | CloudWatch metrics, Enhanced Monitoring (OS-level), Performance Insights |
| **Security** | Runs in your VPC, security groups, encryption at rest (KMS), TLS in transit, IAM database authentication |

#### Supported engines

MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server, IBM Db2, and **Amazon Aurora** (MySQL and PostgreSQL compatible).

#### Instance classes

| Class | Family | Use for |
|---|---|---|
| **General purpose** | `db.m*` | Balanced CPU and memory; most workloads |
| **Memory optimized** | `db.r*`, `db.x*` | Memory-heavy workloads, large caches and datasets |
| **Burstable** | `db.t*` | Dev/test, small or spiky workloads (CPU credits) |

Storage options: **General Purpose SSD (gp2/gp3)** and **Provisioned IOPS SSD (io1/io2)** for I/O-intensive workloads.

#### Deployment types

| Deployment | How it works | Standby readable? | Purpose |
|---|---|---|---|
| **Single-AZ** | One DB instance in one AZ | — | Dev/test, lowest cost, no automatic failover |
| **Multi-AZ DB instance** | Primary + **one standby** in another AZ, **synchronous** replication | ❌ No | High availability / disaster recovery; automatic failover (~1–2 min), same DNS endpoint |
| **Multi-AZ DB cluster** | One writer + **two readable standbys** across 3 AZs | ✅ Yes | HA **and** read scaling; faster failover (typically < 35 s); MySQL & PostgreSQL only |

#### Read replicas (separate from Multi-AZ)

- **Asynchronous** replication → used for **read scaling**, not HA.
- Up to **15** read replicas (MySQL, MariaDB, PostgreSQL; Aurora also 15).
- Can be in the same AZ, a different AZ, or **another Region** (cross-Region DR / lower latency for global readers).
- Each replica has its **own endpoint**; the application must send reads to it.
- A replica can be **promoted** to a standalone database.

!!! tip "Exam tip: Multi-AZ vs Read Replica"
    - **Multi-AZ** = availability (synchronous, failover). The classic standby does *not* serve reads.
    - **Read replica** = performance (asynchronous, serves reads).
    - "Improve read performance" → read replicas. "Survive an AZ failure" → Multi-AZ.

#### Other RDS points worth knowing

- **Encryption** must be enabled **at creation**. To encrypt an existing DB: snapshot → copy the snapshot with encryption → restore.
- Read replicas of an encrypted DB are encrypted with the same key (cross-Region replicas use a key in the target Region).
- **RDS Custom** (Oracle, SQL Server) gives you OS and DB access for apps that need custom settings.
- **Stopping** an RDS instance only lasts 7 days, then AWS starts it again. For long pauses, snapshot and delete.

### Amazon Aurora

AWS's own cloud-native relational engine, compatible with **MySQL** and **PostgreSQL**. It's usually the "best performance / highest availability relational DB" answer.

- **Performance**: up to ~5x MySQL and ~3x PostgreSQL throughput on the same hardware.
- **Storage**: keeps **6 copies across 3 AZs**, self-heals, and grows automatically in 10 GB steps up to **128 TiB**. It survives losing 2 copies for writes and 3 for reads.
- **Replicas**: up to **15** low-latency Aurora Replicas that share the same storage volume. Failover to a replica typically takes **under 30 s**.
- **Endpoints**:
    - **Cluster (writer) endpoint**: always points at the current primary.
    - **Reader endpoint**: load-balances connections across the replicas.
    - **Custom endpoints**: a subset of replicas, e.g. bigger instances for analytics queries.
- **Aurora Serverless v2**: capacity scales in fine steps (ACUs) with demand, and can pause down to 0 ACUs when idle. Good for variable, unpredictable or dev/test workloads.
- **Aurora Global Database**: one primary Region plus read-only secondary Regions, storage-level replication with typically **< 1 s lag**. Promote a secondary in about a minute for cross-Region DR (low RPO/RTO).
- **Backtrack** (MySQL): rewind the DB to an earlier time without restoring a backup.
- **Cloning**: fast copy-on-write clone of a cluster, handy for creating test copies of production.
- **Aurora DSQL**: newer serverless, distributed SQL (PostgreSQL-compatible) with active-active multi-Region writes.

!!! tip "Exam tip"
    "Relational, highest availability, read scaling with minimal lag" → **Aurora**. "Cross-Region DR with RPO of seconds" → **Aurora Global Database**. "Unpredictable / intermittent relational workload" → **Aurora Serverless v2**.

### Amazon RDS Proxy

Fully managed **database connection pooler** for RDS and Aurora.

- Apps connect to the proxy, which keeps a **warm pool of connections** to the DB and shares them. This stops "too many connections" errors and cuts CPU/memory spent opening connections.
- Classic use case: **AWS Lambda**. Thousands of short-lived function invocations would otherwise each open their own connection.
- **Faster failover**: cuts failover time for Aurora and RDS by up to ~66%, because the proxy holds client connections while the DB fails over.
- **Security**: can enforce **IAM authentication**, with DB credentials stored in **Secrets Manager**.
- Serverless, auto-scaling and Multi-AZ. It is **only reachable from inside the VPC**, never public.
- No application code change beyond pointing at the proxy endpoint.

---

## Data warehousing

### Amazon Redshift

Fully managed, petabyte-scale **data warehouse** for **OLAP** (analytics, reporting, BI). It is **not** for OLTP (lots of small transactional reads/writes).

- **Columnar storage + massively parallel processing (MPP)**. Queries run across many nodes at once, which makes big aggregations fast.
- **Architecture**: a **leader node** plans queries and gathers results; **compute nodes** store data and run the query in parallel. **RA3** nodes separate compute from **managed storage** (backed by S3), so you can scale each independently.
- **Loading data**: `COPY` from S3 (fastest for bulk), Kinesis Data Firehose, AWS Glue, DMS, or **zero-ETL integrations** from Aurora, RDS and DynamoDB.
- **Redshift Spectrum**: query data **directly in S3** from Redshift without loading it, and join it with warehouse tables.
- **Concurrency scaling**: adds temporary capacity automatically when many users query at once.
- **HA & DR**: automated and manual snapshots stored in S3; snapshots can be **copied automatically to another Region**. RA3 clusters support **Multi-AZ**.
- **Security**: VPC, KMS encryption, **Enhanced VPC Routing** (forces `COPY`/`UNLOAD` traffic through your VPC).
- Integrates with **QuickSight** and other BI tools via JDBC/ODBC.

!!! tip "Exam tip"
    "Analytics / BI / complex queries over large historical data" → **Redshift**. "Query S3 data occasionally, no infrastructure" → **Athena**. "Join S3 data with warehouse data" → **Redshift Spectrum**.

### Amazon Redshift Serverless

- Run Redshift **without provisioning or managing clusters**. Capacity is measured in **Redshift Processing Units (RPUs)** and scales up and down automatically.
- You **pay only while queries run** (per second), so it suits **intermittent, unpredictable or new** analytics workloads.
- Same SQL, same features (Spectrum, data sharing, zero-ETL) as provisioned Redshift.
- Choose provisioned Redshift for steady, predictable, always-on workloads where reserved nodes are cheaper.

---

## NoSQL & in-memory

### Amazon DynamoDB

Fully managed, **serverless key-value and document** database with **single-digit millisecond** performance at any scale. One of the most tested services on the exam.

**Data model**

- **Tables** of **items** (rows); each item is up to **400 KB**.
- **Primary key**: a **partition key** alone, or a **partition key + sort key** (composite).
- Schema-less: items in the same table can have different attributes.

**Capacity modes**

| Mode | How it works | Use |
|---|---|---|
| **On-demand** | Pay per request; scales instantly | Unpredictable or new workloads, spiky traffic |
| **Provisioned** | You set **RCUs / WCUs**, optionally with auto scaling; cheaper with steady traffic | Predictable workloads, cost optimisation |

**Reads**: **eventually consistent** by default; **strongly consistent** reads available (double the RCU cost). **Transactions** (ACID across multiple items/tables) are supported.

**Indexes**

- **Global Secondary Index (GSI)**: different partition and sort key; can be **added any time**; has its own capacity.
- **Local Secondary Index (LSI)**: same partition key, different sort key; must be created **with the table**.

**Features worth knowing**

- **DynamoDB Streams**: ordered log of item changes (kept 24 h). Trigger **Lambda** for event-driven processing, replication or audit.
- **Global Tables**: **multi-Region, multi-active** replication. Read and write in any Region with low latency.
- **TTL**: automatically deletes expired items (sessions, temporary data) at no cost.
- **Backups**: on-demand backups and **point-in-time recovery** (up to 35 days); **export to S3** without using read capacity.
- **Table classes**: Standard, and **Standard-IA** for rarely accessed data.
- Security: IAM policies (down to item/attribute level), **VPC gateway endpoint** for private access, encryption at rest by default.

!!! tip "Exam tip"
    "Serverless", "NoSQL", "key-value", "session state", "massive scale with ms latency", "schema changes often" → **DynamoDB**. "Multi-Region active-active" → **Global Tables**. "React to every item change" → **Streams + Lambda**.

### DynamoDB Accelerator (DAX)

- Fully managed, **in-memory cache built for DynamoDB**. Cuts read latency from milliseconds to **microseconds**.
- **API-compatible** with DynamoDB, so apps need only a client change, not a rewrite.
- Caches both individual items (item cache) and query/scan results (query cache). Default TTL **5 minutes**.
- Runs as a cluster inside your VPC, Multi-AZ recommended.
- Helps with **hot partitions** and read-heavy workloads; it does **not** help write-heavy workloads.

!!! note "DAX vs ElastiCache"
    **DAX** caches DynamoDB items and queries with no app logic. **ElastiCache** is general-purpose. Use it to cache computed or aggregated results, or data from any source.

### Amazon ElastiCache

Fully managed **in-memory data store** for **sub-millisecond** latency. Engines: **Redis OSS**, **Valkey** (open-source Redis fork) and **Memcached**. There is also a **Serverless** option.

- **Use cases**: cache database query results to offload RDS/Aurora, **session store** for stateless web tiers, **leaderboards** (Redis sorted sets), rate limiting, pub/sub.
- Using it **requires application code changes**. Common caching strategies:
    - **Lazy loading (cache-aside)**: read the cache, on a miss read the DB and write to the cache. Data can be stale.
    - **Write-through**: write to the cache whenever you write to the DB. Data is always fresh but some is never read.
    - Use **TTL** to expire stale entries.

| | **Redis OSS / Valkey** | **Memcached** |
|---|---|---|
| Replication & Multi-AZ failover | ✅ Yes | ❌ No |
| Persistence / backups | ✅ Yes | ❌ No |
| Data types | Rich (lists, sets, sorted sets, hashes) | Simple key-value |
| Threads | Mostly single-threaded per shard | **Multi-threaded** |
| Pick when | You need HA, durability, advanced data types | Simple cache, scale out horizontally, data loss is fine |

- Security: VPC, security groups, encryption in transit and at rest, **Redis AUTH** or IAM authentication.

### Amazon MemoryDB

- **Durable, Redis OSS / Valkey-compatible in-memory database**. Built to be your **primary database**, not just a cache.
- Data is stored in a **Multi-AZ transaction log**, so it survives node failures without data loss.
- **Microsecond reads**, single-digit millisecond writes.
- Use when you want the **Redis API and speed with full durability**, e.g. microservices, gaming state, real-time data that can't be lost.

!!! tip "Exam tip"
    "Cache to reduce DB load" → **ElastiCache**. "Redis-compatible **primary** database, durable" → **MemoryDB**.

### Amazon OpenSearch Service

- Managed **OpenSearch** (and legacy Elasticsearch) clusters, plus **OpenSearch Dashboards** for visualisation. Available **provisioned** or **Serverless**.
- Use for **full-text search** (partial matches, relevance ranking), **log and clickstream analytics**, and observability.
- Usually sits **beside** a primary database rather than replacing it. Common patterns:
    - **DynamoDB → Streams → Lambda → OpenSearch** to make DynamoDB data searchable.
    - **CloudWatch Logs / Kinesis Data Firehose → OpenSearch** for log analytics.
- Security: VPC access, fine-grained access control, Cognito/IAM/SAML authentication, encryption at rest and in transit.

!!! tip "Exam tip"
    "Search any field / partial text match" or "analyse logs in near real time" → **OpenSearch**.

---

## Purpose-built databases

### Amazon DocumentDB (with MongoDB compatibility)

- Managed **JSON document database** compatible with MongoDB APIs and drivers.
- Built on Aurora-like storage: **6 copies across 3 AZs**, grows automatically up to 128 TiB, up to **15 read replicas**.
- **Elastic clusters** shard data to handle millions of reads/writes per second.
- Use for content management, catalogs, user profiles, or to **migrate MongoDB workloads** to a managed service.

### Amazon Keyspaces (for Apache Cassandra)

- **Serverless, Cassandra-compatible wide-column** database. Use the same **CQL** (Cassandra Query Language) code and drivers.
- On-demand or provisioned capacity; data replicated **3 times across AZs**; point-in-time recovery (35 days).
- Use for high-write workloads such as IoT device data, fleet tracking, or **moving an existing Cassandra cluster** to AWS without managing nodes.

### Amazon Neptune

- Fully managed **graph database**. Stores **nodes and relationships (edges)** and queries them fast, even across billions of relationships.
- Query languages: **Gremlin**, **openCypher**, **SPARQL** (RDF).
- HA like Aurora: 6 copies across 3 AZs, up to 15 read replicas.
- Use for **social networks**, **fraud detection** (spot rings of connected accounts), **recommendation engines**, knowledge graphs.

!!! tip "Exam tip"
    The word "relationships", "connected data" or "graph" in a question → **Neptune**.

### Amazon QLDB (Quantum Ledger Database)

- Fully managed **ledger database** with an **immutable, append-only journal**. Each change is chained with **SHA-256** hashes, so you can **cryptographically verify** that history was never altered.
- **Centralised** (one trusted owner), unlike Amazon Managed Blockchain, which is **decentralised**.
- Use cases: financial transaction history, supply-chain tracking, registration and claims records.

!!! warning "Availability"
    AWS **ended support for QLDB on 31 July 2025**. It may still appear in older exam questions. For new builds AWS recommends **Aurora PostgreSQL** with audit/history tables.

### Amazon Timestream

- **Serverless time series database** for IoT sensor data, application metrics and DevOps telemetry.
- Keeps recent data in a fast **memory store** and moves older data automatically to a cheaper **magnetic store**.
- SQL with built-in **time series functions** (interpolation, smoothing, aggregation over time windows).
- **Timestream for InfluxDB** runs managed open-source InfluxDB for teams that already use it.

!!! note "Availability"
    Timestream for LiveAnalytics (the original serverless engine) is closed to new customers. **Timestream for InfluxDB** is the option for new builds, but "time series" in an exam question still means **Timestream**.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Managed relational DB, minimal admin | RDS / Aurora |
| Survive AZ failure for a DB | RDS Multi-AZ |
| Scale database reads | Read replicas (or Multi-AZ DB cluster) |
| Relational DB with best performance and fastest failover | Aurora |
| Cross-Region relational DR with < 1 s lag | Aurora Global Database |
| Variable relational workload, scale to zero | Aurora Serverless v2 |
| Lambda exhausting DB connections | RDS Proxy |
| Encrypt an existing unencrypted RDS DB | Snapshot → encrypted copy → restore |
| Data warehouse / BI analytics | Redshift |
| Intermittent analytics, no cluster management | Redshift Serverless |
| Query S3 data from the warehouse | Redshift Spectrum |
| Serverless NoSQL, ms latency at any scale | DynamoDB |
| Microsecond reads for DynamoDB | DAX |
| Multi-Region active-active NoSQL | DynamoDB Global Tables |
| Run code on every DynamoDB change | DynamoDB Streams + Lambda |
| Cache query results / store sessions | ElastiCache |
| Durable Redis-compatible primary DB | MemoryDB |
| Full-text search, log analytics | OpenSearch |
| MongoDB workloads | DocumentDB |
| Cassandra workloads | Keyspaces |
| Highly connected data, relationships | Neptune |
| Immutable, verifiable history | QLDB (legacy) |
| IoT / time series data | Timestream |
