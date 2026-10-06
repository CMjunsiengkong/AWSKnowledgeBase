---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: IAM
version: C (by service)
source_chapters: [04, 25]
related: [AWS CLI & CloudShell, Organizations & Identity Federation, EC2, S3 Security & Encryption, EventBridge]
tags: [aws, saa-c03, iam, policies, mfa, roles, permission-boundaries, conditions]
---

# AWS IAM (Identity and Access Management)

Concept-only note merged from chapters 04 and 25. Lab steps (console clicking) are in [[04 - IAM & AWS CLI]] and [[25 - IAM Advanced]] (Version B). Related: [[AWS CLI & CloudShell]], [[Organizations & Identity Federation]].

## 1. Users, groups, root account
(src: 04/01-IAM Introduction- Users, Groups, Policies, 04/02-IAM Users & Groups Hands On, 04/03-AWS Console Simultaneous Sign-in)
- IAM is a **global service** (no region selector); users/groups created in IAM are available everywhere.
- **Root user** is created with the account: use it only to set up the account, then stop using and never share it.
- **One IAM user = one physical person.** Groups contain **only users, not other groups**; a user may belong to **multiple groups**; a user may belong to no group (allowed, not best practice).
- Users or groups get permissions through **IAM policies** (JSON documents). Users inherit permissions of every group they are in.
- **Least privilege principle**: give only the permissions a user needs.
- Tags are optional metadata on most AWS resources. An **account alias** customizes the sign-in URL (must be globally unique).
- Console multi-session support lets one browser hold sessions in several accounts/roles at once.

> [!tip] Exam
> Groups hold users only. Users can be in many groups. Least privilege is the guiding principle.

## 2. Policies
(src: 04/04-IAM Policies, 04/05-IAM Policies Hands On)
- Ways to attach: policy on a **group** (all members inherit), **inline policy** on a single user, or managed policy attached directly. Effective permissions = union of everything inherited and attached.
- **Policy structure** (JSON):

| Element | Meaning | Required |
|---|---|---|
| `Version` | policy language version, usually `2012-10-17` | yes |
| `Id` | identifier of the policy | optional |
| `Statement` | one or more statements | yes |
| `Sid` | statement ID | optional |
| `Effect` | `Allow` or `Deny` | yes |
| `Principal` | account/user/role the policy applies to (resource-based policies) | - |
| `Action` | list of API calls allowed/denied | yes |
| `Resource` | list of resources the actions apply to | yes |
| `Condition` | when the statement applies | optional |

- `*` means anything: `Action: *` + `Resource: *` = **AdministratorAccess**. Wildcards group calls (e.g. `Get*`, `List*` in IAMReadOnlyAccess).
- Creation via visual editor or JSON editor. Read-only access cannot create groups; you need a broader policy (e.g. IAM full access).

> [!tip] Exam
> Know Effect, Principal, Action, Resource.

## 3. Password policy and MFA
(src: 04/06-IAM MFA Overview, 04/07-IAM MFA Hands On)
- **Password policy** options: minimum length; required character types (upper, lower, number, non-alphanumeric); allow users to change own password; **expiration** (e.g. every 90 days, optionally requiring admin reset); prevent reuse. Helps against brute force.
- **MFA** = password you know + device you own. A stolen password alone does not compromise the account. Protect at least the **root account** and ideally all IAM users.
- Up to **8 MFA devices** can be registered per user in the console (as shown in the lecture).

| MFA device | Notes |
|---|---|
| Virtual MFA device | Google Authenticator (one phone at a time per account) or Authy (multi-token); many root/IAM users on one device |
| U2F security key | physical, e.g. **YubiKey** (third party); one key supports multiple root and IAM users |
| Hardware key fob | e.g. **Gemalto** (third party) |
| Hardware key fob for **AWS GovCloud (US)** | **SurePassID** (third party) |

- Caution: losing the MFA device can lock you out of the account.

> [!tip] Exam
> Know the MFA device options above and that YubiKey/Gemalto/SurePassID are third parties.

## 4. IAM roles
(src: 04/15-IAM Roles for AWS Services, 04/16-IAM Roles Hands On)
- A role is an identity like a user, but intended for **AWS services** (not people) to perform actions on your behalf.
- Common roles: **EC2 instance roles**, Lambda function roles, CloudFormation roles.
- A role has permission policies plus a **trusted entity** (who may assume it, e.g. the EC2 service).
- Use roles instead of access keys for services, see [[EC2]].

## 5. Security tools
(src: 04/17-IAM Security Tools, 04/18-IAM Security Tools Hands On)

| Tool | Level | What it shows |
|---|---|---|
| **IAM Credentials Report** | account | CSV of all users and credential status: creation, password enabled/last used/last changed/next rotation, MFA active, access keys generated/last rotated/last used |
| **IAM Access Advisor** | user | services a user has permission for and **when last accessed**; used to trim unused permissions (least privilege) |

> [!tip] Exam
> Credentials Report = account-level, CSV. Access Advisor = user-level, last-accessed info.

## 6. Best practices and summary
(src: 04/19-IAM Best Practices, 04/20-IAM Summary)
- Do not use root except for account setup; one user per person, never share users or access keys.
- Assign permissions to groups; strong password policy; enforce MFA.
- Use roles for AWS services (including EC2).
- Access keys are for CLI/SDK programmatic access and are secret (see [[AWS CLI & CloudShell]]).
- Audit with the Credentials Report and Access Advisor.

## 7. IAM conditions
(src: 25/04-IAM - Advanced Policies)
Conditions apply to identity policies, resource policies (e.g. S3 bucket policies) and endpoint policies.

| Condition key | Purpose |
|---|---|
| `aws:SourceIp` | restrict client IP the API call comes from (deny unless from given CIDRs, e.g. company network) |
| `aws:RequestedRegion` | restrict the region API calls go to (can be applied organization-wide via SCP) |
| `ec2:ResourceTag` | match tags on the EC2 resource (e.g. `Project = DataAnalytics`) |
| `aws:PrincipalTag` | match tags on the calling user (e.g. department = data) |
| `aws:MultiFactorAuthPresent` | force MFA (e.g. deny stop/terminate when false) |
| `aws:PrincipalOrgID` | in resource policies, allow only principals from accounts in your AWS Organization |

- S3 policy detail: bucket-level actions (`s3:ListBucket`) use the bucket ARN (`arn:aws:s3:::test`); object-level actions (Get/Put/DeleteObject) need `/*` (`arn:aws:s3:::test/*`). Exam-relevant.

## 8. Resource-based policies vs IAM roles (cross-account)
(src: 25/05-IAM - Resource-based Policies vs IAM Roles)
- Cross-account access options: **resource-based policy** (e.g. S3 bucket policy) or **assume a role** in the other account.
- **Assuming a role = giving up your original permissions** and taking the role's. With a resource-based policy the principal **keeps** its own permissions.
- Example: user in Account A scans a DynamoDB table in A and writes to an S3 bucket in B: use a **resource-based policy** on the bucket, no role switch needed.
- Resource-based policies exist on S3, SNS, SQS, Lambda and more.
- **EventBridge** targets: if the target supports resource policies (Lambda, SNS, SQS, S3, API Gateway) EventBridge adds a resource policy; otherwise it uses an **IAM role** (Kinesis Data Streams, EC2 Auto Scaling, Systems Manager Run Command, ECS task). Kinesis Data Streams supports resource policies but EventBridge does not use them yet. Check the rule's configuration. See [[EventBridge]].

[verify] "Kinesis Data Streams ... EventBridge does not use resource-based policy yet" (stated as current at recording time; may have changed). Unconfirmed, not checked against AWS docs.

## 9. Permission boundaries and evaluation logic
(src: 25/06-IAM - Policy Evaluation Logic)
- **Permission boundary**: supported for **users and roles, not groups**; sets the **maximum** permissions an entity can get. Effective permissions = intersection of identity policy and boundary. Example: boundary allows S3/CloudWatch/EC2, identity policy allows `iam:CreateUser` -> **no permissions** (outside boundary). Admin policy + S3-only boundary -> only S3.
- Effective permissions sit at the intersection of **identity-based policy, permission boundary and Organizations SCP**.
- Use cases: delegate user creation to non-admins within a boundary; let developers self-manage permissions without privilege escalation; restrict one specific user instead of an account-wide SCP.
- **Evaluation order**: explicit **Deny** -> Organizations **SCP** (must allow) -> **resource-based policy** -> **identity-based policy** -> **permission boundary** -> **session policy**. Missing allow = implicit deny.
- Worked example: policy `Deny sqs:*` + `Allow sqs:DeleteQueue`: CreateQueue denied; DeleteQueue **denied** (explicit deny wins); `ec2:DescribeInstances` denied (no allow, implicit deny).

> [!tip] Exam
> Explicit Deny always wins. Boundaries apply to users/roles only (not groups). Permission boundary + identity policy must both allow.

## Not included here
- Organizations, SCPs, tag policies, Identity Center, Directory Services, Control Tower -> [[Organizations & Identity Federation]].
- Access keys, CLI, SDK, CloudShell -> [[AWS CLI & CloudShell]].
- Lab narration (users/groups/MFA/roles/credential report) dropped as hands-on.
- Lecture 25/06 transcript is titled "Policy Evaluation Logic" but opens with permission boundaries (covered above).
