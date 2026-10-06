---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: FSx
version: C (by service)
source_chapters: [16 (lectures 04-05)]
related: [EFS, EBS, S3, Storage Gateway, Storage Options Compared, VPC Connectivity]
tags: [aws, saa-c03, fsx, lustre, windows-file-server, netapp-ontap, openzfs]
---

# Amazon FSx

Concept-only note. The console tour of the four file system options is in [[16 - AWS Storage Extras]] (Version B).

## 1. What it is
(src: 16/04-Amazon FSx)
- Launch **third-party high-performance file systems** as a **fully managed service** (like RDS but for file systems). Four to know: **Windows File Server, Lustre, NetApp ONTAP, OpenZFS**.

## 2. Comparison
(src: 16/04, 16/05-Amazon FSx - Hands On)

| | Windows File Server | Lustre | NetApp ONTAP | OpenZFS |
|---|---|---|---|---|
| Protocols | **SMB**, Windows **NTFS** | Lustre client (Linux) | **NFS, SMB, iSCSI** | **NFS** (multiple versions) |
| Main use | Windows shared drive | **ML, HPC**, video processing, financial modeling, EDA | move workloads already on **ONTAP or on-prem NAS** | move workloads already on **ZFS** |
| Scale / performance | tens of GB/s, millions of IOPS, hundreds of PB | hundreds of GB/s, millions of IOPS, sub-ms latency | broad; auto-grow/shrink | up to **1 million IOPS, < 0.5 ms** latency |
| Storage | **SSD** (latency-sensitive: DBs, media, analytics) or **HDD** (cheaper: home dirs, CMS) | **SSD** (low latency, IOPS-intensive, small random files) or **HDD** (throughput, large sequential); SSD costs more | - | - |
| HA | **Multi-AZ** option; daily backups to S3 | scratch/persistent, single AZ | Multi-AZ or Single AZ | - |
| Special | **Active Directory**, ACLs, user quotas, **DFS** to join on-prem Windows file servers; **mountable on Linux EC2** too | **seamless S3 integration**: read S3 as a file system, write results back | replication, snapshots, compression, **deduplication**, **point-in-time instantaneous cloning** | snapshots, compression, **cloning**, **no deduplication** |
| OS compatibility | Windows, Linux | Linux | **Linux, Windows, macOS**, VMware Cloud on AWS, WorkSpaces, AppStream, EC2, ECS, EKS | Linux, Mac, Windows |
| On-prem access | private connection | **VPN or Direct Connect** | - | - |

- Windows: SMB + Active Directory (AWS-managed or self-managed AD).
- Lustre = **Linux + cluster**. Keyword **HPC** -> FSx for Lustre.
- ONTAP / OpenZFS clone: quickly clone a file system for testing new workloads (staging).

## 3. Lustre deployment options
(src: 16/04)

| | **Scratch** | **Persistent** |
|---|---|---|
| Purpose | temporary, short-term processing, lowest cost | long-term storage, sensitive data |
| Replication | **none**: data lost if the underlying server fails | replicated **within the same AZ**; failed server replaced transparently within minutes |
| Performance | very high burst: **6x persistent**, e.g. **200 MB/s per TB** | lower |
| Note | optional S3 bucket as data repository | Lustre lives in a **single AZ** |

> [!tip] Exam
> Windows/SMB/AD -> FSx for Windows. HPC/ML/S3-integrated -> Lustre (scratch = temporary, persistent = long-term). NAS/ONTAP migration, NFS+SMB+iSCSI -> NetApp ONTAP. ZFS migration -> OpenZFS. Only the four types need to be known.

## Not included here
- 16/05 console exploration of options (Multi-AZ/Single-AZ, storage type, AD integration, deduplication/compression settings are covered in the table above).
