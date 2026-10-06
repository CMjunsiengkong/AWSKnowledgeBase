---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Snow Family
version: C (by service)
source_chapters: [16 (lectures 01-03)]
related: [S3, Data Transfer, Migration Services, Storage Options Compared, EC2, Lambda]
tags: [aws, saa-c03, snowball, snowcone, snowmobile, edge-computing, data-migration]
---

# AWS Snow Family

Concept-only note. The console walk-through of ordering a device is in [[16 - AWS Storage Extras]] (Version B). Other transfer options: [[Data Transfer]], [[Migration Services]].

## 1. What it is
(src: 16/01-AWS Snow Family Overview)
- Highly secure, portable **physical devices** to **collect and process data at the edge** and **migrate data in and out of AWS** (petabyte scale).
- Two **Snowball Edge** devices:

| Device | Capacity | Purpose |
|---|---|---|
| **Edge Storage Optimized** | **210 TB** | storage / data migration |
| **Edge Compute Optimized** | **28 TB** | compute at the edge |

- Compute devices can run **EC2 instances and Lambda functions** locally ([[EC2]], [[Lambda]]).
- Snowcone and Snowmobile are named only in the storage comparison lecture (see [[Storage Options Compared]]): **Snowcone ships with a DataSync agent bundled**.

[verify] Availability of Snowball Edge devices.
> [!warning] Correction [note]
> AWS states that effective **7 November 2025 Snowball Edge devices are available only to existing customers**; new customers are pointed to DataSync, AWS Data Transfer Terminal or partner solutions. The 210 TB Storage Optimized figure is confirmed ("approximately 210 TB"); the 28 TB Compute Optimized figure and the status of Snowcone/Snowmobile are not stated on that page (unconfirmed). Source: [AWS Snowball FAQs](https://aws.amazon.com/snowball/faqs/).

## 2. Use case 1: data migration
(src: 16/01)
- Network transfer is slow: **100 TB over a 1 Gbps link takes about 12 days**.
- Limited bandwidth, high network cost, shared bandwidth, unstable connection, or **a transfer that would take over a week** -> use Snowball.
- Flow: order a device -> it is shipped to you -> load your data -> ship back -> AWS **imports into S3** (export jobs also exist: data loaded from S3 onto the device). Direct upload to S3 is simpler but may consume all your bandwidth.
- Order options (console, concept only): job type **import into S3 / export from S3 / local compute and storage only**; device type; per-day on-demand pricing; IAM service role allowing the device to write to your S3 bucket; encryption; shipping speed (one- or two-day); job status notifications.

## 3. Use case 2: edge computing
(src: 16/01)
- Process data where it is created, with **no or limited internet and compute**: truck on the road, ship at sea, mining station.
- Use the **Compute Optimized** device to run EC2/Lambda: **pre-process data, machine learning at the edge, transcode media**, then send data back to AWS.

## 4. Snowball into Glacier
(src: 16/03-Architecture- Snowball into Glacier)
- **Snowball cannot import directly into Glacier.** Import into **Amazon S3**, then an **S3 lifecycle policy** transitions the objects to Glacier.

> [!tip] Exam
> Slow/limited network, large (TB-PB) one-off data -> Snowball. Edge locations without connectivity -> Snowball Edge Compute Optimized. Snowball -> Glacier = S3 first + lifecycle policy.

## Not included here
- 16/02-AWS Snow Family Hands On (console ordering walk-through; only concepts above retained).
