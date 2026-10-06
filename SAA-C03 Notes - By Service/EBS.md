---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: EBS
version: C (by service)
source_chapters: [07 (EBS lectures 01-04, 08-10, 14)]
related: [EC2, EFS, "KMS, CloudHSM & ACM", S3, Storage Options Compared]
tags: [aws, saa-c03, ebs, snapshots, volume-types, multi-attach, encryption]
---

# Amazon EBS (Elastic Block Store)

Concept-only note. Lab steps (creating/attaching volumes, snapshots, Recycle Bin rule, encryption walkthrough, cleanup) are in [[07 - EC2 Instance Storage]] (Version B). Related storage: [[EFS]], instance store in [[EC2]], overview in [[Storage Options Compared]].

## 1. What EBS is
(src: 07/01-EBS Overview, 07/02-EBS Hands On)
- **Network drive** attachable to an instance while it runs ("network USB stick"); data **persists after the instance is terminated**, so you can mount the same volume on a new instance.
- Because it uses the network, there can be a little **latency**; it can be detached and re-attached to another instance quickly (handy for failover).
- **One instance at a time** (at the CCP level); the exception is Multi-Attach (section 4). One instance can have **several** volumes.
- **Bound to one Availability Zone**: a volume in `us-east-1a` cannot attach to an instance in `us-east-1b`. Move across AZ via snapshot (section 3).
- **Provisioned capacity**: you choose size (GB) and IOPS in advance, are billed for what you provision, and can increase it over time.
- Volumes can exist **unattached** and be attached on demand.

### Delete on Termination
- Controls what happens to the volume when the instance is terminated.
- **Default: enabled for the root volume, disabled for any other attached volume.**
- Use case: disable it on the root volume to **keep the root data** after termination.

> [!tip] Exam
> Root volume is deleted on termination by default; extra volumes are kept. Disable the flag to preserve the root volume. EBS = one AZ, one instance (except io1/io2 Multi-Attach).

## 2. Volume types
(src: 07/08-EBS Volume Types)
Six types; defined by size, throughput and IOPS. **Only gp2, gp3, io1, io2 can be boot (root) volumes**; st1 and sc1 cannot.

| Type | Class | Key numbers | Use cases |
|---|---|---|---|
| **gp3** | General purpose SSD | baseline **3,000 IOPS + 125 MB/s**; IOPS up to **80,000** and throughput up to **2,000 MB/s**, **set independently** of size | boot volumes, virtual desktops, dev/test, medium databases |
| **gp2** | General purpose SSD | size and IOPS **linked**: **3 IOPS per GB**, max **16,000 IOPS** (reached at ~5,334 GB); small volumes **burst to 3,000 IOPS** | same as gp3 |
| **io1** | Provisioned IOPS SSD | up to **64,000 IOPS on Nitro** instances, **32,000 on others**; IOPS independent of size, ratio ~**50:1** IOPS per GB | critical, sustained-IOPS databases; Multi-Attach |
| **io2 Block Express** | Provisioned IOPS SSD | **sub-millisecond latency**, up to **256,000 IOPS**, ratio **1,000:1** IOPS per GB; supports Multi-Attach | highest performance, >64,000 IOPS, IO-sensitive databases |
| **st1** | Throughput-optimized HDD | max **500 MB/s**, **500 IOPS**, up to 16 TB | big data, data warehousing, log processing |
| **sc1** | Cold HDD | max **250 MB/s**, **250 IOPS**, up to 16 TB; **lowest cost** | infrequently accessed / archive data |

- To exceed **32,000 IOPS** you need an **EC2 Nitro** instance with io1/io2.
- The instructor says exact numbers need not be memorized: know SSD general purpose vs provisioned IOPS (databases) vs HDD (throughput / lowest cost).

[verify] gp3 max IOPS 80,000 and 2,000 MB/s; gp2 max 16,000 IOPS.
> [!warning] Correction [note]
> Confirmed against AWS documentation: gp3 baseline 3,000 IOPS / 125 MiB/s, up to 80,000 IOPS and 2,000 MiB/s; gp2 3 IOPS per GiB, 100 to 16,000 IOPS, burst to 3,000 IOPS. Source: [General Purpose SSD volumes](https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html). (The other io1/io2/st1/sc1 figures were not re-checked.)

> [!tip] Exam
> Boot volume = gp2/gp3/io1/io2. Database needing >32,000 IOPS = io1/io2 on Nitro. Cheapest cold storage = sc1; streaming/throughput workloads = st1. gp3 sets IOPS independently of size; gp2 does not.

## 3. Snapshots
(src: 07/03-EBS Snapshots, 07/04-EBS Snapshots - Hands On)
- **Snapshot = point-in-time backup** of a volume. Detaching the volume first is recommended but not required.
- Snapshots can be **copied across AZ and across Regions** (e.g. disaster recovery in another Region). **This is how you move a volume to another AZ**: snapshot, then restore (create a volume from it) in the target AZ.
- When creating a volume from a snapshot you can also choose the type/size and enable encryption.

| Feature | What it does | Key facts |
|---|---|---|
| **Snapshot Archive** | move snapshot to an archive tier | **up to 75% cheaper**; restore takes **24 to 72 hours** |
| **Recycle Bin** | deleted snapshots land in the bin instead of being permanently deleted | retention rule **1 day to 1 year**; recoverable from accidental deletion; also protects **AMIs**; rule can be locked or unlocked |
| **Fast Snapshot Restore (FSR)** | forces full initialization of the snapshot so there is **no latency on first use** | useful for very large snapshots; **expensive** |

- Backups consume I/O, so avoid running them under heavy application traffic (src: 07/13).

> [!tip] Exam
> Cross-AZ/Region move = snapshot. Accidental deletion protection = Recycle Bin. No first-use latency = Fast Snapshot Restore (costly). Cheaper long-term snapshots = Archive tier (24-72 h restore).

## 4. Multi-Attach
(src: 07/09-EBS Multi-Attach)
- Attach the **same volume to multiple EC2 instances in the same AZ**, each with **full read/write**.
- **Only io1 and io2** volumes.
- **Up to 16 instances** at a time (exam number).
- Requires a **cluster-aware file system** (not standard XFS / EXT4).
- Use cases: higher availability for clustered Linux applications (e.g. Teradata), applications that manage concurrent write operations.

[verify] "Multi-Attach: up to 16 instances, io1/io2, same AZ, cluster-aware file system."
> [!warning] Correction [note]
> Confirmed. AWS adds that the instances must be Nitro-based, boot volumes cannot use Multi-Attach, and Windows supports io2 only. Source: [Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html).

## 5. Encryption
(src: 07/10-EBS Encryption)
- An encrypted volume gives: **data at rest encrypted**, **data in flight** between instance and volume encrypted, **all snapshots encrypted**, **all volumes created from those snapshots encrypted**.
- Handled **transparently** by EC2 and EBS; **minimal latency impact**; uses **KMS keys (AES-256)** (see [[KMS, CloudHSM & ACM]]).
- A snapshot of an unencrypted volume is unencrypted; **copying a snapshot lets you enable encryption** (and choose the KMS key).

**Encrypting an existing unencrypted volume:**
1. Create a snapshot of the volume.
2. **Copy** the snapshot with encryption enabled.
3. Create a new volume from the encrypted snapshot (it is encrypted).
4. Attach it to the original instance.

Shortcut: when creating a volume from an unencrypted snapshot you can enable encryption on the fly.

> [!tip] Exam
> Unencrypted volume -> snapshot -> encrypted copy -> new volume. Encryption has almost no latency cost and uses KMS (AES-256).

## Not included here
- Hands-on narration: 07/02 (EBS Hands On), 07/04 (Snapshots Hands On), 07/14 (Section Cleanup: delete file systems, instances, volumes, snapshots, extra security groups to avoid charges).
- AMI (07/05-06) and Instance Store (07/07) -> [[EC2]]. EFS and EFS vs EBS -> [[EFS]].
