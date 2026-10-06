---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: "Athena, Glue & Lake Formation"
version: C (by service)
source_chapters: [22 (01-02, 07-08)]
related: ["S3", "Redshift, EMR, OpenSearch & QuickSight", "Lambda", "EventBridge", "Streaming Analytics - Flink & MSK", "DynamoDB"]
tags: [aws, saa-c03, athena, glue, lake-formation, parquet, etl, data-catalog, data-lake]
---

# Athena, Glue & Lake Formation

Concept-only note. Lab steps: [[22 - Data & Analytics]] (Version B). Related: [[S3]], [[Redshift, EMR, OpenSearch & QuickSight]].

## 1. Amazon Athena
(src: 22/01-Athena, 22/02-Athena Hands On (concept only))
- **Serverless** query service to analyze data **in S3 without moving it**, using standard **SQL**; built on the **Presto** engine.
- Formats: CSV, JSON, **ORC, Avro, Parquet**, others.
- **Pricing: fixed amount per TB of data scanned.** No provisioning.
- Typically paired with **QuickSight** for reports/dashboards.
- Use cases: ad hoc queries, BI, analytics, reporting, and analyzing AWS logs (VPC Flow Logs, load balancer logs, CloudTrail trails).
- Needs an S3 location for query results (setup detail from the lab).

> [!tip] Exam
> Analyze data in S3 with a serverless SQL engine -> **Athena**.

### Performance and cost tuning (exam-tested)
Because cost = data scanned, scan less:

| Technique | Detail |
|---|---|
| **Columnar format** | **Parquet or ORC** - only needed columns are scanned; huge improvement. Convert with **Glue** ETL (e.g. CSV -> Parquet) |
| **Compress** data | smaller retrievals |
| **Partition** datasets | S3 path with `key=value` folders, e.g. `/year=1991/month=1/day=1/`; filters on those columns read only matching folders |
| **Bigger files** | fewer, larger files (e.g. **128 MB and over**) reduce overhead vs many small files |

### Federated Query
- Query **non-S3 sources** (relational/non-relational, custom, AWS or on-premises) via a **Data Source Connector = Lambda function** (one per source).
- Examples: CloudWatch Logs, DynamoDB, RDS, ElastiCache, DocumentDB, Redshift, Aurora, SQL Server, MySQL, HBase on EMR, on-prem DBs; joins across sources are possible; **results can be stored in S3**.

## 2. AWS Glue
(src: 22/07-Glue)
- Managed, **serverless ETL** (extract, transform, load) service to prepare data for analytics.
- Example: extract from S3 / RDS, transform (filter, add columns), load into **Redshift**.
- **Exam favourite:** convert CSV in S3 to **Parquet** (columnar) with a Glue ETL job into an output bucket so Athena performs better. Automation: S3 event notification -> **Lambda** (or **EventBridge**) -> triggers the Glue ETL job.


### Glue Data Catalog
- Catalogs datasets: **crawlers** connect to S3, RDS, DynamoDB, or JDBC databases (e.g. on-prem) and write **metadata** (databases, tables, columns, types) into the catalog.
- Used by Glue jobs for ETL, and by **Athena**, **Redshift Spectrum** and **EMR** for data/schema discovery - central to many services.

### Other Glue features (high level)
| Feature | Purpose |
|---|---|
| **Job Bookmarks** | prevent reprocessing old data on a new ETL run |
| **DataBrew** | clean and normalize data with pre-built transformations |
| **Glue Studio** | GUI to create, run, monitor ETL jobs |
| **Streaming ETL** | built on **Apache Spark Structured Streaming**; runs ETL as streaming jobs from Kinesis Data Streams, Kafka or MSK (see [[Streaming Analytics - Flink & MSK]]) |

## 3. AWS Lake Formation
(src: 22/08-Lake Formation)
- **Data lake** = central place for all data for analytics. Lake Formation makes setup take **days instead of months**.
- Discovers, cleans, transforms and ingests data; automates collecting, cleansing, moving, cataloging and **de-duplication (ML transforms)**. Combines structured and unstructured data.
- **Blueprints** (out of the box) migrate data from S3, RDS / relational, on-prem DBs, NoSQL into the lake (stored in **S3**).
- It is a **layer on top of Glue** (crawlers, ETL, catalog), but you do not interact with Glue directly.
- Consumers: **Athena, Redshift, EMR**, Spark and other analytics tools.
- Key exam value: **centralized permissions with fine-grained, row-level and column-level access control**, instead of scattering security across Athena, QuickSight, S3 bucket policies, RDS/Aurora users.

> [!tip] Exam
> "Centralized data-lake permissions with row/column-level security across Athena, QuickSight, etc." -> **Lake Formation**.

## Not included here
- 22/02 Athena Hands On: console steps omitted (create results bucket, create database/table over S3 access logs, example queries counting requests by HTTP status, spotting 404/403s). The idea that Athena can analyze S3 access logs is kept above.
- 22/03-06 and 22/09-12: see [[Redshift, EMR, OpenSearch & QuickSight]] and [[Streaming Analytics - Flink & MSK]].
