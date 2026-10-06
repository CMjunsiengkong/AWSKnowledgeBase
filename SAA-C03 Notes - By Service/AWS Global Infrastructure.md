---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: AWS Global Infrastructure
version: C (by service)
source_chapters: [03]
related: [IAM, EC2, Route 53]
tags: [aws, saa-c03, regions, availability-zones, edge-locations, global-services]
---

# AWS Global Infrastructure

Concept-only note from chapter 03. Console tour steps are in [[03 - Getting Started with AWS]] (Version B). Related: [[IAM]], [[EC2]], [[Route 53]].

## 1. AWS background
(src: 03/01-AWS Cloud Overview - Regions & AZ)
- Launched internally at amazon.com in **2002**; first public offering **SQS in 2004**; **2006** relaunch with SQS, S3, EC2; then expansion beyond the US.
- Customers named: Dropbox, Netflix, Airbnb, NASA and others.
- Market stats given: Gartner Magic Quadrant leader for 13 consecutive years; **$90B revenue (2023)**; **~31% market share (Q1 2024)** vs Microsoft ~25%; over 1 million active users.
- Use cases: enterprise IT, backup/storage, big data analytics, website hosting, mobile/social backends, gaming servers.

[verify] Revenue, market share and user figures are time-stamped (2023/Q1 2024) and not exam-relevant. Unconfirmed.

## 2. Regions
(src: 03/01)
- A **Region** = a cluster of data centers in a geographic area (e.g. Ohio, Singapore, Sydney, Tokyo). Named with a code, e.g. `us-east-1`, `eu-west-3`; regions are connected by AWS's **private network**.
- **Most services are region-scoped**: using a service in another region is like starting fresh there.

**How to choose a region:**

| Factor | Why |
|---|---|
| Compliance | data may have to stay in the country (e.g. French data in the French region) |
| Latency | deploy close to users |
| Service availability | not all regions have all services (check the regional services table) |
| Pricing | varies by region |

> [!tip] Exam
> Region choice = compliance, latency, available services, price.

## 3. Availability Zones
(src: 03/01)
- Each region has several **AZs**: lecture says **minimum 3, maximum 6, usually 3**. Example: Sydney `ap-southeast-2` has `ap-southeast-2a`, `2b`, `2c`.
- An AZ = **one or more discrete data centers** with redundant power, networking and connectivity; AZs are **isolated from each other's disasters** and linked by **high-bandwidth, ultra-low-latency networking**.

[verify] "Minimum three, maximum six AZs per region."
> [!warning] Correction [note]
> AWS states each Region has **at least three** independent, physically separate AZs, confirming the minimum. The "maximum six" figure was not found on that page. Source: [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

## 4. Points of presence (edge locations)
(src: 03/01)
- Used to deliver content to end users with lowest latency (covered with CloudFront later).
- Lecture figure: **more than 400 points of presence in 90 cities across 40 countries**.

[verify] "400+ points of presence in 90 cities, 40 countries."
> [!warning] Correction [note]
> AWS's page now lists **750+ CloudFront POPs** and 15 regional edge caches, plus Local Zones and Wavelength Zones; it also lists 39 Regions and 124 AZs. Source: [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

## 5. Global vs region-scoped services
(src: 03/01, 03/03-Tour of the AWS Console & Services in AWS)
- **Global services** (no region selector, same view everywhere): **IAM, Route 53, CloudFront, WAF**.
- **Region-scoped**: **EC2, Elastic Beanstalk, Lambda, Rekognition** and most others; resources you see depend on the selected region.
- Choose a region geographically close to you for the course and stay in the same one. Check the **regional services table** if a service is missing in your region.

> [!tip] Exam
> Global: IAM, Route 53, CloudFront, WAF. Most others (EC2, Lambda, Beanstalk...) are regional.

## Not included here
- Console UI tour (region selector, service search, recently visited) dropped as hands-on.
- 03/02 "[Important] AWS Console UI Update" is not in this note's source list (03/01-03 per SPEC lists it; it is a short UI notice with no concept content).
