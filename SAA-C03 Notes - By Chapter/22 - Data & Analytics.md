---
course: Ultimate AWS Certified Solutions Architect Associate 2026
chapter: 22
chapter_title: Data & Analytics
version: B (by chapter)
services: [Athena, Redshift, OpenSearch, EMR, QuickSight, Glue, Lake Formation, Managed Service for Apache Flink, MSK, Kinesis, IoT Core]
tags: [aws, saa-c03, analytics, athena, redshift, opensearch, emr, quicksight, glue, lake-formation, flink, msk]
---

# 22 - Data & Analytics

Related: [[Athena, Glue & Lake Formation]] · [[Redshift, EMR, OpenSearch & QuickSight]] · [[Streaming Analytics - Flink & MSK]] · [[Kinesis & Firehose]] · [[S3]] · [[DynamoDB]] · [[Lambda]]

## Chapter summary
- **Athena**: serverless SQL (Presto) on S3, pay per TB scanned; use **Parquet/ORC**, compression, partitioning, files of 128 MB+; **Federated Query** via Lambda data source connectors.
- **Redshift**: PostgreSQL-based OLAP warehouse, columnar, leader + compute nodes, provisioned or serverless; **Spectrum** queries S3 without loading; snapshots (incremental) with cross-region copy.
- **OpenSearch**: search any field incl. partial match, complement to another DB; fed by DynamoDB Streams + Lambda, CloudWatch Logs, Kinesis.
- **EMR**: Hadoop/Spark/HBase/Presto/Flink clusters on EC2; master + core (long running), task (spot).
- **QuickSight**: serverless BI dashboards; **SPICE** only for imported data; column-level security in Enterprise.
- **Glue**: serverless ETL (e.g. CSV to Parquet) plus **Data Catalog** used by Athena, Redshift Spectrum, EMR; **Lake Formation** = data lake with centralized row/column-level permissions on top of Glue.
- **Managed Service for Apache Flink** processes streams (Kinesis Data Streams, MSK; NOT Firehose); **MSK** = managed Kafka, alternative to Kinesis.
- Reference ingestion pipeline: IoT Core -> Kinesis Data Streams -> Firehose (1 min) -> S3 -> Lambda -> Athena -> S3 reports -> QuickSight / Redshift.

---

## 01 - Athena
(src: 22/01-Athena)

- Serverless query service for data in **S3** using standard SQL; built on **Presto**; data is not moved. Formats: CSV, JSON, ORC, Avro, Parquet (and others).
- Pricing: fixed amount **per TB of data scanned**; no database to provision.
- Commonly paired with **QuickSight** for reports/dashboards. Use cases: ad hoc queries, BI, analytics, reporting, analyzing AWS logs (VPC Flow Logs, load balancer logs, CloudTrail).
- **Performance tuning** (tested at the exam):
  - **Columnar formats** (**Parquet, ORC**) scan only needed columns; convert with **Glue** ETL (e.g. CSV to Parquet).
  - **Compress** data for smaller retrievals.
  - **Partition** data using S3 paths such as `/year=1991/month=1/day=1/` so queries on those columns read only matching folders.
  - Use **bigger files (128 MB+)**; many small files hurt performance.
- **Federated Query**: query non-S3 sources (relational/non-relational, custom, AWS or on-premises) through a **Data Source Connector = Lambda function** (one per connector). Examples: CloudWatch Logs, DynamoDB, RDS, ElastiCache, DocumentDB, Redshift, Aurora, SQL Server, MySQL, HBase on EMR, on-premises DBs; joins across sources possible; results can be stored in S3.

> [!tip] Exam
> Serverless SQL on S3 -> Athena. Cheaper/faster: Parquet/ORC + partitioning + compression + large files.

---

## 02 - Athena Hands On
(src: 22/02-Athena Hands On)

- Goal: query S3 server access logs with SQL, no server needed.
- Setting a **query result location** (an S3 bucket) is required before the first query.
- Created a database from SQL, then a table over the access-log bucket (query taken from the S3/Athena documentation; only the LOCATION changes: bucket name, optional prefix, **trailing slash required**).
- Preview table returns 10 rows (bucket owner, bucket, request time, IP, requester, request ID, ...).
- Example analysis: count requests by HTTP status and operation (scanned about 30 MB); inspect 404 (not found, 142 rows) and 403 (unauthorized) to spot problems or unauthorized access attempts.

### Hands-on steps
1. Open Athena -> launch query editor -> Settings -> Manage -> set query result location to an S3 bucket (create a new bucket in S3 first, e.g. in eu-central-1; name must be unique).
2. `CREATE DATABASE` (e.g. s3_access_logs_db) and select it in the left panel.
3. Run the documented `CREATE TABLE` for S3 access logs, replacing LOCATION with `s3://<log-bucket>/<prefix>/` (add trailing slash; no prefix if objects are at top level). `[screen action]` remove the first line of the pasted query as needed.
4. Table menu (three dots) -> Preview table.
5. New query sheet: aggregate with COUNT grouped by HTTP status and request operation/URI; filter 404 and 403 rows.

---

## 03 - Redshift
(src: 22/03-Redshift)

- Based on **PostgreSQL** but **OLAP** (analytics, data warehousing), not OLTP. **10x better performance** than other warehouses (instructor claim), scales to **petabytes**, **columnar storage**, parallel query engine, SQL interface, BI tools (QuickSight, Tableau).
- Modes: **provisioned cluster** (choose instance types; reserved instances for savings) or **serverless**.
- vs Athena: Redshift has faster queries/joins/aggregations (indexes) but needs a cluster; Athena is serverless with data in S3.
- Architecture: **leader node** (query planning, result aggregation) + **compute nodes** (run queries, return results to leader).
- **DR/snapshots**: most clusters are **single AZ** (Multi-AZ exists for some cluster types, then good for DR); otherwise use snapshots. Snapshots are point-in-time, stored in **S3**, **incremental**, restored into a **new cluster**. Automated: **every 8 hours or every 5 GB or on a schedule**, with configurable retention; manual: kept until deleted. Can **auto-copy snapshots to another region** for DR.
- **Ingestion**:
  1. **Kinesis Data Firehose**: writes to S3, then issues an S3 **COPY** command into Redshift.
  2. Manual **COPY** from S3 using an IAM role; goes over the internet unless **Enhanced VPC Routing** is enabled (then traffic stays in the VPC).
  3. **JDBC driver** from applications (e.g. EC2); write in **large batches**, not row by row.
- **Redshift Spectrum**: analyze data in S3 without loading it, using far more processing power. Requires an existing Redshift cluster; the query is submitted to **thousands of Spectrum nodes** that read S3, aggregate and return results to the cluster.

> [!tip] Exam
> Analytics/OLAP/warehouse -> Redshift. Query S3 with Redshift without loading -> Spectrum. DR for single-AZ -> snapshots with cross-region copy.

---

## 04 - OpenSearch (ex-Elasticsearch)
(src: 22/04-OpenSearch (ex- ElasticSearch))

- Successor to **Amazon Elasticsearch** (renamed due to licensing issues).
- Unlike DynamoDB (query by primary key/index), OpenSearch searches **any field, including partial matches**; used as a **complement to another database**; also supports analytics queries.
- Modes: **managed cluster** (instances you see) or **serverless**.
- Own query language; **SQL only via plugin**. Security: Cognito, IAM, encryption at rest and in flight. **OpenSearch Dashboards** for visualization.
- Ingest sources: Kinesis Data Firehose, IoT, CloudWatch Logs, custom apps.
- Patterns:
  - **DynamoDB -> DynamoDB Stream -> Lambda -> OpenSearch**; app searches partial name in OpenSearch, gets the item ID, then fetches the full item from DynamoDB.
  - **CloudWatch Logs -> Subscription Filter -> AWS-managed Lambda -> OpenSearch** (real time), or **Subscription Filter -> Firehose -> OpenSearch** (near real time).
  - **Kinesis Data Streams -> Firehose (optional Lambda transform) -> OpenSearch** (near real time), or **Kinesis Data Streams -> custom Lambda -> OpenSearch** (real time).

> [!tip] Exam
> Search / partial match / free text -> OpenSearch.

---

## 05 - EMR
(src: 22/05-EMR)

- **Elastic MapReduce**: provisions **Hadoop clusters** (hundreds of EC2 instances) for big data; bundles Spark, HBase, Presto, Flink, etc. and handles setup/configuration. Auto scaling and **Spot** integration. Use cases: data processing, machine learning, web indexing, big data.
- Node types:

| Node | Role | Notes |
|---|---|---|
| Master | manages cluster, coordinates, monitors health | long running |
| Core | runs tasks **and stores data** | long running |
| Task | runs tasks only | optional; good for **Spot** |

- Purchasing: On-Demand = reliable, predictable, never terminated; **Reserved** (min 1 year) = big savings, EMR uses them automatically, good for master and core nodes; Spot = cheaper but can be terminated, good for task nodes.
- Deployment: **long-running** clusters (pair with reserved instances) or **transient** clusters (tear down after the job).

> [!tip] Exam
> Hadoop / Spark big data cluster -> EMR.

---

## 06 - QuickSight
(src: 22/06-QuickSight)

- Serverless, ML-powered **BI** service for interactive dashboards; fast, auto scaling, embeddable, **per-session pricing**. Use cases: business analytics, visualizations, ad hoc analysis.
- Sources: RDS, Aurora, Athena, Redshift, S3, OpenSearch, Timestream; SaaS (Salesforce, Jira); third-party DBs (Teradata, on-premises JDBC); imported files (Excel, CSV, JSON, TSV, log formats).
- **SPICE** = in-memory computation engine; works **only for data imported into QuickSight**, not for live connections to other databases.
- **Column-level security (CLS)**: Enterprise edition.
- **Users** (Standard edition) and **groups** (Enterprise edition only) exist only inside QuickSight, **not IAM users** (IAM is for administration).
- **Analysis** vs **dashboard**: a dashboard is a **read-only snapshot** of an analysis preserving filters, parameters, controls and sorting; publish it and share with users/groups; viewers can also see the underlying data.

> [!tip] Exam
> Common pairings: QuickSight + Athena, QuickSight + Redshift. SPICE = imported data only.

---

## 07 - Glue
(src: 22/07-Glue)

- Managed, serverless **ETL** service to prepare/transform data for analytics (e.g. S3 or RDS -> transform -> Redshift).
- Exam favorite: **convert CSV to Parquet** with Glue ETL into an output S3 bucket so Athena is faster. Automate: S3 event notification -> Lambda (or EventBridge) -> triggers Glue ETL job.
- **Glue Data Catalog**: crawlers scan S3, RDS, DynamoDB, on-premises JDBC sources and store databases, tables, columns, data types as metadata; used by Glue jobs, **Athena**, **Redshift Spectrum**, **EMR**.
- Other features: **Job Bookmarks** (avoid reprocessing old data), **Glue DataBrew** (clean/normalize with pre-built transformations), **Glue Studio** (GUI to create/run/monitor jobs), **Glue Streaming ETL** (built on Spark Structured Streaming; reads Kinesis Data Streams, Kafka, MSK).

---

## 08 - Lake Formation
(src: 22/08-Lake Formation)

- Data lake = central place for all data for analytics. Lake Formation sets one up in **days instead of months**; discovers, cleanses, transforms, ingests data; automates collecting, cleansing, moving, cataloging and **de-duplication (ML transforms)**; handles structured and unstructured data.
- **Blueprints** to ingest from S3, RDS, on-premises relational or NoSQL DBs. Data lake lives in **S3**.
- It is a **layer on top of Glue** (crawlers, ETL, catalog) plus security settings; consumers include Athena, Redshift, EMR, Spark.
- **Fine-grained access control at row and column level**, centralized in Lake Formation instead of scattering security across S3 policies, RDS/Aurora users, Athena, QuickSight.

> [!tip] Exam
> Centralized permissions with row/column-level security across analytics tools -> Lake Formation.

---

## 09 - Amazon Managed Service for Apache Flink
(src: 22/09-Amazon Managed Service for Apache Flink)

- Formerly **Kinesis Data Analytics for Apache Flink**. Flink = framework (Java, SQL, Scala) for real-time stream processing.
- Reads from **Kinesis Data Streams** and **Amazon MSK**; **cannot read from Firehose** (exam trick).
- AWS provisions compute, parallel computation, automatic scaling, and application backups (checkpoints and snapshots). Any Flink programming feature can be used.
- Only for processing data streams.

> [!tip] Exam
> Flink does not read Firehose.

---

## 10 - Managed Service for Apache Flink - Hands On
(src: 22/10-Amazon Managed Service for Apache Flink - Hands On)

- Console tour only; not needed for the exam. Options: **streaming application** (Apache Flink runtime version, upload your own application, monitor via Flink dashboard) or **Studio notebook** for interactive streaming queries.
- **SQL applications (legacy)** are listed separately on the left (used e.g. to read from Firehose); AWS recommends Studio / Apache Flink instead.

### Hands-on steps
1. Kinesis console -> Managed Apache Flink (Data Analytics) options `[screen action]`.
2. Streaming application: pick Flink runtime, name, upload application (not done, none available).
3. Studio notebook: quick notebook for streaming queries (not created).
4. Legacy SQL applications: create application, then "real-time analytics" to write SQL (only shown).

---

## 11 - MSK - Managed Streaming for Apache Kafka
(src: 22/11-MSK - Managed Streaming for Apache Kafka)

- **Kafka** = alternative to Kinesis for streaming. **MSK** = fully managed Kafka: create/update/delete clusters, manages **broker and Zookeeper nodes**, deployed in your **VPC across up to 3 AZ**, automatic recovery from common Kafka failures, data on **EBS** for as long as you want. **MSK Serverless**: no servers or capacity management.
- Kafka model: producers write to a **topic** (replicated across brokers); consumers pull from it.
- Kinesis Data Streams vs MSK:

| Aspect | Kinesis Data Streams | Amazon MSK |
|---|---|---|
| Message size | 1 MB limit | 1 MB default, configurable higher (e.g. 10 MB) |
| Unit | Shards | Topics with partitions |
| Scaling | Shard split / merge | Add partitions only (cannot remove) |
| In-flight encryption | TLS | Plaintext or TLS |
| At-rest encryption | Yes | Yes |
| Retention | up to 1 year | as long as you pay for EBS (over 1 year possible) |

- Consume from MSK with: Managed Service for Apache Flink, Glue streaming ETL (Spark Streaming), Lambda (MSK as event source), or a custom Kafka consumer on EC2/ECS/EKS.

> [!tip] Exam
> Kafka -> MSK. MSK can only add partitions; Kinesis uses shard split/merge.

---

## 12 - Big Data Ingestion Pipeline
(src: 22/12-Big Data Ingestion Pipeline)

- Goal: fully serverless, managed pipeline: real-time collection, transformation, SQL querying, reports in S3, warehouse + dashboards.
- Flow: **IoT devices -> IoT Core** (manages devices) **-> Kinesis Data Streams -> Kinesis Data Firehose** (offloads to S3 every **1 minute**, the lowest frequency; optional **Lambda** transform) **-> ingestion S3 bucket -> (optional SQS queue) -> Lambda -> Athena SQL query -> reporting S3 bucket -> QuickSight** (or load into **Redshift**, which can also feed QuickSight).
- Notes: S3 can notify SQS, SNS or Lambda directly (SQS step is optional); Athena is serverless SQL with results stored back to S3; Redshift for deeper analytics.

---

Not covered: none; all 12 lectures have transcripts.
