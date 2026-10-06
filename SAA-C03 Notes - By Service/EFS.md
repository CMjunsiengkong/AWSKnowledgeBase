---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: EFS
version: C (by service)
source_chapters: [07 (EFS lectures 11-13)]
related: [EBS, EC2, KMS, Storage Options Compared, Storage Gateway, FSx]
tags: [aws, saa-c03, efs, nfs, performance-modes, throughput-modes, storage-classes]
---

# Amazon EFS (Elastic File System)

Concept-only note. Lab steps (creating the file system, mounting on two instances in different AZs) are in [[07 - EC2 Instance Storage]] (Version B). Block storage: [[EBS]].

## 1. What EFS is
(src: 07/11-Amazon EFS, 07/12-Amazon EFS - Hands On)
- **Managed NFS** (network file system) mountable on **many EC2 instances across multiple AZs** at once: highly available, scalable.
- **Expensive: about 3x the cost of a gp2 EBS volume**, but **pay per use**, no capacity planning; grows automatically up to **petabyte scale**.
- Scale: **thousands of concurrent NFS clients**, **10 GB/s+** throughput.
- Access control: **security group** on the file system; the NFS port is **2049** (a rule allowing NFS from the instances' security group). One **mount target per AZ**, each in a subnet.
- **Linux only** (Linux-based AMIs, **POSIX** file system, standard file API); **not Windows**.
- **Encryption at rest** with KMS.
- Use cases: content management, web serving, data sharing, WordPress.

## 2. Performance mode (chosen at creation)
(src: 07/11, 07/12)

| Mode | Behavior | Use |
|---|---|---|
| **General Purpose** (default) | low latency | web servers, CMS, latency-sensitive |
| **Max I/O** | higher latency, **higher throughput, highly parallel** | big data, media processing |

- With **Elastic** throughput, General Purpose is the only performance mode. The recommended setup now is **General Purpose + Elastic**.

## 3. Throughput mode
(src: 07/11, 07/12)

| Mode | Behavior |
|---|---|
| **Bursting** | throughput scales with storage size; e.g. **1 TB = 50 MB/s, bursting to 100 MB/s** (numbers only illustrative) |
| **Provisioned** | set throughput regardless of storage, e.g. **1 GB/s for 1 TB**; you pay for it in advance |
| **Elastic** (recommended) | scales up and down automatically with workload, pay for what you use; up to **3 GB/s reads and 1 GB/s writes**; best for **unpredictable workloads** |

- The console groups Elastic and Provisioned under "Enhanced", but there are only three options to remember: **Bursting, Provisioned, Elastic**.

## 4. Storage classes and lifecycle management
(src: 07/11, 07/12)
- **Standard** (frequent access), **EFS-IA** (infrequent: lower storage price, retrieval fee), **Archive** (rarely accessed, a few times per year: cheapest).
- **Lifecycle policies** move files after N days without access (example: not accessed 60 days -> EFS-IA; console defaults shown: 30 days to IA, 90 days to Archive, optional move back to Standard on first access).
- Availability and durability:

| Option | Layout | Use |
|---|---|---|
| **Regional / Standard** | multi-AZ | production, resilient to AZ failure |
| **One Zone** (and One Zone-IA) | single AZ, backups available | development, cheaper; data unreachable if the AZ fails |

- Right use of storage classes saves **up to 90%**. Automatic backups are recommended to keep enabled.

## 5. EFS vs EBS
(src: 07/13-EFS vs EBS)

| | EBS | EFS |
|---|---|---|
| Attach | one instance (except io1/io2 Multi-Attach) | **hundreds of instances**, across AZs |
| AZ | locked to one AZ; move via snapshot | mount targets in many AZs |
| OS | Linux/Windows | **Linux only** (POSIX) |
| Performance | gp2: IOPS grows with size; gp3/io1: independent | modes above |
| Price | lower | higher (use storage tiers) |
| Root volume | deleted on termination by default | n/a |
| Instance store | physically attached; lost with the instance | - |

> [!tip] Exam
> Shared file system across AZs for Linux = EFS (NFS, POSIX, SG port 2049). Cost saving = EFS-IA / Archive with lifecycle policies; One Zone for dev. Unpredictable throughput = Elastic; Windows = use FSx for Windows ([[FSx]]) instead.

## Not included here
- Hands-on narration: 07/12 (creating the file system, launching Instance A/B, writing a shared file), 07/14 (cleanup).
- EBS content -> [[EBS]].
