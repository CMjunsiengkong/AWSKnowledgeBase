---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Storage Options Compared
version: C (by service)
source_chapters: [16 (lecture 10)]
related: [S3, EBS, EFS, FSx, EC2, Storage Gateway, Data Transfer, Snow Family]
tags: [aws, saa-c03, storage, comparison, summary]
---

# AWS Storage Options Compared

Concept-only summary. Chapter context in [[16 - AWS Storage Extras]] (Version B).

## 1. Which storage for what
(src: 16/10-All AWS Storage Options Compared)

| Need | Service | Note |
|---|---|---|
| Object storage (specific API, great for anything AWS) | [[S3]] | archive objects with **S3 Glacier** |
| Block storage for **one EC2 instance at a time** | [[EBS]] | **Multi-Attach** for io1/io2; types gp3, io2, etc. |
| Very high IOPS, **physical** local storage (not network) | **EC2 Instance Store** ([[EC2]]) | ephemeral |
| Network file system for **Linux**, multi-AZ, POSIX | [[EFS]] | |
| Windows file server | [[FSx]] for Windows | |
| HPC Linux file system (Lustre client) | FSx for Lustre | |
| Highest OS compatibility network file system | FSx for NetApp ONTAP | |
| Managed ZFS file system | FSx for OpenZFS | |
| Bridge on-prem to AWS | [[Storage Gateway]] | S3 and FSx File Gateway (files), Volume Gateway (volumes backed up in cloud), Tape Gateway (tape backups) |
| FTP / FTPS / SFTP on top of S3 or EFS | AWS Transfer Family ([[Data Transfer]]) | |
| Scheduled sync on-prem -> AWS or AWS -> AWS | DataSync ([[Data Transfer]]) | |
| Move large data physically (no network capacity) | Snowcone, Snowball, Snowmobile ([[Snow Family]]) | **Snowcone has a DataSync agent bundled** |
| Databases | not a file/object store | indexing and querying needs: covered in the database chapters |

[verify] Snowcone / Snowmobile as current options and "FSx File Gateway".
> [!warning] Correction [note]
> AWS says Snowball Edge is available only to existing customers since 7 November 2025; the same FAQ does not mention Snowcone or Snowmobile, so their current status is unconfirmed. Source: [AWS Snowball FAQs](https://aws.amazon.com/snowball/faqs/). The Storage Gateway guide still lists FSx File Gateway ([source](https://docs.aws.amazon.com/storagegateway/latest/vgw/WhatIsStorageGateway.html)); the instructor's claim of retirement elsewhere is unconfirmed.

> [!tip] Exam
> Pick by access pattern: object = S3, block = EBS / instance store, shared Linux file = EFS, specialized file = FSx, hybrid = Storage Gateway, protocol transfer = Transfer Family, scheduled sync = DataSync, offline bulk = Snow family.

## Not included here
- Database selection (covered later in the course).
