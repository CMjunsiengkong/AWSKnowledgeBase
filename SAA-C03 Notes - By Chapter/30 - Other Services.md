---
course: Ultimate AWS Certified Solutions Architect Associate 2026
chapter: 30
chapter_title: Other Services
version: B (by chapter)
services: [CloudFormation, SES, Pinpoint, Systems Manager, Cost Explorer, Cost Anomaly Detection, Outposts, Batch, AppFlow, Amplify, Instance Scheduler]
tags: [aws, saa-c03, cloudformation, systems-manager, session-manager, cost-management, outposts, batch, ses, pinpoint, appflow, amplify]
---

# 30 - Other Services

Related: [[CloudFormation]] · [[Messaging & Mobile Services]] · [[Systems Manager]] · [[Cost Management & Billing]] · [[Outposts & Batch]] · [[IAM]] · [[EC2]] · [[Lambda]]

## Chapter summary
- The instructor says this chapter covers technologies that appear in only one or two **very basic** exam questions: overview level, not deep dives.
- **CloudFormation** = declarative infrastructure as code; repeat architectures across environments, regions and accounts; stack resources are tagged, created/deleted in the right order; a **service role** lets users deploy stacks without direct resource permissions (needs **iam:PassRole**).
- **SES** = email (send and receive); **Pinpoint** = full marketing campaigns over email, SMS, push, voice, in-app; SNS/SES leave audience, content and scheduling to your application.
- **SSM Session Manager** = shell on EC2/on-premises without SSH, port 22, bastion or keys (needs SSM Agent + IAM instance profile). Other SSM features: **Run Command, Patch Manager, Maintenance Windows, Automation**.
- **Cost Explorer** = visualize/forecast cost (forecast up to 18 months), Savings Plan recommendations; **Cost Anomaly Detection** = ML-based, no thresholds to define.
- **Outposts** = AWS-managed server racks in your own data center (hybrid cloud, you own physical security); **Batch** = batch jobs on Docker images on managed EC2/Spot, no time limit (unlike Lambda's 15 min).
- **AppFlow** = SaaS <-> AWS data transfer (Salesforce is the one to remember); **Amplify** = web and mobile app dev tool ("Elastic Beanstalk for web and mobile").
- **Instance Scheduler on AWS** = a CloudFormation-deployed *solution* (not a service) that starts/stops EC2, ASG and RDS on a schedule to save up to ~70%.

---

## 01 - Other Services Section Introduction
(src: 30/01-Other Services Section Introduction)

- Overview-only section; the instructor says everything needed for the exam is already covered and lectures will be added as the exam evolves.

---

## 02 - CloudFormation Intro
(src: 30/02-CloudFormation Intro)

- **CloudFormation** = **declarative** way to describe AWS infrastructure for almost any resource. Example: declare a security group, two EC2 instances using it, an S3 bucket and a load balancer; CloudFormation creates them **in the right order** with the exact configuration.
- Benefits:
  - **Infrastructure as code**: no manual resource creation; changes go through **code review**.
  - **Cost**: every resource in a stack gets a shared **tag**; cost can be estimated from templates; **savings strategy** - e.g. delete a dev environment's stack at 5 PM and recreate it at 8-9 AM.
  - **Productivity**: destroy/recreate infrastructure on the fly; diagrams generated from templates; declarative (no need to work out creation order).
  - **No reinventing the wheel**: reuse existing templates and documentation; supports almost all AWS resources; **custom resources** cover unsupported ones.
- **Infrastructure Composer** (visual designer) shows template resources and their relations (example: a WordPress stack with ALB listener, security groups, SQL database, launch configuration).

> [!tip] Exam
> CloudFormation = infrastructure as code; use it to repeat an architecture in different environments, regions or AWS accounts.

[verify] The lecture calls the visual designer "Infrastructure Composer" in this lecture and "Application Composer" in the next one.
> [!warning] Correction [note]
> Unconfirmed: no source was fetched for the naming. Treat both names as the same CloudFormation visual designer tool; check the current console name.

---

## 03 - CloudFormation - Hands On
(src: 30/03-CloudFormation - Hands On)

- Template used: `0-just-EC2.yaml` from the course code. It has a `Resources` block with one EC2 instance (`MyInstance`): hard-coded AZ `us-east-1a`, an AMI ID, type **t2.micro**. Because AZ and **AMI IDs are region-scoped**, the hands-on must be done in **us-east-1**.
- Stack creation options for the template: existing template, sample template, or build in Application Composer.
- Wizard pages: stack name (`demo CloudFormation`), **parameters** (none yet), **tags** (e.g. Name = `CFDemo`), permissions (none), review, submit. Events show resource creation; the instance appears in EC2 with CloudFormation tags (stack name, logical ID, stack ID) plus your own tag.
- **Update** with `1-ec2-with-sg-eip`: adds a **parameter** (security group description), two security groups (an SSH one on port 22; a server one with port 80 from everyone and port 22 from a specific IP) and an **Elastic IP** attached to the instance.
- **Change set** preview shows Add for the EIP and security groups and Modify for the instance with **Replacement = True**: the old instance is deleted and a new one created (matters if the instance holds data). CloudFormation creates the security groups first, then the new instance, attaches the EIP, then cleans up the old instance.
- **Do not change stack resources manually.** Update the template, or **Delete** the stack: CloudFormation deletes all resources in the correct order.

> [!tip] Exam
> Update preview = **change set**; some changes cause **replacement** of the resource. Delete a stack to remove all its resources.

### Hands-on steps
1. Switch region to us-east-1 -> CloudFormation -> Create stack -> Upload a template file (`0-just-EC2.yaml`).
2. (Optional) View in Application Composer to see the visual canvas.
3. Stack name `demo CloudFormation`; no parameters; add tag `Name = CFDemo`; leave permissions and options default; Submit.
4. Check Events, Resources tab and the EC2 console (instance type, AMI, tags).
5. Update stack -> Replace existing template -> upload `1-ec2-with-sg-eip`; enter a description for the security group parameter.
6. Review the change set (note Replacement = True) -> Submit; watch events and the new instance/EIP.
7. View the template in Application Composer again to see the instance linked to the EIP and two security groups.
8. Delete the stack to clean up everything.

---

## 04 - CloudFormation - Service Role
(src: 30/04-CloudFormation - Service Role)

- A **CloudFormation service role** is an IAM role dedicated to CloudFormation that lets it **create, update and delete stack resources on your behalf**.
- Use case: users may manage stacks but lack permissions on the underlying resources. Example: the user has CloudFormation actions + **iam:PassRole**; the service role has `s3:*` for buckets; CloudFormation creates the bucket using the role.
- Achieves **least privilege**: users only need permission to invoke/pass the service role, not every resource permission.
- The user **must have `iam:PassRole`** to give a role to a service.
- If no role is specified at stack creation, CloudFormation uses the user's own permissions; if one is specified, it is used for **all stack operations**.

> [!tip] Exam
> CloudFormation service role + `iam:PassRole` = least privilege for stack deployment.

### Hands-on steps
1. IAM -> Roles -> Create role -> AWS service **CloudFormation** -> attach `AmazonS3FullAccess` -> name e.g. `DemoRole for CFN with S3 capabilities`.
2. CloudFormation -> Create stack (any existing template) -> in Permissions choose that IAM role.
3. The stack fails if it creates an EC2 instance, because the role only has S3 permissions (demonstrating that the role, not your own permissions, is used).

Related: [[IAM]]

---

## 05 - Amazon SES
(src: 30/05-Amazon SES)

- **SES (Simple Email Service)**: fully managed service to send email **securely, globally, at scale** via the SES API or an **SMTP** server; supports **outbound and inbound** email (receive replies).
- Features: reputation dashboard, performance insights, anti-spam feedback; statistics for deliveries, bounces, feedback loop results, opens; supports **DKIM** and **SPF**.
- Deployment: **shared IP, dedicated IP, or customer-owned IP**. Access: console, AWS APIs, SMTP.
- Use cases: **transactional, marketing and bulk** email.

---

## 06 - Amazon Pinpoint
(src: 30/06-Amazon Pinpoint)

- **Pinpoint** = scalable **two-way (inbound and outbound)** marketing communication service: **email, SMS, push, voice, in-app** messaging; **SMS** is a main use case.
- Segment and personalize messages (groups/segments); receive replies; scales to **billions of messages per day**.
- Use cases: bulk marketing campaigns, transactional SMS. Events (sent, delivered, replies) go to **SNS, Kinesis Data Firehose and CloudWatch Logs** for automation.
- **Pinpoint vs SNS/SES**: with SNS/SES your application manages each message's audience, content and delivery schedule; with Pinpoint you create **message templates, delivery schedules, targeted segments and full campaigns**. The instructor calls it the next evolution of SNS/SES for full marketing communications.

[verify] The lecture presents Pinpoint as a current service and the evolution of SNS/SES.
> [!warning] Correction [note]
> AWS documents that Amazon Pinpoint reaches **end of support on 30 October 2026**, after which the console and resources (endpoints, segments, campaigns, journeys, analytics) are no longer accessible; SMS, voice, push and OTP APIs continue under AWS End User Messaging, and email moves to SES. Source: [Amazon Pinpoint end of support](https://docs.aws.amazon.com/pinpoint/latest/userguide/migrate.html) (found via search result; details beyond the date come from search summaries).

---

## 07 - SSM Session Manager
(src: 30/07-SSM Session Manager)

- **Session Manager** starts a secure shell on EC2 instances and **on-premises servers without SSH access, bastion host or SSH keys**; **port 22 can stay closed** (better security).
- Mechanism: the instance runs the **SSM Agent**, which connects to the Session Manager service; users reach the instance through the service. Supports **Linux, macOS and Windows**. Session logs can be sent to **S3 or CloudWatch Logs**; session history is saved.
- Requirement: the instance needs an **IAM instance profile/role** allowing it to talk to Systems Manager (managed policy **AmazonSSMManagedInstanceCore**). Instances registered with SSM appear in **Fleet Manager** as **managed nodes** (SSM Agent online, platform, agent version).
- Demo instance: security group with **zero inbound rules**, no key pair; the shell still worked and `ping google.com` succeeded.
- **Three ways to access an EC2 instance**:

| Method | Needs port 22 open | Needs SSH keys |
|---|---|---|
| SSH with terminal | yes | yes (you manage) |
| EC2 Instance Connect | yes | no (temporary key uploaded) |
| **SSM Session Manager** | **no** | **no** (needs SSM Agent + IAM role) |

> [!tip] Exam
> Need shell access with no inbound ports, no keys, no bastion -> **SSM Session Manager** (instance role with SSM permissions).

[verify] The demo uses the Amazon Linux 2 AMI.
> [!warning] Correction [note]
> AWS states Amazon Linux 2 reached end of support on 30 June 2026 and recommends Amazon Linux 2023. Source: [Amazon Linux 2 FAQs](https://aws.amazon.com/amazon-linux-2/faqs/).

### Hands-on steps
1. EC2 -> Launch instance: Amazon Linux 2 AMI, t2.micro, **no key pair**, security group with SSH disabled (no inbound rules).
2. Advanced details -> IAM instance profile -> Create new IAM role: service EC2, policy `AmazonSSMManagedInstanceCore`, name `Demo EC2 Role for SSM`; refresh and select it; launch.
3. Systems Manager -> **Fleet Manager** -> wait until the instance appears as a managed node with SSM Agent online.
4. Systems Manager -> **Session Manager** -> Start session -> select instance -> Start.
5. In the shell run `ping google.com` and check the host name/private IP matches the instance's private IP.
6. Terminate the session (history is kept in Session Manager), then terminate the instance.

---

## 08 - SSM Other Services
(src: 30/08-SSM Other Services)

The instructor says each may appear in about one question; remember the general idea.

- **Run Command**: execute a document (script or single command) on **multiple instances** (selected by resource groups); EC2 or on-premises servers registered with SSM. **No SSH**: same agent mechanism as Session Manager. Output to **S3 or CloudWatch Logs**; status (in progress, success, failed) to **SNS**; integrated with **IAM** and **CloudTrail** (who ran what); can be triggered by **EventBridge**.
- **Patch Manager**: automates patching of managed instances (OS, application and security updates) on EC2 and on-premises, Linux/macOS/Windows. Patch **on demand** or on a schedule via a **Maintenance Window**; can **scan** and produce a **patch compliance report**. Uses the **AWS-RunPatchBaseline** run command (invoked from console, SDK or Maintenance Window).
- **Maintenance Windows**: define a schedule for actions on instances: **schedule** (when), **duration**, **target instances**, **tasks to run** (e.g. OS patching, driver updates, software installs); e.g. triggered every 24 hours.
- **Automation**: simplifies common maintenance/deployment tasks on EC2 and other AWS resources (restart many instances, create an AMI, create EBS snapshots, snapshot all RDS databases). Uses **Automation Runbooks** (SSM documents with predefined actions). Triggered from console, SDK, CLI, **EventBridge**, **Maintenance Windows**, or **AWS Config** (automatic remediation of non-compliant resources).

| SSM feature | Purpose |
|---|---|
| Session Manager | shell without SSH |
| Run Command | run a script/command on many instances |
| Patch Manager | OS/app patching + compliance report |
| Maintenance Windows | schedule (what/when/how long/targets) |
| Automation | runbooks for maintenance tasks, Config remediation |

> [!tip] Exam
> Config non-compliance -> remediation via SSM Automation; patching -> Patch Manager (+ Maintenance Window); run commands at scale without SSH -> Run Command.

---

## 09 - AWS Cost Explorer
(src: 30/09-AWS Cost Explorer)

- **Cost Explorer** visualizes, understands and manages AWS **cost and usage over time**; custom reports, dashboards and charts.
- Granularity: high level (total across all accounts) down to **monthly, hourly or resource level**. Use it to spot costly instance types and ask whether instances are correctly used and right-sized.
- **Savings Plan recommendations**: suggests a plan based on your usage with estimated monthly spend (Savings Plans are an alternative to Reserved Instances).
- **Forecast** usage **up to 18 months** from previous usage, with a confidence range.
- The instructor says this is probably the **only billing service** asked in the exam.

> [!tip] Exam
> Visualize/analyze cost over time, forecast, and get Savings Plan recommendations -> Cost Explorer.

[verify] "Forecast up to 18 months."
> [!warning] Correction [note]
> Consistent with AWS: Cost Explorer now supports 18-month forecasting (announced Nov 2025, up from 12 months). Source: [AWS Cost Explorer now provides 18-month forecasting](https://aws.amazon.com/about-aws/whats-new/2025/11/cost-explorer-18-month-forecasting-ai-powered-forecasts/).

---

## 10 - AWS Cost Anomaly Detection
(src: 30/10-AWS Cost Anomaly Detection)

- Continuously monitors cost and usage with **machine learning** to detect unusual spend; learns your **historical patterns**; detects **one-time spikes and continuous increases**.
- **No thresholds to define.**
- Monitors AWS services, **member accounts, cost allocation tags and cost categories**.
- Produces an anomaly report with **root cause analysis**. Notifications: individual alerts or **daily/weekly summary** using **SNS**.

> [!tip] Exam
> ML-based unusual-spend detection without setting thresholds -> Cost Anomaly Detection.

---

## 11 - AWS Outposts
(src: 30/11-AWS Outposts)

- **Hybrid cloud** = on-premises plus cloud infrastructure; normally two skill sets and two sets of APIs. **Outposts** = **server racks** offering the **same AWS infrastructure, services, APIs and tools** on-premises. AWS sets up and manages the racks in your data center; **fully managed**.
- **Customer responsibility**: **physical security** of the rack, since it sits in your data center.
- Benefits: **low-latency** access to on-premises systems, **local data processing**, **data residency** (data need not leave), easy step-by-step migration (on-premises -> Outposts -> cloud).
- Services available: **EC2, EBS, S3, EKS, ECS, RDS, EMR**.

> [!tip] Exam
> Native AWS services/APIs running in your own data center, managed by AWS, for low latency or data residency -> Outposts.

---

## 12 - AWS Batch
(src: 30/12-AWS Batch)

- **AWS Batch** = fully managed batch processing at any scale (hundreds of thousands of jobs). A **batch job has a start and an end** (unlike continuous/streaming jobs).
- Batch **dynamically launches EC2 instances or Spot Instances**, provisioning the right compute/memory for the job queue; you submit or schedule jobs into a **batch queue**. Cost-optimized (Spot) and no infrastructure focus.
- Jobs are defined as **Docker images** and run on **ECS, EKS or Fargate**.
- Example architecture: image uploaded to S3 triggers a Batch job; Batch runs an ECS cluster of EC2/Spot instances; containers process the image and write the result to another S3 bucket.

| | Lambda | Batch |
|---|---|---|
| Time limit | **15 minutes** | **none** (runs on EC2) |
| Runtime | limited set of languages | **any**, packaged as Docker image |
| Disk | limited temporary space | EC2 storage (**EBS** or **instance store**), much more |
| Model | **serverless** | **managed** service on real EC2 instances (AWS manages scaling) |

> [!tip] Exam
> Long-running or large-disk batch jobs in containers on Spot/EC2 -> Batch; short event-driven serverless code -> Lambda.

---

## 13 - Amazon AppFlow
(src: 30/13-Amazon AppFlow)

- **AppFlow** = fully managed integration service to transfer data between **SaaS applications and AWS** without writing integrations.
- Sources: **Salesforce** (the one to remember for the exam), SAP, Zendesk, Slack, ServiceNow.
- Destinations: **S3, Redshift**, non-AWS targets such as **Snowflake** and Salesforce.
- Run **on a schedule, on specific events, or on demand**. Transformations: filtering, validation. Data encrypted over the public internet, or private via **PrivateLink**.

---

## 14 - AWS Amplify
(src: 30/14-AWS Amplify)

- **Amplify** = web and mobile application development tool; one place to integrate AWS services. The instructor says to think of it as **"Elastic Beanstalk for web and mobile applications"**.
- Flow:
  1. Create a **backend** with the Amplify CLI, using S3 (storage), Cognito (identity), AppSync / API Gateway (APIs, REST or GraphQL), SageMaker (ML), Lex, Lambda, DynamoDB, etc.; also auth, CI/CD, PubSub, analytics, AI/ML predictions, monitoring.
  2. Connect code from GitHub, CodeCommit, Bitbucket, GitLab or direct upload.
  3. Add **Amplify frontend libraries** (web, mobile, many frameworks).
  4. Deploy with the **Amplify Console**, using **CloudFront** for delivery.

[verify] Backend creation is described as using the "Amplify CLI".
> [!warning] Correction [note]
> Search results indicate Amplify Gen 2 replaces the CLI-and-Studio workflow with a TypeScript, code-first backend (built on CDK), while Gen 1 (CLI based) remains supported; AWS recommends Gen 2 for new projects. Source: [FAQ - AWS Amplify Gen 2 Documentation](https://docs.amplify.aws/react/how-amplify-works/faq/) (from search result summaries; page not opened).

---

## 15 - Instance Scheduler on AWS
(src: 30/15-Instance Scheduler on AWS)

- **Instance Scheduler on AWS** is **not a service but an AWS Solution** deployed through **CloudFormation**. It automatically **starts and stops** resources to **reduce costs, maybe up to 70%** (e.g. stop company EC2 instances outside business hours).
- Supports **EC2 instances, EC2 Auto Scaling groups, RDS instances** (also RDS clusters, Neptune, DocumentDB per the stack parameters); **cross-account and cross-Region**; production ready.
- Architecture: schedules live in a **DynamoDB table** (config table); a **Lambda** function reads the schedules and triggers other Lambdas that start/stop the resources.
- Stack parameters shown: tag key (`Schedule`), frequency (checks **every 5 minutes**), default time zone, enable/disable scheduling, services to schedule, tagging of actions (a tag is added describing when/where Instance Scheduler acted), RDS snapshot on stop.
- Schedule items define name, begin time, end time, description (e.g. office hours, working days, first Monday of each quarter).
- The instructor says the exam may ask about the idea: stop and start resources to save cost.

> [!tip] Exam
> Stop/start EC2 and RDS on a schedule to cut cost -> Instance Scheduler on AWS (CloudFormation + DynamoDB + Lambda).

### Hands-on steps
1. Search "Instance Scheduler on AWS" -> open the solution page (benefits, implementation guide, source code, CloudFormation template, architecture steps 1-7).
2. Click **Launch solution** -> opens CloudFormation with the template URL pre-filled.
3. Stack name `Instance Scheduler`; review the parameters (tag key, 5-minute frequency, time zone, enable scheduling, services, tagging); accept the IAM acknowledgement; create. (Demo only.)
4. Inspect Resources: DynamoDB config table (schedule items) and the Lambda functions.
5. Delete the stack.

---

## Not covered
- All 15 lectures in this chapter have transcripts; none are missing.
