---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Messaging & Mobile Services
version: C (by service)
source_chapters: [30]
related: [SNS, SQS, Kinesis & Firehose, CloudWatch, S3, Elastic Beanstalk, "API Gateway, Step Functions & Cognito", DynamoDB, Lambda, CloudFront & Global Accelerator, Redshift]
tags: [aws, saa-c03, ses, pinpoint, appflow, amplify, email, sms, saas-integration]
---

# Messaging & Mobile Services

Concept-only note on Amazon SES, Amazon Pinpoint, Amazon AppFlow and AWS Amplify. These are short overview lectures with no hands-on part; see [[30 - Other Services]] (Version B).

## 1. Amazon SES (Simple Email Service)
(src: 30/05-Amazon SES)
- Fully managed service to **send email securely, globally and at scale**; also **inbound** email (receive replies).
- Applications call the **SES API**, the **SMTP** interface, or use the console.
- Features: **reputation dashboard**, performance insights, **anti-spam feedback**; statistics for deliveries, bounces, feedback-loop results, opens.
- Supports **DKIM** and **SPF**.
- Deployment: **shared IP, dedicated IP, or customer-owned IP**.
- Use cases: **transactional, marketing and bulk emails**.

## 2. Amazon Pinpoint
(src: 30/06-Amazon Pinpoint)
- Scalable **two-way (inbound and outbound) marketing communications** service: **email, SMS, push notifications, voice, in-app messaging**; main use case is SMS.
- **Segment and personalize** messages (groups, segments, templates, delivery schedules, full campaigns); receive replies; scales to **billions of messages per day**.
- Events (text success, delivered, replies...) are published to **SNS, Kinesis Data Firehose and CloudWatch Logs**, so you can build automation on top ([[SNS]], [[Kinesis & Firehose]], [[CloudWatch]]).

| | SNS / SES | Pinpoint |
|---|---|---|
| Audience, content, delivery schedule | **Your application** manages each message's audience, content and schedule | Managed by the service: templates, schedules, targeted segments, campaigns |
| Positioning | Simple messaging building blocks | "Next evolution" of SNS/SES for full marketing communications |

[verify] Amazon Pinpoint is presented as a current service.
> [!warning] Correction [note]
> AWS states that **on October 30, 2026 AWS will end support for Amazon Pinpoint**; after that date the Pinpoint console and resources (endpoints, segments, campaigns, journeys, analytics) are no longer accessible. SMS, voice, mobile push, OTP and phone-number-validate APIs are not impacted and are supported by **AWS End User Messaging**. Source: [Amazon Pinpoint (aws.amazon.com/pinpoint)](https://aws.amazon.com/pinpoint/). Exam questions may still mention Pinpoint.

> [!tip] Exam
> Bulk/transactional email -> SES. Full marketing campaigns with segments, SMS, push -> Pinpoint (per the course). Compare with [[SNS]] pub/sub.

## 3. Amazon AppFlow
(src: 30/13-Amazon AppFlow)
- Fully managed **integration service to transfer data between SaaS applications and AWS**, without writing integration code.
- **Sources**: Salesforce (can appear on the exam), SAP, Zendesk, Slack, ServiceNow...
- **Destinations**: **Amazon S3, Amazon Redshift**, and non-AWS targets such as Snowflake and Salesforce ([[S3]], [[Redshift, EMR, OpenSearch & QuickSight]]).
- Flows can run **on a schedule, in response to events, or on demand**.
- Built-in **transformations** (filtering, validation).
- Data is **encrypted**, over the public internet or **privately via AWS PrivateLink**.

> [!tip] Exam
> Move data from a SaaS app such as Salesforce into S3/Redshift with no custom code -> AppFlow.

## 4. AWS Amplify
(src: 30/14-AWS Amplify)
- A **web and mobile application development tool**: one place to integrate many AWS services. The instructor's mental model: **"Elastic Beanstalk for web and mobile applications"** ([[Elastic Beanstalk]]).
- Workflow:
  1. Create a **backend** with the Amplify CLI, which uses services you know: **S3** (storage), **Cognito** (identity), **AppSync** and **API Gateway** (APIs, GraphQL or REST), **DynamoDB** (data), **Lambda** (functions), SageMaker/Lex (AI/ML) and more; configure auth, storage, API, CI/CD, PubSub, analytics, AI/ML predictions, monitoring in one place ([[API Gateway, Step Functions & Cognito]], [[DynamoDB]], [[Lambda]]).
  2. **Connect your code** from GitHub, CodeCommit, Bitbucket, GitLab, or upload directly.
  3. Add the **Amplify frontend libraries** (web, mobile, many frameworks) to talk to the backend.
  4. **Deploy** with the Amplify Console, using **CloudFront** to serve the app ([[CloudFront & Global Accelerator]]).

> [!tip] Exam
> Amplify = one-stop developer tool for building and hosting web/mobile apps on AWS; think "Elastic Beanstalk for web and mobile".

## Not included here
- No hands-on content existed in these lectures.
- Other 30/* lectures belong elsewhere: CloudFormation (02-04), Systems Manager (07-08), Cost lectures (09-10, 15) -> [[Cost Management & Billing]], Outposts and Batch (11-12).
- No lecture here lacks a transcript.
