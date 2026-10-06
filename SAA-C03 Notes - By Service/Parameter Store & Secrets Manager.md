---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Parameter Store & Secrets Manager
version: C (by service)
source_chapters: [26 (lectures 08-11)]
related: ["KMS, CloudHSM & ACM", IAM, EventBridge, CloudFormation, Lambda, RDS & Aurora, Systems Manager]
tags: [aws, saa-c03, ssm, parameter-store, secrets-manager, kms, secrets, rotation]
---

# SSM Parameter Store and Secrets Manager

Concept-only note from chapter 26. CLI/console walkthroughs are in [[26 - Security & Encryption]] (Version B). Encryption itself: [[KMS, CloudHSM & ACM]]. Parameter Store is a feature of [[Systems Manager]].

## 1. SSM Parameter Store
(src: 26/08-SSM Parameter Store Overview, 26/09-SSM Parameter Store Hands On (CLI))
- **Secure storage for configuration and secrets**; optionally **encrypted with KMS** (SecureString).
- **Serverless, scalable, durable**, easy SDK; **version tracking** of parameters; security via **IAM**; notifications via **EventBridge**; **CloudFormation** can use parameters as stack inputs.
- Flow: app (e.g. EC2 instance role or Lambda role) -> IAM check -> plain text parameter returned; encrypted parameter -> Parameter Store calls KMS to decrypt, so the app also needs **access to the KMS key**.

### Hierarchy
- Parameters are stored by path, e.g. `/my-department/my-app/dev/db-url`, `/my-department/my-app/prod/db-password`.
- Simplifies IAM: grant access to a whole department, an app, or one environment path (e.g. a Dev Lambda role only sees `.../dev/`, a Prod Lambda role only `.../prod/`).
- Retrieval concepts: get by name (`get-parameters`) or **by path** (`get-parameters-by-path`; **recursive** flag needed to include nested levels); **with-decryption** flag decrypts SecureString values (requires KMS permission).
- Types: **String**, **StringList**, **SecureString** (KMS-encrypted; default key `alias/aws/ssm` or your own, can be in another account).
- You can read **Secrets Manager secrets through Parameter Store** via a reference.
- **Public parameters** issued by AWS, e.g. the latest Amazon Linux 2 AMI ID per region.

[verify] Public parameter example refers to "Amazon Linux 2".
> [!warning] Correction [note]
> Amazon Linux 2 reaches end of support on 30 June 2026 and AWS recommends Amazon Linux 2023. Source: [Amazon Linux 2 FAQs](https://aws.amazon.com/amazon-linux-2/faqs/).

### Tiers

| | Standard | Advanced |
|---|---|---|
| Max parameters | 10,000 | 100,000 |
| Max value size | 4 KB | 8 KB |
| Parameter policies | no | **yes** |
| Share with other accounts | no | yes |
| Cost | free | **$0.05 per advanced parameter per month** |

> [!warning] Correction [note]
> AWS confirms the counts, sizes, policies and sharing (and adds that a standard parameter can be upgraded but an advanced one cannot be downgraded). The $0.05 price is not stated on that page. Source: [Choosing parameter tiers in Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-advanced-parameters.html).

### Parameter policies (advanced only)
- Assign a **TTL (expiration)** to force updating/deleting sensitive data such as passwords; **multiple policies** at a time.
- With **EventBridge**: e.g. an expiration notification **15 days before** expiry; a **no-change notification** if a parameter has not changed for **20 days**.

> [!tip] Exam
> Parameter Store = cheap/free config and secrets, hierarchical paths, KMS optional, TTL only on advanced tier.

## 2. Secrets Manager
(src: 26/10-AWS Secrets Manager - Overview, 26/11-AWS Secrets Manager - Hands On)
- Newer service **meant for storing secrets**. Differs from Parameter Store by **forced rotation every X days** and **automated secret generation on rotation** via a **Lambda function** you define.
- **Out-of-the-box integration** with RDS (MySQL, PostgreSQL, SQL Server), Aurora, DocumentDB, Redshift and other databases: credentials stored in Secrets Manager and rotated, and the database is updated on rotation.
- Secret types: database credentials or **other** (arbitrary key/value, or plain text JSON, e.g. API keys).
- Encrypted with **KMS** (default key or your own).
- **Resource policy** (similar to an S3 bucket policy) can allow **cross-account** access.
- **Multi-region secrets**: replicate to other regions (each with its key); replicas stay in sync with the primary. Uses: promote a replica to a standalone secret if the primary region fails, multi-region apps, disaster recovery, and replicated RDS databases using the matching secret.
- Pricing: 30-day free trial, then **$0.40 per secret per month** and **$0.05 per 10,000 API calls**.

> [!warning] Correction [note]
> $0.40/secret/month and $0.05 per 10,000 API calls are confirmed. The 30-day trial was not: the page now describes new-customer Free Tier credits (up to $200, free plan for 6 months). Source: [AWS Secrets Manager Pricing](https://aws.amazon.com/secrets-manager/pricing/).
[verify] "30-day free trial."

> [!tip] Exam
> Secrets with **rotation** or **RDS/Aurora integration** -> Secrets Manager.

## 3. Comparison

| | Parameter Store | Secrets Manager |
|---|---|---|
| Purpose | config and secrets | secrets |
| Automatic rotation | no (TTL policies on advanced tier only) | **yes**, via Lambda |
| Database integration | none out of the box | RDS, Aurora, DocumentDB, Redshift |
| Multi-region replication | not mentioned | **yes** |
| Encryption | optional (KMS) | KMS |
| Cost | standard free; advanced $0.05/param/month | $0.40/secret/month + API calls |

## Not included here
- 26/09 and 26/11 console and CLI step narration dropped.
- No diagrams: no AWS reference fetched. All lectures have transcripts.
