---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Cost Management & Billing
version: C (by service)
source_chapters: [05, 30]
related: [EC2, CloudFormation, DynamoDB, RDS & Aurora, ASG, SNS, Systems Manager, IAM]
tags: [aws, saa-c03, billing, budgets, cost-explorer, cost-anomaly-detection, instance-scheduler, free-tier]
---

# Cost Management & Billing

Concept-only note on the billing console, budgets, Cost Explorer, Cost Anomaly Detection and Instance Scheduler on AWS. Click-by-click steps (budget creation, stack deployment) are in [[05 - EC2 Fundamentals]] and [[30 - Other Services]] (Version B). Purchasing options (RI, Savings Plans, Spot) are in [[EC2]].

## 1. Billing console, free tier and budgets
(src: 05/01-AWS Budget Setup)
- **IAM users (even administrators) cannot see billing data by default**: the **root user** must enable *IAM user and role access to billing information* in the account settings ([[IAM]]). Activation can take a few refreshes.
- Useful views:
  - **Billing home**: month-to-date cost, forecasted total for the month, last month's total, cost by month;
  - **Bills**: charges **by service** (and by Region) for a chosen month; drill down to see what drives cost (the example shows NAT Gateway, EBS and Elastic IP charges under EC2);
  - **Free Tier**: current vs forecasted usage; if the forecast goes red you will be billed, so turn off what is running.
- **Budgets** alert you when thresholds are reached (simplified templates):
  - **Zero spend budget**: email as soon as spend reaches **$0.01**;
  - **Monthly cost budget** (e.g. $10): alerts at **85% actual**, **100% actual** and **100% forecasted** spend.
- Set a budget and alarm before experimenting so mistakes do not become large bills.

> [!tip] Exam
> Billing access for IAM users needs root to enable it. Budgets = alerts on actual/forecasted cost; Bills = per-service breakdown.

## 2. AWS Cost Explorer
(src: 30/09-AWS Cost Explorer)
- **Visualize, understand and manage cost and usage over time**; custom reports, dashboards and graphs.
- Granularity: total across accounts, **monthly, hourly, resource level**.
- Use cases: find expensive services or instance types (and ask whether instances are right-sized and well used); **choose an optimal Savings Plan** (the console shows recommendations and an estimated monthly spend; Savings Plans are an alternative to Reserved Instances, see [[EC2]]); **forecast usage up to 18 months** from past usage, with a confidence range.
- The instructor says this is probably the only billing service you will be asked about on the exam.

[verify] "Cost Explorer forecasts usage up to 18 months."
> [!warning] Correction [note]
> Unconfirmed: AWS's forecasting page that I fetched does not state the maximum horizon. It does say forecasts are based on past usage with an **80% prediction interval** and no forecast is produced with less than about one billing cycle of data. Source: [Forecasting with Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-forecast.html).

> [!tip] Exam
> Cost Explorer = analyze past cost, get Savings Plan recommendations, forecast future cost.

## 3. AWS Cost Anomaly Detection
(src: 30/10-AWS Cost Anomaly Detection)
- Continuously monitors cost and usage with **machine learning** to detect unusual spend; learns your historical patterns; **no thresholds to define**.
- Detects **one-time cost spikes** and **continuous cost increases**.
- Monitors AWS services, **member accounts**, **cost allocation tags** and **cost categories**.
- Sends an anomaly report with **root cause analysis**; notifications as **individual alerts** or **daily/weekly summary** via **SNS** ([[SNS]]).

## 4. Instance Scheduler on AWS
(src: 30/15-Instance Scheduler on AWS)
- Not a service: an **AWS solution deployed with CloudFormation** ([[CloudFormation]]) that automatically **starts and stops resources to cut cost (up to ~70%)**, e.g. stop company EC2 instances outside business hours.
- Supports **EC2 instances, EC2 Auto Scaling groups, RDS instances** (the deployment parameters also mention RDS clusters, Neptune and DocumentDB) ([[RDS & Aurora]], [[ASG]]).
- How it works: **schedules are kept in a DynamoDB table** ([[DynamoDB]]); a **Lambda function** periodically (every 5 minutes by default) reads the schedules and triggers other Lambdas that start/stop the targets. Resources are tagged with the schedule name; actions leave a tag describing when they were performed.
- Supports **cross-account and cross-Region** resources; production ready.

> [!tip] Exam
> Stop/start EC2 and RDS on a schedule to save cost -> Instance Scheduler on AWS (CloudFormation + DynamoDB + Lambda).

## Not included here
- Budget creation clicks (05/01) and CloudFormation stack deployment demo (30/15): hands-on, dropped.
- EC2 purchasing options -> [[EC2]]; other 30/* lectures belong to other notes (CloudFormation, Systems Manager, Outposts & Batch, Messaging & Mobile Services).
- No lecture here lacks a transcript.
