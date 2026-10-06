---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: GuardDuty, Inspector & Macie
version: C (by service)
source_chapters: [26 (lectures 19-21)]
related: [EventBridge, CloudTrail & Config, VPC, S3, Lambda, "Containers (ECS, ECR, EKS)", Systems Manager]
tags: [aws, saa-c03, guardduty, inspector, macie, security, threat-detection, vulnerability, pii]
---

# GuardDuty, Inspector and Macie

Concept-only note from chapter 26 (lectures 19-21). See [[26 - Security & Encryption]] in Version B. Findings from all three flow to [[EventBridge]].

## 1. Amazon GuardDuty
(src: 26/19-Amazon GuardDuty)
- **Intelligent threat discovery** for your AWS accounts using **machine learning, anomaly detection and third-party data**.
- **One click to enable**, **30-day trial**, **no software to install**.
- Always-on input data:
  - **CloudTrail event logs**: unusual API calls, unauthorized deployments; both **management events** (e.g. create VPC subnet) and **S3 data events** (get/list/delete object).
  - **VPC Flow Logs**: unusual internet traffic and IP addresses.
  - **DNS logs**: EC2 instances sending **encoded data in DNS queries** (sign of compromise).
- **Optional features**: EKS audit logs and runtime monitoring, RDS and Aurora login events, EBS volumes, Lambda network activity, S3 logs (more over time).
- Findings generate an **EventBridge** event; rules trigger **Lambda** automation or **SNS** notifications.
- Has a dedicated finding for **cryptocurrency attacks**.

> [!warning] Correction [note]
> The 30-day trial is confirmed (applies per new account per region; also for newly enabled protection plans). Source: [Amazon GuardDuty pricing](https://aws.amazon.com/guardduty/pricing/).

> [!tip] Exam
> GuardDuty: threat detection from CloudTrail + VPC Flow + DNS logs; crypto attack finding; EventBridge for automation.

## 2. Amazon Inspector
(src: 26/20-Amazon Inspector)
- Automated **vulnerability and security assessments**, **continuous**, only on:

| Target | What is checked | Notes |
|---|---|---|
| **EC2 instances** | known OS/package vulnerabilities (CVE) and **unintended network reachability** | uses the **Systems Manager agent** |
| **Container images in ECR** | package vulnerabilities (CVE) | scanned as images are **pushed** |
| **Lambda functions** | vulnerabilities in function **code and package dependencies** | assessed at deployment |

- Re-runs automatically when the **CVE database** is updated; each finding gets a **risk score** for prioritization.
- Reports to **AWS Security Hub** and sends events to **EventBridge**.

> [!tip] Exam
> Inspector = only EC2, ECR images and Lambda; CVE and network reachability. Not for S3 or other resources.

## 3. Amazon Macie
(src: 26/21-Amazon Macie)
- Fully managed **data security and privacy** service using **ML and pattern matching** to discover and protect **sensitive data**, notably **PII**.
- Analyzes **S3 buckets** only (you choose the buckets); notifies via **EventBridge**, then **SNS**, **Lambda** etc.; one click to enable.

> [!tip] Exam
> Sensitive data/PII in S3 -> Macie.

## 4. Comparison

| Service | Question it answers | Looks at |
|---|---|---|
| GuardDuty | "Is someone attacking or misusing my account?" | CloudTrail, VPC Flow Logs, DNS logs (+ optional sources) |
| Inspector | "Are my workloads vulnerable?" | EC2, ECR images, Lambda |
| Macie | "Where is my sensitive data?" | S3 |

## Not included here
- Other security services (WAF, Shield, Firewall Manager) are covered in their own note. No diagrams: no AWS reference fetched. All lectures have transcripts.
