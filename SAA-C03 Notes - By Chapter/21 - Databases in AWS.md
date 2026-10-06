---
course: Ultimate AWS Certified Solutions Architect Associate 2026
chapter: 21
chapter_title: Databases in AWS
version: B (by chapter)
services: [RDS, Aurora, ElastiCache, DynamoDB, S3, DocumentDB, Neptune, Keyspaces, Timestream, QLDB]
tags: [aws, saa-c03, databases, rds, aurora, dynamodb, neptune, timestream]
---

# 21 - Databases in AWS

Related: [[Other Databases & Choosing a DB]] · [[RDS & Aurora]] · [[ElastiCache]] · [[DynamoDB]] · [[S3]] · [[Athena, Glue & Lake Formation]] · [[Redshift, EMR, OpenSearch & QuickSight]]

## Chapter summary
- The exam asks you to **choose the right database** from workload traits: read/write heavy, throughput, fluctuation, data size/growth, object size, latency, concurrency, data model, query pattern (joins), schema strictness, reporting, search, license cost.
- Mapping by type: RDBMS/OLTP = RDS, Aurora; NoSQL = DynamoDB, ElastiCache, Neptune, DocumentDB, Keyspaces; object store = S3, Glacier; warehouse/OLAP = Redshift, Athena, EMR; search = OpenSearch; graph = Neptune; ledger = QLDB; time series = Timestream.
- **Aurora**: PostgreSQL/MySQL compatible, 6 copies across 3 AZ (fixed), Global Database (up to 16 read instances per region, under 1 s replication), Serverless, cloning, ML integration.
- **ElastiCache** needs application code changes; no SQL. **DynamoDB** + **DAX** = microsecond reads; Global Tables = active-active.
- **DocumentDB = MongoDB**, **Neptune = graph**, **Keyspaces = Cassandra (CQL)**, **Timestream = time series**.
- Keyword to service: MongoDB -> DocumentDB; Cassandra -> Keyspaces; graph -> Neptune; time series -> Timestream.

---

## 01 - Choosing the right database
(src: 21/01-Choosing the right database)

- Questions to ask: write-heavy / read-heavy / balanced? Does load fluctuate? Data volume, growth, retention? Average object size, access frequency? Durability and source of truth? Latency and concurrent-user needs? Data model (structured, semi-structured), joins, strong schema vs flexibility? Reporting? Search? Relational vs NoSQL? License cost? Move to cloud-native (Aurora)?
- Categories:

| Category | Services |
|---|---|
| RDBMS (SQL / OLTP, joins) | RDS, Aurora |
| NoSQL (flexible, usually no joins / no SQL) | DynamoDB, ElastiCache, Neptune, DocumentDB, Keyspaces |
| Object store | S3 (big objects), Glacier (backups/archives) |
| Data warehouse (SQL analytics / BI, OLAP) | Redshift, Athena, EMR |
| Search (free text, unstructured) | OpenSearch |
| Graph (relationships) | Neptune |
| Ledger (transaction ledger) | QLDB (Quantum Ledger Database) |
| Time series | Timestream |

[verify] QLDB listed as an available ledger database.
> [!warning] Correction [note]
> AWS announced end of support for Amazon QLDB on **July 31, 2025**; after that date the service no longer operates (suggested alternative: Aurora PostgreSQL for audit use cases, without cryptographic verifiability). Source: [Amazon QLDB to Amazon Aurora PostgreSQL migration (AWS blog)](https://aws.amazon.com/jp/blogs/news/amazon-qldb-to-aurora-postgresql/) (found via search; the page itself was not opened, so the date comes from the search summary).

> [!tip] Exam
> Pick the database from the question's architecture clues; later sections and chapter 22 detail each one.

---

## 02 - RDS (summary)
(src: 21/02-RDS)

- Managed PostgreSQL, MySQL, Oracle, SQL Server, DB2, MariaDB, or **RDS Custom**.
- Provision instance size + EBS volume type/size; **storage auto scaling** available.
- **Read replicas** scale reads (run analytics on a replica, not production). **Multi-AZ** = standby for HA/DR only, cannot be queried.
- Security: username/password or **IAM authentication** (enforceable via **RDS Proxy**), security groups, **KMS** at rest, SSL/TLS in transit; **Secrets Manager** integration for credentials.
- Backups: **automated up to 35 days** with point-in-time restore (creates a new DB); **manual snapshots** for longer retention.
- Managed, scheduled maintenance (causes downtime).
- **RDS Custom** gives access to the underlying instance: Oracle and SQL Server.
- Use case: relational / OLTP with SQL and transactions.

---

## 03 - Aurora (summary)
(src: 21/03-Aurora)

- API-compatible with **PostgreSQL and MySQL**; **storage and compute are separate**.
- Storage: **6 replicas across 3 AZ by default, cannot be changed**; self-healing; automatic storage auto scaling.
- Compute: cluster of DB instances (can span AZ); read replicas auto scale. **Writer endpoint** and **reader endpoint**.
- Same security, monitoring, maintenance as RDS.
- Features: **Aurora Serverless** (unpredictable/intermittent workloads, no capacity planning); **Aurora Global** (up to **16 read instances per region**, cross-region storage replication usually **under 1 second**, promote a secondary region on primary failure); **Aurora Machine Learning** (SageMaker, Comprehend); **Cloning** (new cluster from existing, much faster than snapshot/restore, good for test/staging).
- Same use cases as RDS with less maintenance, more flexibility, performance and features.

> [!tip] Exam
> Aurora Global: under 1 s cross-region replication, up to 16 read instances per region. Cloning is faster than snapshot + restore.

---

## 04 - ElastiCache (summary)
(src: 21/04-ElastiCache)

- Managed **Redis or Memcached**; in-memory store, **sub-millisecond** read latency; provision an instance type.
- Redis: clustering (sharding), Multi-AZ, read replicas. Security: IAM, security groups, KMS at rest, **Redis AUTH**. Backups/snapshots/PITR like RDS; managed maintenance.
- **Using ElastiCache requires application code changes.**
- Use cases: key/value store, caching database reads, session data. **No SQL.**

> [!tip] Exam
> A caching solution that requires **no code change** is not ElastiCache.

---

## 05 - DynamoDB (summary)
(src: 21/05-DynamoDB)

- AWS proprietary, managed, serverless NoSQL with millisecond latency; highly available across multiple AZ; reads and writes decoupled; supports transactions.
- Capacity modes: **provisioned** (optional auto scaling; smooth, gradual workloads) or **on-demand** (unpredictable, steep spikes).
- Can replace ElastiCache as key/value store (e.g. session data) with **TTL** to expire rows.
- **DAX** = fully compatible read cache, **microsecond** read latency.
- Security/authorization via IAM.
- Events: **DynamoDB Streams** (invoke Lambda per change) or **Kinesis Data Streams** (adds Firehose integration, retention up to **1 year**).
- **Global Tables**: active-active multi-region, read/write anywhere.
- Backups: **PITR up to 35 days** (restore to new table) or on-demand backups (longer retention, restore to new table).
- **Export to S3** (within PITR window, 35 days) and **import from S3** use no RCU/WCU, import creates a new table.
- Use cases: serverless apps, small documents (hundreds of KB max), distributed serverless cache, rapidly evolving / flexible schema.

> [!tip] Exam
> "Rapidly evolving schema" or "flexible schema" -> DynamoDB. DAX = microsecond reads.

---

## 06 - S3 from a database perspective
(src: 21/06-S3)

- Key/value store for objects; good for **big** objects, not many small ones. Serverless, "infinite" scaling, **max object size 50 TB** (as stated by the instructor), versioning.
- Tiers: Standard, Infrequent Access, Intelligent, Glacier; move with **lifecycle policies**.
- Features: versioning, encryption, replication, MFA delete, access logs.
- Security: IAM, bucket policies, ACLs, access points, **S3 Object Lambda**, CORS, **Object Lock / Vault Lock** (Glacier).
- Encryption: SSE-S3, SSE-KMS, SSE-C, client-side, TLS in transit, default bucket encryption.
- **S3 Batch Operations**: act on all objects (e.g. encrypt unencrypted objects, copy before enabling replication); build the object list with **S3 Inventory**.
- Performance: **multi-part upload**, **Transfer Acceleration**, **S3 Select**.
- Automation: **Event Notifications** to SNS, SQS, Lambda, EventBridge.
- Use cases: static files, huge files, website hosting. Details live in the S3 chapters.

---

## 07 - DocumentDB
(src: 21/07-DocumentDB)

- Aurora-style cloud-native version for **MongoDB** (NoSQL, stores/queries/indexes JSON).
- Fully managed, highly available, data replicated across **3 AZ**, storage grows automatically in **10 GB increments**, scales to **millions of requests per second**.

> [!tip] Exam
> MongoDB -> DocumentDB. Generic NoSQL -> DocumentDB or DynamoDB.

---

## 08 - Neptune
(src: 21/08-Neptune)

- Fully managed **graph database** (social networks, knowledge graphs like Wikipedia, fraud detection, recommendation engines).
- Replication across **3 AZ**, up to **15 read replicas**; billions of relations, millisecond query latency; optimized for complex queries on highly connected data.
- **Neptune Streams**: real-time, ordered, no-duplicate sequence of every graph change, read through an **HTTP REST API**. Use for notifications, syncing to S3 / OpenSearch / ElastiCache, or cross-region replication into another Neptune cluster.

> [!tip] Exam
> Graph database -> Neptune.

---

## 09 - Keyspaces (for Apache Cassandra)
(src: 21/09-Keyspaces (for Apache Cassandra))

- Managed **Apache Cassandra** (open-source NoSQL); serverless, scalable, highly available; auto scales tables with traffic.
- Data replicated **3 times** across multiple AZ; queries use **CQL**; single-digit millisecond latency at any scale, thousands of requests per second.
- Capacity modes like DynamoDB: **on-demand** or **provisioned with auto scaling**. Encryption, backups, **PITR up to 35 days**.
- Use cases: IoT device data, time-series data.

> [!tip] Exam
> Apache Cassandra -> Keyspaces.

---

## 10 - Timestream
(src: 21/10-Timestream)

- Fully managed, fast, scalable, **serverless time series database**; scales up/down automatically; stores/analyzes **trillions of events per day**; faster and cheaper than relational for time series.
- Scheduled queries, multi-measure records, **full SQL compatibility**, time series analytics functions. **Recent data in memory**, historical data in cost-optimized tier. Encryption in transit and at rest.
- Use cases: IoT, operational applications, real-time analytics.
- Ingest from: AWS IoT, Kinesis Data Streams (via Lambda or Managed Service for Apache Flink), MSK (via Flink), Prometheus, Telegraf. Consume with: QuickSight, SageMaker, Grafana, any **JDBC** + SQL application.

---

Not covered: none; all 10 lectures have transcripts.
