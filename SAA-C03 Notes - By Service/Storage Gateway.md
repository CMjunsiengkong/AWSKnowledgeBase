---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Storage Gateway
version: C (by service)
source_chapters: [16 (lectures 06-07)]
related: [S3, EBS, FSx, Data Transfer, Storage Options Compared, Disaster Recovery & Backup]
tags: [aws, saa-c03, storage-gateway, hybrid-cloud, file-gateway, volume-gateway, tape-gateway]
---

# AWS Storage Gateway

Concept-only note. The console walk-through of creating a gateway is in [[16 - AWS Storage Extras]] (Version B).

## 1. Hybrid cloud and the gateway
(src: 16/06-Storage Gateway Overview)
- **Hybrid cloud** = part of the infrastructure on AWS, part on-premises (long migration, security/compliance, strategy to use the cloud for elastic workloads only).
- S3 is a proprietary object API (not NFS like EFS), so **Storage Gateway is the bridge between on-premises data and cloud storage**.
- AWS native storage: **block** (EBS, instance store), **file** (EFS, FSx), **object** (S3, Glacier).
- Use cases: disaster recovery, backup and restore, tiered storage (cold data in the cloud, warm on-prem), **on-prem cache for low-latency access** to cloud data.
- The gateway runs **in your data center** (VM on VMware, Hyper-V, Linux KVM, or hardware); it can also be hosted on **EC2**, which puts the cache in your AWS account.

## 2. Gateway types
(src: 16/06, 16/07-Storage Gateway Hands On)

| Type | Protocol | Backed by | Notes |
|---|---|---|---|
| **S3 File Gateway** | **NFS, SMB** (translated to **HTTPS** to S3) | S3: Standard, Standard-IA, One Zone-IA, Intelligent-Tiering (**not Glacier directly**) | **most recently used data cached**; **IAM role per gateway**; **SMB integrates with Active Directory**; lifecycle policy moves objects to Glacier |
| **Volume Gateway** | **iSCSI** block storage | S3, with **EBS snapshots**; restorable as EBS volumes in AWS | **Cached volumes**: primary data in S3, frequently used data cached locally (low latency). **Stored volumes**: **entire dataset on-prem**, scheduled/asynchronous backup to S3 |
| **Tape Gateway** | **iSCSI VTL** (virtual tape library) | S3 and **Glacier / Glacier Deep Archive** | replaces physical tape backups; works with leading backup software vendors |

- The three flows: file share (NFS/SMB) -> S3 File Gateway -> S3; application servers (iSCSI) -> Volume Gateway -> S3 -> EBS snapshots/volumes; backup software (iSCSI VTL) -> Tape Gateway -> S3 -> Glacier tiers.
- The comparison lecture also mentions an **FSx File Gateway** (see below).

[verify] "FSx File Gateway is greyed out because it is going away, so forget about it."
> [!warning] Correction [note]
> The current AWS Storage Gateway user guide still lists "file-based (S3 File Gateway and FSx File Gateway)" solutions, so I could not confirm the retirement the instructor mentions; treat as unconfirmed. Source: [What is Volume Gateway? (AWS Storage Gateway)](https://docs.aws.amazon.com/storagegateway/latest/vgw/WhatIsStorageGateway.html). The same page also lists Nutanix AHV as a supported hypervisor.

> [!tip] Exam
> On-prem NFS/SMB access to S3 = S3 File Gateway. On-prem block volumes backed up to cloud = Volume Gateway (cached vs stored). Physical tape replacement = Tape Gateway. S3 File Gateway cannot use Glacier directly; use a lifecycle policy. Need an on-prem VM/hardware to run the gateway.

## Not included here
- 16/07 console tour of gateway creation (platform options per type are summarized above).
