---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Organizations & Identity Federation
version: C (by service)
source_chapters: [25]
related: [IAM, CloudTrail & Config, EventBridge]
tags: [aws, saa-c03, organizations, scp, tag-policies, identity-center, directory-service, active-directory, control-tower]
---

# Organizations & Identity Federation

Concept-only note from chapter 25. Lab steps (where any exist) are in [[25 - IAM Advanced]] (Version B). Related: [[IAM]].

## 1. AWS Organizations
(src: 25/01-Organizations - Overview)
- **Global service** to manage multiple AWS accounts. **Management account** (main) + **member accounts**; an account can belong to **only one organization**.
- Benefits: **consolidated billing** (one payment method on the management account); **aggregated usage pricing** discounts (e.g. EC2, S3 volume); **sharing Reserved Instance and Savings Plans discounts** across accounts; **API to automate account creation**.
- Structure: **root OU** (contains the management account) with nested **OUs**, organized by business unit, environment (prod/test/dev) or project; mix freely.
- Advantages: better isolation than multiple VPCs in one account; enforce tagging standards for billing; enable **CloudTrail on all accounts** to a central S3; send CloudWatch Logs to a central logging account; automatic **cross-account admin roles**.

### Service Control Policies (SCP)
- IAM-style policy applied to **OUs or accounts** to restrict what users and roles (including root of member accounts) can do.
- **Do not apply to the management account** (it always has full admin power).
- SCPs do not grant permissions; you need an **explicit Allow at every level** (root OU, each OU, account), e.g. FullAWSAccess. An explicit Deny at any level wins.
- Examples from the lecture: Sandbox OU with FullAWSAccess + deny S3 -> accounts inside cannot use S3; account with deny EC2 on top -> neither S3 nor EC2; test OU with only allow EC2 -> its account can only use EC2. Policy styles: **deny list** (allow all, deny DynamoDB) or **allow list** (allow only EC2 and CloudWatch).

> [!tip] Exam
> SCP = guardrail, never applies to the management account, needs explicit allow down the tree, deny always wins.

## 2. Tag policies
(src: 25/03-Organizations - Tag Policies)
- Standardize **tag keys and allowed values** across all accounts in the organization; audit tagged resources, generate compliance reports, optionally **prevent non-compliant tagging operations** on specified services/resources. No effect on resources that have no tags.
- Good with cost allocation tags and attribute-based access control. Use **EventBridge** to detect non-compliant tags.

> [!tip] Exam
> "Keep tags consistent across accounts" -> tag policies in Organizations.

## 3. IAM Identity Center
(src: 25/07-AWS IAM Identity Center)
- Successor (renamed) of **AWS Single Sign-On**: **one login** for all AWS accounts in the organization, business cloud apps (Salesforce, Box, Microsoft 365, any **SAML 2.0** app) and **EC2 Windows instances**.
- **Identity store**: built-in Identity Center store, or third-party IdP (Active Directory, OneLogin, Okta...).
- **Permission sets** = collection of IAM policies assigned to users/groups for chosen accounts; Identity Center creates a matching IAM role in each target account that the user assumes on login. Example: developers group gets an admin permission set on dev accounts and a read-only set on prod accounts.
- **Application assignments** (URLs, certificates, metadata supported out of the box).
- **Attribute-based access control (ABAC)**: fine-grained permissions from user attributes in the Identity Center store (cost center, title, locale); define permission sets once and change access by changing attributes.
- Set up in the management account of the organization.

> [!tip] Exam
> One login into multiple AWS accounts -> IAM Identity Center.

## 4. AWS Directory Services
(src: 25/08-AWS Directory Services, 25/09-AWS Directory Services - Hands On)
- **Microsoft Active Directory (AD)**: database of objects (users, computers, printers, file shares, security groups) on Windows Server with AD Domain Services; centralized security; objects in trees, trees grouped in a **forest**; domain controllers authenticate logins across machines.

| Option | What it is | MFA | Users managed | On-prem link |
|---|---|---|---|---|
| **AWS Managed Microsoft AD** | real AD in AWS; **Standard up to 30,000 objects, Enterprise up to 500,000** | yes | locally in AWS (and/or on-prem) | **two-way trust** with on-prem AD |
| **AD Connector** | **proxy** to on-prem AD; **small up to 500 users, large up to 5,000** | yes | only on-prem AD | proxies requests |
| **Simple AD** | AD-compatible managed directory (not Microsoft AD), standalone | not mentioned | in AWS | **cannot join** on-prem AD |

- Use case: AD in AWS lets EC2 Windows instances join the domain and share logins.
- Exam picks: proxy to on-prem -> **AD Connector**; manage users in cloud with MFA / trust with on-prem -> **AWS Managed Microsoft AD**; no on-prem AD, basic needs -> **Simple AD**.
- Console also lists **Cognito User Pools** (not a Directory Service type).
- **Identity Center integration**: with AWS Managed Microsoft AD it is out of the box. For a **self-managed (on-prem) directory**: (1) AWS Managed Microsoft AD + **two-way trust** with the on-prem AD (also lets you manage users in cloud), or (2) **AD Connector** proxy (some extra latency).

[verify] Simple AD "does not support MFA" is not stated in the lecture; and Managed AD sizes may change.
> [!warning] Correction [note]
> AWS documents Standard Edition up to ~30,000 directory objects and Enterprise up to ~500,000 (approximate), confirming the lecture. It also states Simple AD (Samba 4) does **not** support MFA, trust relationships, schema extensions or LDAPS, and AD Connector can enable MFA via an existing RADIUS infrastructure. A newer **Hybrid Edition** of Managed Microsoft AD (to extend self-managed AD) is listed as well. The AD Connector size limits were not shown on that page. Source: [What is AWS Directory Service?](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html)

## 5. AWS Control Tower
(src: 25/10-AWS Control Tower)
- Easy setup and governance of a **secure, compliant multi-account environment** following best practices; uses **AWS Organizations** to create accounts.
- Benefits: automated environment setup, ongoing policy management with **guardrails**, detect and remediate violations, **dashboard** of compliance.
- **Guardrails**:

| Type | Mechanism | Example |
|---|---|---|
| **Preventive** | **SCPs** (Organizations) | restrict accounts to only us-east-1 and eu-west-2 |
| **Detective** | **AWS Config** rules deployed in member accounts | detect untagged resources; non-compliance triggers an **SNS** topic (notify admin or invoke **Lambda** to remediate, e.g. add tags) |

> [!tip] Exam
> Control Tower = Organizations + guardrails. Preventive = SCP, detective = Config.

## Not included here
- Lectures without transcript: 25/02 Organizations - Hands On.
- 25/09 hands-on narration dropped; only its concept content (editions, sizes) is kept.
- IAM policies, boundaries, evaluation logic -> [[IAM]].
