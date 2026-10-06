---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Systems Manager
version: C (by service)
source_chapters: [30]
related: [EC2, IAM, EventBridge, CloudTrail & Config, CloudWatch]
tags: [aws, saa-c03, ssm, session-manager, run-command, patch-manager, automation]
---

# AWS Systems Manager (SSM)

Concept-only note. Lab steps are in [[30 - Other Services]] (Version B).

## 1. Managed nodes and the SSM Agent
(src: 30/07-SSM Session Manager, 30/08-SSM Other Services)
- Instances (EC2 or **on-premises servers**) running the **SSM Agent** and registered with SSM are **managed nodes**; they appear in **Fleet Manager**.
- EC2 instances need an **IAM instance profile** that lets them talk to SSM (managed policy **AmazonSSMManagedInstanceCore**).

## 2. Session Manager
(src: 30/07-SSM Session Manager)
- Secure shell on EC2 and on-premises servers **without SSH, bastion host or SSH keys**; **port 22 can stay closed** (security group with zero inbound rules still works). Better security.
- Works via the SSM Agent connected to the Session Manager service; supports Linux, macOS, Windows.
- Session **logs can go to S3 or CloudWatch Logs**; session history is kept. See [[CloudWatch]].

| Way to reach EC2 | Port 22 open | SSH keys | Needs |
|---|---|---|---|
| SSH | yes | yes (permanent) | terminal |
| EC2 Instance Connect | yes | temporary key pushed | browser |
| **Session Manager** | **no** | **no** | SSM Agent + IAM role |

> [!tip] Exam
> "Shell without opening port 22 / no bastion / no SSH keys" = Session Manager.

[verify] The lecture demo uses Amazon Linux 2 for Session Manager.
> Amazon Linux 2 end of support is reported for 2026 (see the Correction on Amazon Linux 2 in [[EC2]], source [Amazon Linux 2 FAQs](https://aws.amazon.com/amazon-linux-2/faqs/)); not independently re-confirmed here. SSM Agent works with other supported AMIs.

## 3. Run Command
(src: 30/08-SSM Other Services)
- Execute a **document** (script) or a single command on **many instances** (selected via resource groups); **no SSH**, same agent mechanism as Session Manager.
- Output to **S3 or CloudWatch Logs**; status updates (in progress, success, failed...) to **SNS**.
- Integrated with **IAM** and **CloudTrail** (who ran what). Can be triggered by **EventBridge**. See [[EventBridge]], [[CloudTrail & Config]].

## 4. Patch Manager
(src: 30/08-SSM Other Services)
- Automates patching of managed instances: OS, application and security updates; EC2 and on-premises; Linux, macOS, Windows.
- Patch **on demand** or on a schedule via a **maintenance window**. Can **scan** and produce a **patch compliance report**.
- Implemented through the `AWS-RunPatchBaseline` run command, invoked from console/SDK/maintenance window.

## 5. Maintenance Windows
(src: 30/08-SSM Other Services)
- Defines a **schedule** for actions on instances. A window contains: **schedule**, **duration**, **targets** (which instances), **tasks** (what to run). Example: every 24 hours run patching, drivers or software installs.

## 6. Automation
(src: 30/08-SSM Other Services)
- Simplifies common maintenance/deployment tasks on EC2 and other AWS resources: restart many instances, create an AMI, create EBS snapshots, snapshot all RDS databases.
- Uses **Automation Runbooks** (SSM documents with predefined actions).
- Triggers: console, SDK, CLI, **EventBridge**, **Maintenance Windows**, **AWS Config** (automatic remediation of non-compliant resources).

| Feature | Purpose |
|---|---|
| Session Manager | shell without SSH |
| Run Command | run scripts/commands on many nodes |
| Patch Manager | patching and compliance report |
| Maintenance Windows | schedule + targets + tasks |
| Automation | runbooks for ops tasks; Config remediation |

> [!tip] Exam
> Instructor: if unsure, remember the general idea of each feature; Config remediation uses SSM Automation.

## Not included here
- Parameter Store (covered in [[Parameter Store & Secrets Manager]]).
- Hands-on narration of 30/07 (launching instance, role, Fleet Manager) -> [[30 - Other Services]].
