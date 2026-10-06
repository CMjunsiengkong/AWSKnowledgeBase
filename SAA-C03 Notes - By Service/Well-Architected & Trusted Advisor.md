---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Well-Architected & Trusted Advisor
version: C (by service)
source_chapters: [31]
related: [Cost Management & Billing, Classic Solution Architectures, CloudFormation, ASG]
tags: [aws, saa-c03, well-architected, trusted-advisor, pillars, support-plans]
---

# Well-Architected & Trusted Advisor

Concept-only note on the AWS Well-Architected Framework / Tool and AWS Trusted Advisor. Console walkthroughs are omitted; see [[31 - WhitePapers and Architectures]] (Version B). Related: [[Classic Solution Architectures]], [[CloudFormation]], [[Cost Management & Billing]].

## 1. AWS Well-Architected Framework
(src: 31/02-AWS Well-Architected Framework & Well-Architected Tool)
General design principles:
- **Stop guessing capacity**: use Auto Scaling ([[ASG]]).
- **Test at production scale**: spin up big infrastructure, test, shut down an hour later.
- **Automate** to make experimentation easy (e.g. a CloudFormation template deployed to many environments, [[CloudFormation]]).
- **Allow evolutionary architectures** (e.g. EC2 + load balancer evolving into API Gateway + Lambda).
- **Drive architecture using data**.
- **Improve through game days** (e.g. simulate flash-sale pressure on production).

### The six pillars (know the names)
1. Operational Excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization
6. Sustainability

- Pillars are a **synergy, not a trade-off**: e.g. better operational excellence tends to improve cost optimization; more sustainable tends to be more performance efficient.

> [!tip] Exam
> Memorize the six pillar names only; the exam does not expect deep pillar detail from this course.

## 2. AWS Well-Architected Tool
(src: 31/02)
- Tool to **review workloads against the six pillars** and adopt best practices.
- Flow: **define a workload** (name, review owner, environment, Regions incl. non-AWS Regions, account IDs) -> **apply lenses** -> **answer questions** per pillar -> receive **advice** (videos, documentation, reports), a **dashboard** of results, **risks** (high / medium) with an **improvement plan** that links to the relevant framework section; set **milestones**.
- **Lenses** = sets of questions: Well-Architected Framework lens, FTR lens, Serverless lens, SaaS lens, and **custom lenses**.
- Question counts mentioned for the framework lens (can change over time): e.g. 11 on operational excellence, 10 on security, 6 on sustainability.
- Answer all questions for all pillars; once confident, the workload is considered production ready, compliant and well architected.

## 3. AWS Trusted Advisor
(src: 31/03-AWS Trusted Advisor Overview + Hands-On)
- No installation: a **high-level account assessment** giving recommendations. Example checks: **EBS public snapshots**, **RDS public snapshots**, use of the **root account**, S3 bucket permissions (global access), **security groups** with unrestricted access to specific ports.
- Six categories: **Cost optimization, Performance, Security, Fault tolerance, Service limits, Operational excellence**.
- **Core checks** (everyone) vs **full set of checks**: the full set needs a **Business or Enterprise support plan**. With those plans you also get **programmatic access via the AWS Support API**.
- Without a paid plan (demo): only **Security** core checks and **Service limits** (e.g. ASG, CloudFormation stacks, DynamoDB capacity) were available; cost optimization, performance, fault tolerance and operational excellence showed nothing.

[verify] "Full set of checks requires a Business or Enterprise support plan."
> [!warning] Correction [note]
> AWS's current Trusted Advisor documentation says all checks are available with **Business Support+, Enterprise Support or Unified Operations**, while **Basic Support** gets all **Service Limits** checks and selected Security and Fault tolerance checks. It also announces that **Developer Support and Business Support will be discontinued on January 1, 2027** (replaced by Business Support+) and Enterprise On-Ramp is being merged into Enterprise Support. Exam questions may still use the older names. Source: [AWS Trusted Advisor (AWS Support User Guide)](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html).

> [!tip] Exam
> Trusted Advisor = automated checks in 6 categories; full checks + Support API need Business/Enterprise support. Well-Architected Tool = review workloads against 6 pillars.

## Not included here
- 31/01 Section Introduction (admin only).
- 31/04 Examples of Architecture -> [[Classic Solution Architectures]].
- Console walkthroughs of both tools dropped (workload definition clicks, Trusted Advisor screens); concepts retained.
- No lecture here lacks a transcript.
