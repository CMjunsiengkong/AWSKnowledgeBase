---
course: Ultimate AWS Certified Solutions Architect Associate 2026
chapter: 31
chapter_title: WhitePapers and Architectures
version: B (by chapter)
services: [Well-Architected Framework, Well-Architected Tool, Trusted Advisor, Architecture Center, Solutions Library]
tags: [aws, saa-c03, well-architected, trusted-advisor, whitepapers]
---

# 31 - WhitePapers and Architectures

Related: [[Well-Architected & Trusted Advisor]] · [[Classic Solution Architectures]] · [[Disaster Recovery & Backup]] · [[CloudFormation]]

## Chapter summary
- Whitepapers are not heavily tested, but know the **Well-Architected Framework** and Disaster Recovery basics.
- **Six pillars**: operational excellence, security, reliability, performance efficiency, cost optimization, sustainability. Know the names.
- Pillars are a **synergy**, not trade-offs.
- **Well-Architected Tool**: pick workload, apply lenses, answer questions per pillar, get risks and improvement plan.
- **Trusted Advisor**: account assessment in six categories; full checks and Support API need Business or Enterprise support (per lecture).
- AWS **Architecture Center** (2,000+ diagrams) and **Solutions Library** (diagrams plus CloudFormation templates) are real-world references.

---

## 01 - WhitePaper Section Introduction
(src: 31/01-WhitePaper Section Introduction)
- Section covers: Well-Architected Framework, Well-Architected Tool, Trusted Advisor, reference architecture resources, and the Disaster Recovery whitepaper (boring but important for the exam; covered elsewhere in the course).

## 02 - AWS Well-Architected Framework & Tool
(src: 31/02-AWS Well-Architected Framework & Well-Architected Tool)

### Design principles
- Stop guessing capacity (use Auto Scaling).
- Test systems at production scale (spin up, then shut down).
- Automate for easier experimentation (e.g. CloudFormation templates across environments).
- Allow evolutionary architectures (e.g. EC2 + load balancer evolving to API Gateway + Lambda).
- Drive architecture using data.
- Improve through game days (e.g. simulate flash-sale load).

### Six pillars
1. Operational excellence 2. Security 3. Reliability 4. Performance efficiency 5. Cost optimization 6. Sustainability

> [!tip] Exam
> Know the six pillar names (not details). They are a synergy: improving operational excellence likely improves cost optimization; sustainability tends to improve performance efficiency.

### Well-Architected Tool
- Review architectures against the six pillars and adopt best practices. Flow: select workload -> answer questions -> review against pillars -> get advice (videos, docs, reports, dashboard).
- Lenses: Well-Architected Framework lens, FTR lens, serverless lens, SaaS lens, custom lenses.
- Question counts mentioned (may change): 11 on operational excellence, 10 on security, 6 on sustainability.
- Results show high/medium risks; the improvement plan links to the framework section; milestones track progress.

### Hands-on steps
1. Well-Architected Tool -> Define workload: name, review owner, environment (production), regions (e.g. one plus us-west-2); non-AWS regions and account IDs optional.
2. Apply lens: Well-Architected Framework.
3. Start reviewing; answer questions per pillar (answer several, then Save and continue).
4. Open the lens overview -> risks (e.g. 3 high risks) -> Improvement plan -> follow the link into the framework text.

## 03 - AWS Trusted Advisor Overview + Hands-On
(src: 31/03-AWS Trusted Advisor Overview + Hands-On)

- High-level **account assessment**; nothing to install. Example checks: EBS public snapshots, RDS public snapshots, root account usage, open security group ports, S3 bucket permissions.
- **Six categories**: cost optimization, performance, security, fault tolerance, service limits, operational excellence.
- **Core checks** are available to all; the **full set** needs **Business or Enterprise** support. With those plans you also get **programmatic access via the AWS Support API**.
- Service limits can be viewed in Trusted Advisor (Auto Scaling groups, CloudFormation stacks, DynamoDB read/write capacity).

> [!tip] Exam
> Trusted Advisor: 6 categories; full checks + Support API require Business/Enterprise support.

[verify] "Business or Enterprise support plan" for full checks.
> [!warning] Correction [note]
> AWS docs now say full checks are available with Business Support+, Enterprise Support or Unified Operations plans; Basic support gets all Service Limits checks plus selected Security and Fault tolerance checks. Developer, Business and Enterprise On-Ramp plans are being discontinued on 1 January 2027. Source: [AWS Trusted Advisor](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html)

### Hands-on steps
1. Open Trusted Advisor -> Recommendations; review the summary (e.g. open bucket, security group rules with unrestricted ports).
2. Open each category; on Basic support only security core checks are available.
3. Check Service limits.

## 04 - Examples of Architecture
(src: 31/04-Examples of Architecture - AWS Certified Solutions Architect Associate)

- Course covered classic (EC2, ELB, RDS, ElastiCache) and serverless (S3, Lambda, DynamoDB, CloudFront, API Gateway) architectures; two more resources:
- **AWS Architecture Center**: reference architectures and diagrams (over 2,000), filterable; many in PDF/HTML with CloudFormation templates (e.g. automating DR for relational databases, WordPress best practices).
- **AWS Solutions Library**: vetted solutions and guidance; architecture diagrams, implementation guides, **CloudFormation templates** to launch in the console, source on GitHub. Examples: Live Streaming on AWS, Serverless Image Handler. Browse by industry, cross-industry, technology, organization type.

### Hands-on steps
1. Browse the Architecture Center, filter, open a PDF (e.g. DR automation).
2. Browse the Solutions Library, open "Live Streaming on AWS" and view architecture, implementation guide, CloudFormation template.

Not covered: none (all lectures have transcripts).
