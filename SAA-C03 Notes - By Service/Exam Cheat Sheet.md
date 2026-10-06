---
course: Ultimate AWS Certified Solutions Architect Associate 2026
version: C (by service)
title: Exam Cheat Sheet
tags: [aws, saa-c03]
---
# Exam Cheat Sheet - By Service

Automatically collected from the `Exam` callouts of each service note (nothing added). Open the service note for tables, limits and corrections.

## Foundations & Identity

### [[AWS Global Infrastructure]]
- Region choice = compliance, latency, available services, price.
- Global: IAM, Route 53, CloudFront, WAF. Most others (EC2, Lambda, Beanstalk...) are regional.

### [[IAM]]
- Groups hold users only. Users can be in many groups. Least privilege is the guiding principle.
- Know Effect, Principal, Action, Resource.
- Know the MFA device options above and that YubiKey/Gemalto/SurePassID are third parties.
- Credentials Report = account-level, CSV. Access Advisor = user-level, last-accessed info.
- Explicit Deny always wins. Boundaries apply to users/roles only (not groups). Permission boundary + identity policy must both allow.

### [[AWS CLI & CloudShell]]
- Access keys power CLI and SDK; keep them secret. For AWS services use IAM roles instead (see [[IAM]]).
- CloudShell = free browser terminal using your console credentials, region-limited, with persistent home storage.

### [[Organizations & Identity Federation]]
- SCP = guardrail, never applies to the management account, needs explicit allow down the tree, deny always wins.
- "Keep tags consistent across accounts" -> tag policies in Organizations.
- One login into multiple AWS accounts -> IAM Identity Center.
- Control Tower = Organizations + guardrails. Preventive = SCP, detective = Config.

## Compute

### [[EC2]]
- EC2 User Data runs only at first launch, with root privileges.
- Timeout = SG. Refused = app. 22 = SSH/Linux, 3389 = RDP/Windows. SG can reference SGs.
- Give EC2 access to AWS services with an IAM role, not access keys. See [[IAM]].
- ENI = AZ-bound, movable network card for failover; keeps its private IP and SGs.
- Low latency/high throughput -> Cluster. Max isolation of few critical instances -> Spread (7 per AZ). Big distributed data stores -> Partition.
- Hibernate = RAM state saved to encrypted EBS root volume; instance resumes with processes intact.
- "Very high performance hardware-attached volume" -> EC2 Instance Store. Not durable.
- Pick by workload: Spot for resilient batch; Dedicated Host for licenses/compliance; Capacity Reservation = capacity not discount; Savings Plan = $/hour commitment.
- Spot Fleet strategies: lowestPrice, diversified, capacityOptimized, priceCapacityOptimized. Cancel the request, then terminate instances.

### [[ELB]]
- Scale up/down = vertical. Scale out/in = horizontal. HA = multiple AZs. Questions can trick you on these terms.
- SG chaining: instance SG references the LB SG. See [[EC2]] for security group basics.
- Path/host/query/header routing, microservices, containers, Lambda targets -> ALB. Real client IP -> `X-Forwarded-For`.
- Extreme performance, TCP/UDP, or static / Elastic IPs -> NLB. NLB in front of ALB is a valid combination.
- Third-party security appliances / traffic inspection, **GENEVE port 6081**, layer 3 -> Gateway Load Balancer.
- ALB: cross-zone on by default, free. NLB / GWLB: off by default, paid if enabled.
- Multiple SSL certificates on one load balancer -> ALB or NLB (SNI). CLB = one certificate. Certificates managed in ACM.

### [[ASG]]
- ASG + ELB: ELB health checks can make the ASG replace instances. ASG is free; launch template defines the instances. Multi-AZ ASG = high availability.
- Know the five policy types (target tracking, simple/step, scheduled, predictive), the default 300 s cooldown, and CPU / RequestCountPerTarget / network / custom metrics.

### [[Elastic Beanstalk]]
- Fast startup = Golden AMI + User Data for dynamic bits; RDS/EBS from snapshots.
- Beanstalk = deploy code without managing infra (PaaS, free service, pay for resources). Know: application / version / environment, web vs worker (SQS), single instance (dev) vs HA with ELB (prod). Beanstalk is code-centric; CloudFormation is for arbitrary infrastructure.

### [[Lambda]]
- The exam tests serverless knowledge heavily. Lecture 19/01 (section intro) has no transcript.
- Needs like "30 GB RAM", "30 minutes of runtime" or "a 3 GB file in the package" mean Lambda is the wrong choice. Large files: use `/tmp` at runtime, not the package.
- Cold start fix choices: **SnapStart** (snapshot at publish) or **provisioned concurrency** (pre-warmed instances, extra cost).
- Simple, ultra-fast, high-scale viewer-only tweaks = CloudFront Functions. Origin-side triggers, longer runtime, body access or SDK calls = Lambda@Edge.
- Lambda + RDS with connection errors under load = **RDS Proxy**, and Lambda in the VPC. Lambda reaches DynamoDB without being in a VPC.

### [[Containers (ECS, ECR, EKS)]]
- The exam favors **Fargate**: serverless and far easier to manage than the EC2 launch type.
- Know the difference: EC2 instance profile (agent, EC2 type only) vs ECS task role (per task, both types).
- EC2 launch type: prefer Capacity Provider over plain ASG scaling. Fargate is the easiest.
- "Store Docker images" = ECR.
- Kubernetes / pods / "cloud-agnostic containers" = EKS. EFS is the only StorageClass for EKS on Fargate.

### [[Outposts & Batch]]
- "Same AWS APIs/services on premises, AWS-managed hardware, hybrid, low latency, local processing, data residency" = Outposts.
- Batch vs Lambda: no time limit, any Docker runtime, larger disk, EC2/Spot based. See [[Lambda]].

## Storage & Content Delivery

### [[S3]]
- Bucket = regional; key = prefix + object name; max object 50 TB; multi-part mandatory above 5 GB.
- EC2 -> S3: IAM role. Cross-account: bucket policy. Public bucket needs BOTH Block Public Access off AND a public bucket policy.
- Delete = delete marker (recoverable). Delete of a version ID = permanent. Suspend = keeps old versions.
- Versioning required; only new objects (Batch Replication for old); delete markers optional; permanent deletes not replicated; no chaining.
- Rapid access but infrequent -> Standard-IA; re-creatable -> One Zone-IA; archive in ms -> Glacier Instant; unknown pattern -> Intelligent-Tiering; cheapest archive -> Deep Archive (180 days).
- Single-digit ms, one AZ, directory bucket, co-locate with compute.
- Lifecycle = transition + expiration, by prefix/tag. Analytics only advises Standard vs Standard-IA.
- Native targets: SNS, SQS, Lambda. Need richer filtering or more targets -> EventBridge. Permissions are resource policies on the target.
- Upload speed over distance -> Transfer Acceleration; big files -> multi-part; fast partial reads/parallel downloads -> byte-range fetches. Also see [[CloudFront & Global Accelerator]].
- "Encrypt all existing unencrypted objects" -> S3 Inventory + Athena + S3 Batch Operations.
- Organization-wide visibility, unencrypted-object counts, buckets lacking best practices -> Storage Lens. Free vs paid: 14 days vs 15 months.

### [[S3 Security & Encryption]]
- Know which option fits which wording: "AWS-managed, default" = SSE-S3; "audit key usage / control keys" = SSE-KMS; "multi-layer encryption" = DSSE-KMS; "keys managed outside AWS but S3 does the encryption" = SSE-C; "client does everything" = client-side.
- MFA Delete = extra protection against permanent deletion of object versions; needs versioning; root-only to turn on.
- Whole-vault immutability = Glacier Vault Lock. Per-object WORM = S3 Object Lock (needs versioning). Compliance = nobody can delete, even root. Governance = privileged IAM users can. Legal hold = no expiry, separate from retention.

### [[EBS]]
- Root volume is deleted on termination by default; extra volumes are kept. Disable the flag to preserve the root volume. EBS = one AZ, one instance (except io1/io2 Multi-Attach).
- Boot volume = gp2/gp3/io1/io2. Database needing >32,000 IOPS = io1/io2 on Nitro. Cheapest cold storage = sc1; streaming/throughput workloads = st1. gp3 sets IOPS independently of size; gp2 does not.
- Cross-AZ/Region move = snapshot. Accidental deletion protection = Recycle Bin. No first-use latency = Fast Snapshot Restore (costly). Cheaper long-term snapshots = Archive tier (24-72 h restore).
- Unencrypted volume -> snapshot -> encrypted copy -> new volume. Encryption has almost no latency cost and uses KMS (AES-256).

### [[EFS]]
- Shared file system across AZs for Linux = EFS (NFS, POSIX, SG port 2049). Cost saving = EFS-IA / Archive with lifecycle policies; One Zone for dev. Unpredictable throughput = Elastic; Windows = use FSx for Windows ([[FSx]]) instead.

### [[FSx]]
- Windows/SMB/AD -> FSx for Windows. HPC/ML/S3-integrated -> Lustre (scratch = temporary, persistent = long-term). NAS/ONTAP migration, NFS+SMB+iSCSI -> NetApp ONTAP. ZFS migration -> OpenZFS. Only the four types need to be known.

### [[Storage Gateway]]
- On-prem NFS/SMB access to S3 = S3 File Gateway. On-prem block volumes backed up to cloud = Volume Gateway (cached vs stored). Physical tape replacement = Tape Gateway. S3 File Gateway cannot use Glacier directly; use a lifecycle policy. Need an on-prem VM/hardware to run the gateway.

### [[Snow Family]]
- Slow/limited network, large (TB-PB) one-off data -> Snowball. Edge locations without connectivity -> Snowball Edge Compute Optimized. Snowball -> Glacier = S3 first + lifecycle policy.

### [[Data Transfer]]
- FTP/FTPS/SFTP to S3/EFS = Transfer Family. Scheduled sync preserving metadata/permissions, on-prem or AWS-to-AWS = DataSync (agent needed for NFS/SMB). DataSync is not continuous.
- Huge one-off dataset + weak bandwidth = Snowball. Ongoing sync = DataSync / DMS / VPN / Direct Connect. Direct Connect takes about a month to provision.

### [[Storage Options Compared]]
- Pick by access pattern: object = S3, block = EBS / instance store, shared Linux file = EFS, specialized file = FSx, hybrid = Storage Gateway, protocol transfer = Transfer Family, scheduled sync = DataSync, offline bulk = Snow family.

### [[CloudFront & Global Accelerator]]
- CloudFront in front of a **private** ALB/NLB/EC2 -> **VPC origin**. Private S3 content via CloudFront -> **OAC + bucket policy**.
- Need new origin content visible immediately despite TTL -> **CloudFront invalidation** (partial or full).
- Caching static/dynamic web content = CloudFront. Static anycast IPs, TCP/UDP, non-HTTP, fast regional failover = Global Accelerator. Both integrate with Shield. "CDN" always means CloudFront.

## Databases & Analytics

### [[RDS & Aurora]]
- "Avoid manually scaling database storage" -> RDS Storage Auto Scaling (needs a max storage threshold).
- Scale reads / run analytics -> Read Replica. Disaster recovery / HA -> Multi-AZ. Single-AZ to Multi-AZ = no downtime (snapshot -> restore -> sync).
- 6 copies / 3 AZs, 4 of 6 write, 3 of 6 read. Writer endpoint + reader endpoint. Shared auto-expanding storage. Failover < 30 s.
- "Replicate across regions in under 1 second" / "DR with RTO under 1 minute" -> **Aurora Global Database**.
- Automated backups = 1-35 days + PITR; manual snapshot = retained indefinitely. Percona XtraBackup + S3 -> Aurora MySQL.
- Many connections / Lambda to RDS -> RDS Proxy. Enforce IAM auth for the DB -> RDS Proxy. Failover time -66%.
- Event notification = DB lifecycle events, never data changes. For data events, invoke Lambda from the database.

### [[ElastiCache]]
- ElastiCache **requires application code changes** (query the cache before/after the database). If a question asks for caching with **no code change**, ElastiCache is not the answer.
- Lazy loading = load on miss, may be stale. Write-through = update cache on every DB write, never stale. Sessions = TTL.
- "Real-time gaming leaderboard" -> **ElastiCache for Redis (sorted sets)**.

### [[DynamoDB]]
- "Schema must evolve rapidly / flexible schema" -> DynamoDB over RDS/Aurora. Use cases: serverless apps with small documents (hundreds of KB max), and a distributed serverless cache / key-value store (can replace ElastiCache for session data with TTL).
- Traffic going from 1,000 to 1 million transactions in under a minute -> provisioned mode does not scale fast enough -> **On-Demand**.
- Global tables = active-active multi-region. (AWS docs describe them as multi-active, multi-Region with any replica serving reads and writes: [Global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html).)

### [[Other Databases & Choosing a DB]]
- MongoDB -> DocumentDB. "NoSQL" -> DocumentDB or DynamoDB.
- Graph database -> Neptune.
- Apache Cassandra -> Keyspaces.

### [[Athena, Glue & Lake Formation]]
- Analyze data in S3 with a serverless SQL engine -> **Athena**.
- "Centralized data-lake permissions with row/column-level security across Athena, QuickSight, etc." -> **Lake Formation**.

### [[Redshift, EMR, OpenSearch & QuickSight]]
- Analytics/warehouse/OLAP/BI on petabytes -> Redshift. Query S3 with Redshift's power without loading -> Spectrum. Cross-region DR -> automated snapshot copy.
- Hadoop / Spark big data clusters -> EMR.

### [[Streaming Analytics - Flink & MSK]]
- Flink = stream processing only. Reads Kinesis Data Streams / MSK, not Firehose. Not needed beyond this level.
- Apache Kafka -> MSK. Know the Kinesis vs MSK differences above.

## Application Integration & Serverless

### [[SQS]]
- Visibility timeout default 30 s (max 12 h). Message processed twice -> timeout too short; use `ChangeMessageVisibility`.
- "Decoupling", "sudden spike", "timeouts", "must not lose requests" -> SQS (with ASG scaling on queue length). Dequeue/delete only after success.

### [[SNS]]
- Need fan-out + ordering + deduplication -> **SNS FIFO topic -> SQS FIFO queues**.
- Different subscribers need different subsets of the same topic -> SNS message filtering (JSON policy), not multiple topics.

### [[Kinesis & Firehose]]
- Real-time, replay, custom consumers -> Kinesis Data Streams. Near real-time load into S3/Redshift/OpenSearch with no code -> Data Firehose.

### [[Amazon MQ & Messaging Comparison]]
- Migrating an on-premises app that uses MQTT/AMQP/STOMP/Openwire/WSS without code changes -> Amazon MQ. Building new cloud-native -> SQS/SNS.
- Queue and decouple, delete after processing -> SQS. Notify many receivers -> SNS (fan-out with SQS). Real-time big data with replay -> Kinesis. Open-protocol migration -> Amazon MQ.

### [[API Gateway, Step Functions & Cognito]]
- API Gateway + Lambda = serverless REST API. Need auth, throttling, API keys, caching or versioning in front of HTTP/AWS services = API Gateway. Edge-optimized cert in us-east-1; regional cert in the API's Region.
- Orchestrate multi-step serverless workflows with retries and approvals = **Step Functions**.
- **User Pools = who are you (authenticate, integrates with API Gateway / ALB). Identity Pools = what AWS resources can you reach (temporary credentials, fine-grained IAM).** Never ship AWS access keys in a mobile app; use Cognito. See also [[IAM]] and [[Organizations & Identity Federation]].

### [[EventBridge]]
- EventBridge = formerly CloudWatch Events. Schedule/cron + event patterns. React to **any API call** = CloudTrail + EventBridge. Cross-account = resource-based policy on the bus. Replay = archive.

### [[Serverless Solution Architectures]]
- Wrong answer: store AWS user credentials in the mobile app. Right answer: **Cognito temporary credentials**. Read-heavy DynamoDB = **DAX**; static API responses = **API Gateway cache**.
- No Cognito needed for a fully public REST API. Static + global = S3 + CloudFront + OAC. Reacting to table changes = DynamoDB Streams + Lambda.
- Microservices are a **design**, not a service; both sync (API Gateway, ELB) and async (SQS, SNS, Kinesis) integration patterns apply. See [[SQS]], [[SNS]], [[Kinesis & Firehose]], [[Containers (ECS, ECR, EKS)]].
- Mostly static content served at scale from an existing app = **CloudFront caching** as the simplest, cheapest improvement. See [[ELB]], [[ASG]], [[EFS]].

### [[Messaging & Mobile Services]]
- Bulk/transactional email -> SES. Full marketing campaigns with segments, SMS, push -> Pinpoint (per the course). Compare with [[SNS]] pub/sub.
- Move data from a SaaS app such as Salesforce into S3/Redshift with no custom code -> AppFlow.
- Amplify = one-stop developer tool for building and hosting web/mobile apps on AWS; think "Elastic Beanstalk for web and mobile".

## Networking

### [[Route 53]]
- TTL mandatory except for Alias records. High TTL = cheaper but stale; low TTL = costlier but fast changes.
- Root domain pointing to an ELB/CloudFront -> **Alias record (A/AAAA)**, never CNAME. Alias = free, native health check, no TTL. EC2 DNS name cannot be an Alias target.
- Shift traffic between regions -> **Geoproximity (bias)**. Per-country content/compliance -> **Geolocation** (+ Default). Lowest latency -> **Latency**. Active-passive -> **Failover** (primary needs health check). Known client CIDRs -> **IP-based**. Up to 8 healthy answers -> **Multi-value**. Percent split / canary -> **Weighted**.
- Private resource health check = CloudWatch alarm. Failover primary record needs a health check. Allow health checker IPs in the firewall.
- Third-party domain + Route 53 DNS = public hosted zone + update NS records at the registrar.
- Two-way DNS between AWS and a data center = Resolver **inbound + outbound** endpoints over VPN/Direct Connect. See [[VPC Connectivity]].

### [[VPC]]
- Know how to read a CIDR: /24 = 256, /16 = 65,536, /32 = 1, /0 = everything. The three private ranges above.
- VPC CIDR: /28 to /16, up to 5 CIDRs, 5 VPCs per region (soft). Plan non-overlapping CIDRs.
- Need 29 IPs for EC2? /27 = 32 - 5 = 27 (too few). Choose **/26** (64 - 5 = 59).
- Public subnet = IGW attached to the VPC + route `0.0.0.0/0` -> IGW. Both are required.
- Bastion in public subnet, locked-down SSH source; private SG references bastion SG on port 22.
- Prefer NAT gateway (managed, scalable). One per AZ for HA. Needs an IGW. NAT instance needs source/destination check disabled.
- SG = instance, allow-only, stateful, all rules. NACL = subnet, allow+deny, stateless, first match by number; default NACL allows all; custom NACL denies all; remember ephemeral ports. Block one IP -> NACL.

### [[VPC Connectivity]]
- Peering = non-transitive + no overlapping CIDRs + route tables on both sides.
- Private access to S3/DynamoDB from a VPC -> Gateway Endpoint (free). Any other service, or on-premises access -> Interface Endpoint (PrivateLink).
- VGW (AWS) + CGW (customer) + route propagation + ICMP in SG for ping. Multiple sites talking to each other via one VGW = CloudHub.
- DX = private, not encrypted, slow to provision (>1 month). Multi-region VPCs -> DX Gateway. Encrypted -> DX + VPN. Maximum resiliency = 2 locations x 2 connections. Cheap backup = Site-to-Site VPN.
- Transitive hub across many VPCs/on-prem -> Transit Gateway. IP multicast -> TGW only. Higher VPN throughput -> ECMP with several VPN connections on TGW. Share DX across accounts -> DX gateway + TGW.

### [[VPC Monitoring, IPv6 & Network Firewall]]
- Inbound ACCEPT + outbound REJECT = NACL. Flow logs -> S3 + Athena for analysis; CloudWatch Logs for alarms. Needs an IAM role for CloudWatch Logs.
- "Capture and inspect traffic without disrupting the instance" -> Traffic Mirroring (source ENI -> NLB/ENI target).
- Cannot launch an instance in an IPv6-enabled VPC? It is **not** IPv6 exhaustion (huge space): it is **no free IPv4 addresses in the subnet**. Fix: **add a new IPv4 CIDR** to the VPC/subnet.
- IPv4 outbound-only = NAT gateway; IPv6 outbound-only = egress-only internet gateway.
- Cheapest = same AZ + private IP. Prefer private IPs over public/Elastic IPs. Gateway endpoint beats NAT gateway for S3 cost. Traffic into AWS is free; out is charged.
- Sophisticated VPC-wide filtering (L3-L7, domain/protocol/IP filtering, intrusion prevention) -> AWS Network Firewall.

## Security & Encryption

### [[KMS, CloudHSM & ACM]]
- KMS = managed keys, IAM + key policy, CloudTrail audit, regional. Cross-account sharing of an encrypted snapshot needs a customer managed key with a custom key policy. No key policy = no access.
- Multi-Region keys = same key ID and material across Regions, enable cross-Region client-side encryption (Global Tables / Global Aurora). Not a global service.
- Need to manage your own keys on dedicated, single-tenant hardware, or SSE-C style control -> CloudHSM. Access via IAM only -> KMS.
- CloudFront / edge-optimized API Gateway -> certificate in us-east-1. Imported certs do not auto-renew. Expiry alerts: EventBridge (45 days) or Config rule. DNS validation = easy auto-renewal. See [[CloudFront & Global Accelerator]], [[API Gateway, Step Functions & Cognito]].

### [[Parameter Store & Secrets Manager]]
- Parameter Store = cheap/free config and secrets, hierarchical paths, KMS optional, TTL only on advanced tier.
- Secrets with **rotation** or **RDS/Aurora integration** -> Secrets Manager.

### [[WAF, Shield & Firewall Manager]]
- WAF = Layer 7, web ACL, targets ALB / API Gateway / CloudFront / AppSync / Cognito (never NLB). Rate-based rule = DDoS/brute force per IP. Need fixed IP + WAF -> Global Accelerator + ALB.
- WAF protects applications individually, Shield protects against DDoS, Firewall Manager centralizes policies across the Organization. They are used together.
- Edge location services (CloudFront, Global Accelerator, Route 53) + Shield/WAF + ELB + Auto Scaling + hiding backend resources = the DDoS-resilient pattern. See [[CloudFront & Global Accelerator]], [[Route 53]], [[ELB]], [[VPC]].

### [[GuardDuty, Inspector & Macie]]
- GuardDuty: threat detection from CloudTrail + VPC Flow + DNS logs; crypto attack finding; EventBridge for automation.
- Inspector = only EC2, ECR images and Lambda; CVE and network reachability. Not for S3 or other resources.
- Sensitive data/PII in S3 -> Macie.

## Monitoring, Management & Governance

### [[CloudWatch]]
- Out of the box EC2 gives CPU, disk and network at a high level, **not memory or swap**. RAM needs a custom metric or the Unified Agent.
- Logs to S3 in batch = Export task (up to 12 h). Real-time = subscription filter -> Kinesis / Firehose / Lambda. Logs Insights = historical queries only.
- More granularity than default EC2 metrics (RAM, processes, swap) = **CloudWatch Unified Agent**. See [[Parameter Store & Secrets Manager]] for central config.
- Alarm states: OK / INSUFFICIENT_DATA / ALARM. Composite alarms = AND/OR of other alarms. Alarm on a status check -> EC2 recovery keeps IPs and placement group.
- "Top N contributors" = Contributor Insights. Containers = Container Insights. Know these only at this level.

### [[CloudTrail & Config]]
- CloudTrail = who did what API call. Data events (S3 objects, Lambda invoke) are **off by default**. Beyond 90 days -> S3 + Athena. API call alerts -> CloudTrail + EventBridge.
- Config = configuration history + compliance rules; **does not prevent** actions. Remediate with SSM Automation. Per-region, use aggregators for multi-account.
- Performance/metrics -> CloudWatch. Who made the API call -> CloudTrail. What changed and is it compliant -> Config. They are complementary. See [[ELB]].

### [[CloudFormation]]
- CloudFormation = infrastructure as code; repeat an architecture across **environments, regions or accounts**.
- Users need **iam:PassRole** to hand a service role to CloudFormation.

### [[Systems Manager]]
- "Shell without opening port 22 / no bastion / no SSH keys" = Session Manager.
- Instructor: if unsure, remember the general idea of each feature; Config remediation uses SSM Automation.

### [[Cost Management & Billing]]
- Billing access for IAM users needs root to enable it. Budgets = alerts on actual/forecasted cost; Bills = per-service breakdown.
- Cost Explorer = analyze past cost, get Savings Plan recommendations, forecast future cost.
- Stop/start EC2 and RDS on a schedule to save cost -> Instance Scheduler on AWS (CloudFormation + DynamoDB + Lambda).

### [[Well-Architected & Trusted Advisor]]
- Memorize the six pillar names only; the exam does not expect deep pillar detail from this course.
- Trusted Advisor = automated checks in 6 categories; full checks + Support API need Business/Enterprise support. Well-Architected Tool = review workloads against 6 pillars.

## Migration, DR & Architectures

### [[Disaster Recovery & Backup]]
- RPO = data loss tolerated. RTO = downtime tolerated. Lower targets cost more.
- Scenario questions ask which strategy to recommend. Pilot Light = only core (DB) running, compute created on failover. Warm Standby = everything running, small, scale up on failover. Hot site = full scale, lowest RTO, highest cost.
- AWS Backup = central, policy-based, cross-Region/cross-account backups. Vault Lock = WORM, nobody (not even root) can delete backups.

### [[Migration Services]]
- Different engines = DMS **plus SCT**. Same engine = DMS only. Continuous replication = CDC. DMS needs a replication instance (or serverless).
- MGN = lift-and-shift / rehost of servers, continuous replication then cutover. DMS = databases. Same agent-and-staging idea as [[Disaster Recovery & Backup]] DRS.
- Existing VMware on-premises and want to extend/migrate to AWS with the same tooling = VMware Cloud on AWS.

### [[Classic Solution Architectures]]
- Alias record to an ELB (not A record). ELB health checks stop traffic to bad instances. SG referencing: EC2 only accepts traffic from the ELB's SG. Multi-AZ for HA; reserve the baseline.
- Stickiness = simple but not resilient. Cookies = stateless but small/untrusted. Session ID + ElastiCache/DynamoDB = stateless web tier with server-side state. Read replicas or cache to scale reads; Multi-AZ for DR.
- EBS = one instance, one AZ. Many instances/AZs needing the same files = EFS.
- Blocking one IP: NACL deny (subnet) or WAF (ALB/CloudFront). SG cannot deny. Behind CloudFront, NACL/SG see CloudFront IPs, not the client.
- ENA = enhanced networking; EFA = HPC / MPI / OS-bypass (Linux); cluster placement group = best network performance; ParallelCluster + EFA; FSx for Lustre for HPC storage. HPC is a combination of services, not a single service.
- Patterns: alarm -> Lambda -> move EIP; ASG 1/1/1 + User Data + role; lifecycle hooks + snapshots to move EBS data across AZs.

## Machine Learning

### [[Machine Learning Services]]
- Rekognition = images/video; content moderation with confidence threshold, then human review via A2I.
- Polly: stylized words/acronyms -> **lexicons**; whisper/phonetic/more control -> **SSML**.
- Lex = ASR/chatbots. Connect = contact centers.
- Document search service -> Kendra.
- Personalized recommendations -> Personalize.

