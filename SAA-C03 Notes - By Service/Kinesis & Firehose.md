---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Kinesis & Firehose
version: C (by service)
source_chapters: [17]
related: [SQS, SNS, Amazon MQ & Messaging Comparison, Streaming Analytics - Flink & MSK, S3, Lambda, KMS]
tags: [aws, saa-c03, kinesis, data-streams, firehose, streaming, shards]
---

# Kinesis Data Streams & Amazon Data Firehose

Concept-only note. Lab steps (create stream, put-record, delivery stream to S3) are in [[17 - Decoupling Applications - SQS, SNS, Kinesis, Amazon MQ]] (Version B). Flink and MSK are in [[Streaming Analytics - Flink & MSK]]. Related: [[SQS]], [[SNS]], [[Amazon MQ & Messaging Comparison]].

## 1. Kinesis Data Streams (KDS)
(src: 17/11-Amazon Kinesis Data Streams)
- Collect and store **streaming data in real time** (keyword: **real-time**). Examples: click streams, IoT devices (connected bicycle), server metrics and logs.
- **Producers**: your application code (SDK), the **Kinesis Agent** on servers (logs/metrics), **KPL** (Kinesis Producer Library, optimized high-throughput producers).
- **Consumers**: your own code, **Lambda**, **Amazon Data Firehose**, **Managed Service for Apache Flink**; **KCL** (Kinesis Client Library) for optimized consumers (handles shard reading for you).
- Features:
  - **Retention up to 365 days**; data persisted so consumers can **reprocess/replay**; data **cannot be deleted**, it only expires.
  - Record data up to **10 MB**, typical use is many small records.
  - **Ordering** is preserved for records with the **same partition key** (same shard).
  - Security: **KMS at-rest**, **HTTPS in flight**.

[verify] Record size "up to 10 MB" and on-demand figures below.
> [!warning] Correction [note]
> AWS confirms max record payload **10 MiB**, retention max **365 days**, per shard **1 MB/s or 1,000 records/s write** and **2 MB/s read**, and on-demand default of **4 MB/s write / 8 MB/s read** for new streams. It also states on-demand streams scale up to **10 GB/s write / 20 GB/s read in us-east-1, us-west-2 and eu-west-1** (200 MB/s write / 400 MB/s read in other Regions), higher than the 200 MB/s the hands-on lecture mentions. Source: [Quotas and limits (Kinesis Data Streams)](https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html).

## 2. Capacity modes
(src: 17/11, 17/12-Amazon Kinesis Data Streams - Hands On, 17/15-SQS vs SNS vs Kinesis)

| | Provisioned | On-demand |
|---|---|---|
| Capacity | you choose the **number of shards** (1 to 1,000s) | none to manage |
| Per shard | **write 1 MB/s or 1,000 records/s; read 2 MB/s** | default about **4 MB/s or 4,000 records/s** in |
| Scaling | manual (increase/decrease shards); monitor throughput | automatic, based on peak throughput of the **past 30 days** |
| Billing | **per shard per hour** (+ PUT cost) | **per stream per hour + data in/out** |
| Max (lecture) | scale by adding shards | 200 MB/s and 200,000 records/s write; 400 MB/s read per consumer with enhanced fan-out |

- Sizing example: 10,000 records/s or 10 MB/s write needs **10 shards**. A **shard estimator** helps.
- No free tier for either mode. Lecture quotes **$0.05 per shard-hour** [verify] (unconfirmed; check current pricing).
- **Shard** = unit of stream capacity; more shards = more throughput. Data with the same partition key goes to the same shard.

## 3. Consumption modes
(src: 17/12, 17/15)
- **Shared (standard) consumption**: consumers **pull**; **2 MB/s per shard shared** across all consumers.
- **Enhanced fan-out**: Kinesis **pushes** data; **2 MB/s per shard per consumer**; more consumer apps on the same stream, higher throughput.
- Low-level API reading requires picking a shard and a shard iterator (`TRIM_HORIZON` = from the beginning); KCL does this for you.

## 4. Amazon Data Firehose
(src: 17/13-Amazon Data Firehose, 17/14-Amazon Data Firehose - Hands On)
- Formerly **Kinesis Data Firehose** (renamed because it does more than Kinesis). **Fully managed, serverless, auto-scaling, pay for what you use**.
- Loads streaming data into destinations; **near real-time** (keyword for the exam) because of an internal **buffer** flushed by **size or time**.
- **Sources**: your apps/clients (SDK), Kinesis Agent, **Kinesis Data Streams**, CloudWatch Logs and Events, **AWS IoT**, EventBridge and other AWS services (also SNS can send to Firehose).
- **Optional transformation** with a **Lambda function** (e.g. CSV -> JSON, filter, decompress); built-in **format conversion to Parquet or ORC** and **compression (gzip, snappy, zip, Hadoop-compatible snappy)**. Input formats include CSV, JSON, Parquet, Avro, text, binary.
- **Destinations**: AWS: **S3, Redshift, OpenSearch** (remember these three); partners: **Datadog, Splunk, New Relic, MongoDB**; **custom HTTP endpoint**. Can back up all or only failed records to an S3 bucket.
- Buffer (hands-on): flush when size or interval is reached; size default **5 MB** (can go up to 128 MB for efficiency or smaller for speed); interval from **60 seconds (minimum in demo) up to 900 seconds**, 300 seconds as the default shown. Buffering can optionally be disabled.
- An **IAM role** is created so Firehose can read the source and write the destination. Errors go to CloudWatch Logs. Firehose only delivers data sent **after** the delivery stream is active.

[verify] Buffer defaults and ranges (5 MB / 300 s as defaults, 60 s min, 900 s max as stated in the hands-on).
> [!warning] Unconfirmed
> I could not retrieve an AWS source for Firehose buffer defaults and ranges. Current AWS documentation may use different defaults and wider ranges (for example buffer interval up to 3,600 s), so check the Firehose buffering hints page before relying on these numbers. Do not memorize them for the exam; remember only "buffer by size or time -> near real-time".

## 5. Data Streams vs Firehose

| | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Purpose | collect streaming data | **load** streaming data into destinations (S3, Redshift, OpenSearch, HTTP, partners) |
| Code | you write producers and consumers | managed, optional Lambda transform |
| Latency | **real-time** | **near real-time** (buffered) |
| Capacity | provisioned or on-demand | automatic scaling |
| Storage / replay | up to 365 days, **replay supported** | **no storage, no replay** |

> [!tip] Exam
> Real-time, replay, custom consumers -> Kinesis Data Streams. Near real-time load into S3/Redshift/OpenSearch with no code -> Data Firehose.

## Not included here
- 17/12 and 17/14 hands-on (CloudShell commands, base64 decoding, delivery stream wizard, cleanup): only the concepts above were kept.
- Analytics on streams (Managed Service for Apache Flink, MSK) -> [[Streaming Analytics - Flink & MSK]].
- Messaging comparison and Amazon MQ -> [[Amazon MQ & Messaging Comparison]].
- All 17/11-14 lectures have transcripts.
