---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Data Transfer
version: C (by service)
source_chapters: [16 (lectures 08-09), 28 (lecture 10)]
related: [S3, EFS, FSx, Snow Family, Migration Services, VPC Connectivity, Route 53, Storage Gateway]
tags: [aws, saa-c03, transfer-family, datasync, large-dataset, snowball, direct-connect]
---

# Data Transfer (Transfer Family, DataSync, large datasets)
Concept-only note. Lab-free chapters in Version B: [[16 - AWS Storage Extras]] and [[28 - Disaster Recovery & Migrations]]. Related: [[Snow Family]], [[Migration Services]].
Concept-only note. Version B notes:  [[28 - Disaster Recovery & Migrations]] and  - AWS Storage Extras]] (Version B). Related: [[Snow Family]], [[Migration Services]].

## 1. AWS Transfer Family
(src: 16/08-AWS Transfer Family)
- Send data in and out of **S3 or EFS** using **FTP-family protocols** instead of S3 APIs or NFS.
- Protocols: **FTP** (unencrypted), **FTPS** (FTP over SSL, encrypted), **SFTP** (secure FTP, encrypted).
- **Fully managed**, scalable, reliable, highly available.
- **Pricing: per provisioned endpoint per hour + per GB transferred in/out.**
- Users: store credentials in the service, or integrate **Active Directory, LDAP, Okta, Amazon Cognito, or a custom source**.
- Architecture: clients reach the endpoint (optionally with your own host name via [[Route 53]]); the service assumes an **IAM role** to read/write S3 or EFS transparently.
- Use cases: sharing files, public datasets, CRM, ERP.

## 2. AWS DataSync
(src: 16/09-DataSync - Overview)
- Move **large amounts of data** to and from on-premises/other clouds into AWS, or **between AWS storage services**. Appears frequently on the exam.
- On-prem/other cloud sources via **NFS, SMB, HDFS** etc.: requires the **DataSync agent** installed on-prem; the agent connects **encrypted** to the DataSync service. **AWS-to-AWS needs no agent.**
- Destinations/sources: **S3 (any storage class, including Glacier)**, **EFS**, **FSx**; sync can go **either direction**.
- **Scheduled, not continuous**: hourly, daily or weekly.
- **Preserves file permissions and metadata** (NFS POSIX, SMB): the exam answer when metadata must be kept.
- One agent task can use up to **10 Gbps**; a **bandwidth limit** can be set.
- Snowcone ships with a DataSync agent already installed (src: 16/10).

> [!tip] Exam
> FTP/FTPS/SFTP to S3/EFS = Transfer Family. Scheduled sync preserving metadata/permissions, on-prem or AWS-to-AWS = DataSync (agent needed for NFS/SMB). DataSync is not continuous.

## 3. Transferring large datasets into AWS
(src: 28/10-Transferring Large Datasets into AWS)
Example: **200 TB** to move, **100 Mbps** internet connection.

| Method | Setup | Time for 200 TB | Notes |
|---|---|---|---|
| **Public internet / Site-to-Site VPN** | immediate | about **16 million seconds, ~185 days** (200 TB -> GB -> MB -> x8 megabits / 100 Mbps) | usable right away, impractical here |
| **Direct Connect 1 Gbps** | long one-time setup, **about 1 month** | about **18.5 days** (10x faster) | see [[VPC Connectivity]] |
| **Snowball** | order and ship | about **1 week end to end** | one-off large transfers; can be **combined with DMS** for database changes afterwards |

- **Ongoing replication**: Site-to-Site VPN, Direct Connect, **DMS**, or **DataSync**; Snowball is for one-off bulk loads.
- Exam asks for the easiest, fastest or most reliable method given data size and constraints.

> [!tip] Exam
> Huge one-off dataset + weak bandwidth = Snowball. Ongoing sync = DataSync / DMS / VPN / Direct Connect. Direct Connect takes about a month to provision.

## Not included here
- No hands-on lectures in this note; Transfer Family and DataSync have no lab in the transcripts.
