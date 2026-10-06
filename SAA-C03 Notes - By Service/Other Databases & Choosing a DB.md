---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: "Other Databases & Choosing a DB"
version: C (by service)
source_chapters: [21 (01, 07-10)]
related: ["RDS & Aurora", "DynamoDB", "ElastiCache", "S3", "Redshift, EMR, OpenSearch & QuickSight", "Athena, Glue & Lake Formation", "Streaming Analytics - Flink & MSK"]
tags: [aws, saa-c03, documentdb, neptune, keyspaces, timestream, qldb, database-selection]
---

# Other Databases & Choosing a DB

Concept-only note. Lab steps: [[21 - Databases in AWS]] (Version B). Core databases have their own notes: [[RDS & Aurora]], [[DynamoDB]], [[ElastiCache]], [[S3]].

## 1. Choosing the right database
(src: 21/01-Choosing the right database)
Exam questions are architecture-driven. Dimensions to weigh: read-heavy / write-heavy / balanced; fluctuation during the day; data volume, growth, retention, average object size; access frequency; durability and source of truth; latency and concurrent users; data model and queries (joins, structured vs semi-structured, strong schema vs flexibility); reporting; search; relational vs NoSQL; license cost; cloud-native (Aurora).

| Category | AWS services |
|---|---|
| RDBMS (SQL, OLTP, joins) | RDS, Aurora |
| NoSQL | DynamoDB, ElastiCache, Neptune, DocumentDB, Keyspaces |
| Object store | S3 (big objects), Glacier (backups/archives) |
| Data warehouse (SQL analytics, BI, OLAP) | Redshift, Athena, EMR |
| Search | OpenSearch (free text, unstructured search) |
| Graph | Neptune |
| Ledger | QLDB (Quantum Ledger Database) |
| Time series | Timestream |

NoSQL: more flexible, usually no joins and no SQL (exceptions exist). Analytics services are covered in [[Redshift, EMR, OpenSearch & QuickSight]] and [[Athena, Glue & Lake Formation]].

[verify] QLDB is presented as a current ledger database.
> [!warning] Correction [note]
> AWS announced that QLDB support **ended on 31 July 2025**; AWS recommends Aurora PostgreSQL for audit/ledger use cases (without cryptographic verifiability). Source: web search results (InfoQ "AWS kill QLDB" and AWS Database Blog migration post: https://www.infoq.com/news/2024/07/aws-kill-qldb); AWS documentation page itself was not fetched.

## 2. DocumentDB
(src: 21/07-DocumentDB)
- "Aurora for **MongoDB**": **NoSQL document database**, MongoDB-compatible, stores/queries/indexes **JSON** data.
- Same deployment concept as Aurora: fully managed, HA, data replicated across **3 AZs**; storage grows automatically in **10 GB increments**; scales to millions of requests/second.

> [!tip] Exam
> MongoDB -> DocumentDB. "NoSQL" -> DocumentDB or DynamoDB.

## 3. Neptune
(src: 21/08-Neptune)
- Fully managed **graph database** for highly connected data (social networks: users, friends, posts, comments, likes).
- Replication across **3 AZs**, up to **15 read replicas**; billions of relationships, **millisecond** query latency; optimized for complex graph queries.
- Use cases: social networking, knowledge graphs (e.g. Wikipedia), fraud detection, recommendation engines.
- **Neptune Streams**: real-time, **ordered**, **no-duplicate** sequence of every change to the graph, read via an **HTTP REST API**. Uses: notifications, sync to S3 / OpenSearch / ElastiCache, cross-region replication into another Neptune cluster.

> [!tip] Exam
> Graph database -> Neptune.

## 4. Keyspaces (for Apache Cassandra)
(src: 21/09-Keyspaces (for Apache Cassandra))
- Managed **Apache Cassandra** (open-source distributed NoSQL): serverless, scalable, HA; scales tables up/down automatically; data replicated **3 times across AZs**.
- Query with **CQL**; single-digit millisecond latency at any scale; thousands of requests/second.
- Capacity modes like DynamoDB: **on-demand** or **provisioned with auto scaling**. Encryption, backups, **PITR up to 35 days**.
- Use cases: IoT device info, time-series data.

> [!tip] Exam
> Apache Cassandra -> Keyspaces.

## 5. Timestream
(src: 21/10-Timestream)
- Fully managed, fast, scalable, **serverless time-series database**; auto scales; trillions of events/day; much faster and cheaper than relational DBs for time series.
- Scheduled queries, multi-measure records, **full SQL**; **recent data in memory**, historical data in a cost-optimized tier; time-series analytics functions; encryption in transit and at rest.
- Ingest from: AWS IoT, Kinesis Data Streams (via Lambda or Flink), Prometheus, Telegraf, MSK (via Flink). Consume with: QuickSight, SageMaker, Grafana, any **JDBC** / SQL client.
- Use cases: IoT, operational apps, real-time analytics.

## Quick comparison

| Service | Type | Key phrase | Replication |
|---|---|---|---|
| DocumentDB | document (MongoDB) | MongoDB | 3 AZ, 10 GB storage steps |
| Neptune | graph | relationships, social, fraud | 3 AZ, up to 15 read replicas |
| Keyspaces | wide-column (Cassandra) | Cassandra, CQL | 3 copies across AZs |
| Timestream | time series | IoT, time-stamped events | managed, serverless |

## Not included here
- 21/02 RDS, 21/03 Aurora, 21/04 ElastiCache (see [[RDS & Aurora]], [[ElastiCache]]); 21/05 DynamoDB ([[DynamoDB]]); 21/06 S3 ([[S3]]).
