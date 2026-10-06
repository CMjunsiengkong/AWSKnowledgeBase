---
course: Ultimate AWS Certified Solutions Architect Associate 2026
version: B (by chapter)
title: Exam Cheat Sheet
tags: [aws, saa-c03]
---
# Exam Cheat Sheet - By Chapter

Automatically collected from the `Exam` callouts of each chapter note (nothing added). Open the chapter note for context, tables, limits and corrections.

## [[03 - Getting Started with AWS]]
- How to choose a region: **compliance** (data residency, e.g. French data stays in France), **latency** (close to users), **service availability** (not all regions have all services), **pricing** (varies by region).
- **Global services**: IAM, Route 53, CloudFront, WAF. **Region-scoped**: EC2, Elastic Beanstalk, Lambda, Rekognition. Check the regional services table for availability.

## [[04 - IAM & AWS CLI]]
- Groups contain only users. IAM is global. Always least privilege.
- Know Effect, Principal, Action, Resource. Inline policy = attached to a single user.
- Know the four MFA device options and that password policy + MFA are the two protection mechanisms.
- CLI/SDK = access keys; console = password (+MFA). CLI is built on the Python SDK (Boto).
- CLI calls use the same IAM permissions as the console; keys are visible only once at creation.
- Services (EC2, Lambda, CloudFormation) get permissions through roles, never through users/access keys.
- Credentials Report = account level, per-user credential status. Access Advisor = user level, last-accessed services.
- Root only for setup, MFA everywhere, roles for services, groups for permissions, least privilege.

## [[05 - EC2 Fundamentals]]
- EC2 User Data = bootstrap script, runs once at first launch, as root.
- Public IP may change on stop/start; private IP does not. Terminate deletes the root volume by default.
- Map workload to family: CPU-heavy -> C, RAM-heavy -> R/X/Z, local disk IOPS-heavy -> I/D/H.
- Timeout = security group. Connection refused = app problem. 22 = Linux SSH, 3389 = Windows RDP.
- Any timeout when connecting (SSH, HTTP, anything) is a security group problem.
- EC2 gets AWS credentials via IAM roles (instance profile), never by storing access keys on the instance.
- Match the option to the workload: Spot for resilient batch jobs, never DBs; Dedicated Host for BYOL/compliance; Capacity Reservation guarantees capacity but gives no discount.
- Cancel the Spot request, *then* terminate instances. Spot Fleet strategies: lowestPrice, diversified, capacityOptimized, priceCapacityOptimized (best for most workloads).

## [[06 - EC2 Solutions Architect Associate Level]]
- Fixed public IP needed -> Elastic IP, but the "better" answers are DNS name (Route 53) or a load balancer. Limit: 5 Elastic IPs per account.
- Cluster = performance (single AZ). Spread = HA for critical apps (7 per AZ). Partition = large distributed, partition-aware (Kafka, Cassandra, Hadoop).
- ENI = virtual NIC, AZ-bound, movable between instances for failover; carries private IPs, Elastic IPs, security groups, MAC.
- Hibernate = RAM dumped to encrypted root EBS; fast resume, keeps in-memory state. Root volume must be EBS, encrypted, big enough for RAM.

## [[07 - EC2 Instance Storage]]
- Root volume deleted on terminate by default, other attached volumes kept. EBS = AZ-locked network drive.
- Move an EBS volume across AZ/Region = snapshot + restore/copy. Recycle Bin = accidental deletion; Archive = cheaper but slow restore; FSR = fast but expensive.
- "Very high performance hardware-attached volume" = **EC2 Instance Store**. Ephemeral: not for long-term data.
- Database / sustained IOPS -> io1/io2. General boot/dev -> gp2/gp3. Big data throughput -> st1. Cheapest cold -> sc1. >32,000 IOPS needs Nitro + io1/io2.
- Multi-Attach = io1/io2, same AZ, max 16 instances, cluster-aware file system.
- Encrypting an existing unencrypted EBS volume: snapshot -> encrypted copy -> new volume. Related: [[KMS, CloudHSM & ACM]].
- EFS = shared, multi-AZ, Linux-only NFS; use lifecycle policies (IA/Archive) to cut cost; Elastic throughput for unpredictable workloads.
- Shared storage for many instances across AZs on Linux = EFS; single-AZ block volume = EBS; ultra-fast ephemeral = Instance Store.

## [[08 - High Availability and Scalability - ELB & ASG]]
- Exam questions use these terms to trick you: scale up/down = vertical, scale out/in = horizontal, multi-AZ = high availability.
- EC2 security group should reference the load balancer's security group as source.
- ALB = HTTP/HTTPS/WebSocket, path/host/query/header routing, microservices and containers; client IP in X-Forwarded-For.
- Fixed response, redirect and forward are the three ALB rule actions; priorities 1-50,000.
- Extreme performance, TCP/UDP, or static/Elastic IPs -> Network Load Balancer.
- Third-party firewalls/IDPS appliances, Layer 3, GENEVE 6081 -> Gateway Load Balancer.
- Sticky sessions = cookie-based affinity; trade-off is uneven load.
- ALB: cross-zone on by default, no inter-AZ charge. NLB/GWLB: off by default, paid if enabled.
- Multiple SSL certificates / multiple domains on one load balancer -> ALB or NLB with SNI (never CLB).
- Connection draining (CLB) = deregistration delay (ALB/NLB); default 300 s, 0 = off.
- ASG = min/desired/max, free, uses a launch template, replaces unhealthy instances, integrates with ELB and CloudWatch alarms.
- Know the four policy types; cooldown default 300 s; pre-baked AMI shortens warm-up; RequestCountPerTarget is an ALB-based scaling metric.
- Target tracking auto-creates the high/low CloudWatch alarms; scale-in is slower (15 data points) than scale-out (3 data points).

## [[09 - RDS, Aurora & ElastiCache]]
- Unpredictable storage growth on RDS -> enable **RDS Storage Auto Scaling** with a maximum storage threshold.
- Read Replicas = scale reads, async. Multi-AZ = DR, sync, one DNS name, auto failover. Same-Region replica traffic is free, cross-Region is paid. Single-AZ -> Multi-AZ needs no downtime.
- Need OS-level access / custom patches on Oracle or SQL Server while still on RDS -> **RDS Custom**.
- Remember: **writer endpoint, reader endpoint, replica auto-scaling, shared auto-expanding storage, 6 copies / 3 AZ**.
- "Replicate across Regions in **< 1 second**" or cross-Region DR with RTO < 1 min -> **Aurora Global Database**. SQL Server app moving to Aurora with minimal changes -> **Babelfish**. Unpredictable / intermittent load -> **Aurora Serverless**.
- Quick staging copy of Aurora prod -> **cloning**. Rarely used DB -> snapshot + delete + restore. Aurora MySQL import from S3 needs **Percona XtraBackup**.
- Encrypt an unencrypted RDS DB -> snapshot, copy/restore as encrypted. Unencrypted master = unencrypted replicas.
- Many connections / Lambda overloading RDS, faster failover, or enforcing IAM authentication to the DB -> **RDS Proxy**.
- Offload a read-heavy DB or share session state across instances -> **ElastiCache**. Needs HA / persistence / backup -> **Redis**; simple multi-threaded sharded cache -> Memcached.
- **Gaming leaderboard = Redis Sorted Sets**. Lazy Loading = possible stale data; Write Through = always fresh. IAM auth = Redis only.

## [[10 - Route 53]]
- Know A, AAAA, CNAME, NS. CNAME cannot be used at the zone apex. Private hosted zone = resolution only inside your VPC.
- TTL = caching time at resolvers. Mandatory on all records except Alias. Low TTL = faster changes but more queries/cost.
- Zone apex (naked domain) -> Alias, never CNAME. Alias = free, native health check, A/AAAA only, AWS targets only (not EC2 DNS names).
- Simple = one or more values, random client choice, no health checks.
- Weighted = percentage split by weights; weight 0 = no traffic; each record needs a unique record ID, same name/type.
- Latency routing = lowest latency to the AWS region, region must be declared per record.
- Private resource health check = CloudWatch alarm health check. Allow Route 53 health checker IPs. Calculated health check = AND/OR/NOT of up to 255 children. Healthy if more than 18% of checkers say healthy; 10 s = fast (costlier), 30 s = standard; string match within first 5,120 bytes.
- Failover = active-passive, health check on primary is mandatory, one primary + one secondary.
- Geolocation = user's location (not latency). Always add a Default record.
- Geoproximity = shift traffic between regions by changing the bias; needs Traffic Flow. Do not confuse with Geolocation (fixed country/continent mapping).
- IP-based = routing by client IP/CIDR ranges.
- Multi-Value = client-side load balancing with health checks, up to 8 healthy records; Simple multi-value has no health checks.
- Third-party registrar + Route 53 DNS = public hosted zone + update the NS records at the registrar.
- Hybrid DNS both ways = Resolver **inbound** (on-premises -> AWS) and **outbound** (AWS -> on-premises) endpoints.

## [[11 - Classic Solutions Architecture Discussions]]
- Public vs private IP placement; EIP vs Route 53 vs ELB; **A record cannot be used with an ELB, use Alias**; ELB health checks; ASG for elasticity; Multi-AZ for survival of an AZ failure; reserve the baseline, On-Demand/Spot for the rest; SG referencing (EC2 accepts only from ELB SG).
- Stickiness = ELB feature; cookies = stateless but small/untrusted; session ID + ElastiCache (or DynamoDB) = secure and common; RDS Read Replicas or ElastiCache to scale reads; Multi-AZ for DR; chain security groups.
- Single instance -> EBS; many instances / multi-AZ shared files -> EFS (NFS). Aurora = fewer operations than RDS.
- Speed-up toolbox: Golden AMI, User Data, RDS snapshot restore, EBS snapshot restore.
- Beanstalk = PaaS reusing EC2/ASG/ELB/RDS, free service, pay for resources. Know web vs worker tier and single-instance vs HA modes.

## [[12 - Amazon S3 Introduction]]
- S3 buckets are regional but names are (by default) global; key = prefix + object name; max object 50 TB, multi-part upload recommended/required above 5 GB; tags max 10.
- Cross-account access -> bucket policy. EC2 -> IAM role. Public access needs **both** Block Public Access off and a public bucket policy.
- 403 Forbidden on an S3 website = missing public read (bucket policy / Block Public Access).
- Delete of a versioned object = delete marker; suspend != delete versions; pre-existing objects have `null` version.
- Replication needs **versioning on both buckets** + IAM role. CRR = compliance/latency; SRR = log aggregation / prod-test sync.
- Existing objects -> Batch Replication; version-ID deletes not replicated; delete markers optional; no chaining.
- Know each class's purpose: Standard (frequent), IA (infrequent, rapid), One Zone-IA (re-creatable, single AZ), Glacier IR (ms, 90 days), Flexible (min-hours), Deep Archive (12-48 h, 180 days, cheapest), Intelligent-Tiering (unknown patterns, no retrieval fee).
- Express One Zone = directory bucket, single AZ, ultra-low latency; pick it for latency-sensitive, co-located compute workloads.

## [[13 - Advanced Amazon S3]]
- Transition = change class; expiration = delete. S3 Analytics works for Standard -> Standard IA only. Versioned buckets: use non-current-version transitions/expiration.
- Requester Pays = requester pays the download cost, must be authenticated; storage stays with the owner.
- Targets: SNS, SQS, Lambda (resource policies) plus EventBridge. Access is granted by resource policy, not IAM role.
- 3,500 write / 5,500 read per second per prefix. Multipart upload for big files; Transfer Acceleration for long distance; Byte-Range Fetches for parallel or partial reads.
- Find unencrypted objects with S3 Inventory (+ Athena), then encrypt them all with S3 Batch Operations.
- Storage Lens = org-wide visibility. Know free vs paid, that the default dashboard spans accounts/regions, and that it can show e.g. how many objects are encrypted.

## [[14 - Amazon S3 Security]]
- SSE-S3 = AWS-managed AES-256 (default). SSE-KMS = KMS key + CloudTrail audit, watch KMS quotas. DSSE-KMS = two layers. SSE-C = your key, HTTPS only. Client-side = you encrypt. In transit = HTTPS / `aws:SecureTransport`.
- CORS question: images/assets in one S3 bucket requested from a page on another origin -> configure CORS on the bucket being requested (the cross-origin one).
- MFA Delete protects against permanent version deletion and versioning suspension; versioning must be on; root only.
- Vault Lock = policy-level WORM on Glacier, irreversible. Object Lock = per-object-version WORM: Compliance (no one, incl. root) vs Governance (privileged IAM can bypass); Legal Hold ignores retention and needs `s3:PutObjectLegalHold`.

## [[15 - CloudFront & Global Accelerator]]
- CDN = CloudFront. S3 origin secured with OAC. CloudFront = caching at the edge (global); S3 CRR = full bucket copy to chosen regions.
- Private ALB/NLB/EC2 behind CloudFront = **VPC origin**. Old pattern = public origin with security group allowing CloudFront IPs.
- Geo restriction = allow list / block list of countries, based on a Geo-IP database.
- Updated origin but users still see old content -> invalidate the CloudFront cache (`/*` or specific paths).
- Static anycast IPs / non-HTTP (TCP/UDP) / fast regional failover -> Global Accelerator. Caching content at the edge -> CloudFront.

## [[16 - AWS Storage Extras]]
- Offline petabyte migration or limited bandwidth -> Snowball. Edge computing without connectivity -> Snowball Edge Compute Optimized.
- Snowball -> S3 -> lifecycle policy -> Glacier.
- Windows shares + AD -> FSx for Windows. HPC / ML with S3 -> FSx for Lustre. Move NetApp / NAS workloads, multi-protocol -> ONTAP. Move ZFS workloads, NFS -> OpenZFS.
- On-premises NFS/SMB over S3 -> S3 File Gateway. On-premises block volumes backed up to AWS -> Volume Gateway (cached vs stored). Physical tape replacement -> Tape Gateway.
- FTP/FTPS/SFTP into S3 or EFS -> Transfer Family.
- Scheduled sync preserving metadata/permissions -> DataSync. On-premises NFS/SMB source requires the agent.
- Pick the storage service from the requirement keywords: protocol (SMB / NFS / iSCSI / FTP), workload (HPC, tape backup, hybrid), and size/bandwidth (offline Snow devices).

## [[17 - Decoupling Applications - SQS, SNS, Kinesis, Amazon MQ]]
- Sudden spikes / unpredictable load / "decouple" -> think SQS, SNS or Kinesis.
- SQS standard: unlimited throughput, 4 d default / 14 d max retention, at-least-once, best-effort ordering. Scale consumers with an ASG on queue length. Use an access policy so SNS / S3 can write to the queue.
- Default visibility timeout = 30 s; `ChangeMessageVisibility` to extend; scenarios on duplicate processing are likely.
- Long polling = 1-20 s, `WaitTimeSeconds`, fewer API calls, lower latency.
- Need ordering and/or no duplicates -> FIFO (message group ID for order, dedup ID for exactly once).
- Decoupling, sudden spike load, timeouts, or "don't lose transactions" -> SQS buffer (with ASG on queue length). Very common in the exam.
- One message to many receivers = SNS pub/sub. Topic access policy for cross-account / S3 events.
- Fan-out = SNS topic + SQS queues. S3 one-rule limitation -> fan-out via SNS. FIFO fan-out = SNS FIFO + SQS FIFO. Filter policy = JSON per subscription.
- Real-time -> Kinesis Data Streams. Provisioned = shards (1 MB/s in, 2 MB/s out each); on-demand = auto scale. Retention up to 365 days with replay. Same partition key = same shard = ordered.
- Near real-time load into S3 / Redshift / OpenSearch = Data Firehose. Real-time with replay and custom consumers = Data Streams.
- SQS = pull + delete, no replay. SNS = push to many, not persistent. Kinesis = ordered per shard, replay, retention up to 365 days, enhanced fan-out for many consumers.
- Migrating an on-prem app that uses MQTT / AMQP / STOMP / Openwire / WSS -> Amazon MQ. Cloud-native new build -> SQS/SNS. HA = multi-AZ active/standby with EFS.

## [[18 - Containers on AWS]]
- Docker keyword = microservices / lift-and-shift. Storing Docker images on AWS = ECR.
- The exam loves **Fargate**: serverless and much easier to manage than the EC2 launch type.
- ECS service scaling metrics: CPU, memory, ALB request count per target. EC2 launch type: use Cluster Capacity Provider rather than plain ASG scaling. Fargate = simplest.
- "Storing Docker images" = ECR.
- Kubernetes / pods / "already using Kubernetes" / cloud-agnostic = EKS. EFS is the storage class compatible with EKS on Fargate.

## [[19 - Serverless Overviews]]
- Serverless does not mean no servers; it means you do not provision or see them. The exam tests serverless knowledge heavily.
- Lambda = short, on-demand, auto-scaling, pay per request + duration. Event-driven patterns: S3 upload -> Lambda; EventBridge schedule -> Lambda (serverless cron).
- "30 GB RAM", "30 minutes of execution" or "a 3 GB file" in the question means Lambda is NOT the right choice.
- Sync throttle = 429; async = retry up to 6 h then DLQ. One runaway function can starve others unless you set reserved concurrency. Cold start fix = provisioned concurrency.
- SnapStart = pre-initialized snapshot taken at version publish; faster cold starts without paying for provisioned concurrency.
- Simple high-volume header/URL/JWT tweaks at viewer stage = CloudFront Functions. Needs network, SDK, body access or origin events = Lambda@Edge.
- Lambda needs VPC access to reach private RDS / ElastiCache. Many Lambdas + RDS = use RDS Proxy (and put Lambda in the VPC).
- Reacting to data changes in the DB = invoke Lambda from the DB. RDS event notifications = infrastructure events only, no data events.
- Sudden steep spikes or near-zero traffic -> On-demand. Predictable load -> Provisioned (+ auto scaling).
- Microsecond cache -> DAX. Change stream: DynamoDB Streams (24 h) vs Kinesis (1 yr). Active-active multi-region -> Global Tables (needs Streams). Auto-expire sessions -> TTL. Restore always creates a new table.
- API Gateway = serverless front door with auth, throttling, API keys, caching, versioning, stages, and direct AWS-service integrations (Kinesis, SQS, Step Functions).
- Complex multi-step/branching workflow orchestration (with retries and human approval) = Step Functions.
- Web/mobile users + sign-in + API Gateway/ALB = Cognito User Pools. Direct temporary AWS access (S3, DynamoDB), row-level security = Cognito Identity Pools.

## [[20 - Serverless Solution Architecture Discussions]]
- Mobile/web users needing AWS resources = Cognito temporary credentials, never embedded IAM user keys. Read-heavy DynamoDB = DAX. Cache REST responses = API Gateway cache.
- Know sync (API Gateway / ELB) vs async (SQS, SNS, Kinesis, Lambda triggers, S3) microservice communication.
- Mostly static content served from EC2 with high cost/load, no re-architecture wanted = add CloudFront.

## [[21 - Databases in AWS]]
- Pick the database from the question's architecture clues; later sections and chapter 22 detail each one.
- Aurora Global: under 1 s cross-region replication, up to 16 read instances per region. Cloning is faster than snapshot + restore.
- A caching solution that requires **no code change** is not ElastiCache.
- "Rapidly evolving schema" or "flexible schema" -> DynamoDB. DAX = microsecond reads.
- MongoDB -> DocumentDB. Generic NoSQL -> DocumentDB or DynamoDB.
- Graph database -> Neptune.
- Apache Cassandra -> Keyspaces.

## [[22 - Data & Analytics]]
- Serverless SQL on S3 -> Athena. Cheaper/faster: Parquet/ORC + partitioning + compression + large files.
- Analytics/OLAP/warehouse -> Redshift. Query S3 with Redshift without loading -> Spectrum. DR for single-AZ -> snapshots with cross-region copy.
- Search / partial match / free text -> OpenSearch.
- Hadoop / Spark big data cluster -> EMR.
- Common pairings: QuickSight + Athena, QuickSight + Redshift. SPICE = imported data only.
- Centralized permissions with row/column-level security across analytics tools -> Lake Formation.
- Flink does not read Firehose.
- Kafka -> MSK. MSK can only add partitions; Kinesis uses shard split/merge.

## [[23 - Machine Learning]]
- Rekognition = images and videos. Content moderation = confidence threshold, then A2I for human review.
- Transcribe = speech to text, PII redaction, automatic language identification.
- Stylized words/acronyms = lexicon. Whisper/phonetics/pauses = SSML.
- Lex = ASR/chatbots. Connect = contact center. Together = smart call center.
- Custom ML models for developers/data scientists = SageMaker.
- Document search service = Kendra.
- Personalized recommendations = Personalize.

## [[24 - Monitoring & Audit - CloudWatch, CloudTrail & Config]]
- EC2 basic monitoring = 5 min, detailed = 1 min. RAM is not a default EC2 metric: use a custom metric / Unified Agent.
- Logs Insights = query engine for historical logs, not real time.
- Batch export to S3 (not real time) vs subscription filters (real time) is a common question.
- More granular / RAM metrics from EC2 or on-premises = CloudWatch Unified Agent.
- Recovery keeps IPs, metadata and placement group. Composite alarm = combine alarms with AND/OR.
- EventBridge = CloudWatch Events; combine with CloudTrail to react to any API call.
- High level only: Container = ECS/EKS/Fargate; Lambda = serverless detail; Contributor = "top N" from logs; Application = automated app dashboard.
- Default 90-day history; longer retention = S3 (+ Athena). Data events are off by default; Insights is paid and opt-in.
- "Alert on a specific API call" = CloudTrail + EventBridge + SNS.
- Config = compliance/configuration history, per region, cannot block actions; remediate with SSM Automation.
- Performance = CloudWatch; who did it (API calls) = CloudTrail; what changed / is it compliant = Config.

## [[25 - IAM Advanced]]
- SCP never affects the management account. Explicit allow needed at every OU level; explicit deny anywhere wins. Organizations = consolidated billing + shared RI/Savings Plans.
- Keep tags consistent across accounts = Organizations **tag policies**.
- Bucket ARN vs object ARN (`/*`); `aws:PrincipalOrgID` limits resource policies to organization members; `aws:SourceIp` / `aws:RequestedRegion` restrict network and region.
- Role = lose original permissions; resource-based policy = keep them. EventBridge: resource policy for Lambda/SNS/SQS/S3, IAM role for Kinesis/ASG/SSM Run Command/ECS.
- Explicit deny always wins. No allow = implicit deny. Boundaries apply to users/roles, not groups.
- Multi-account SSO + SAML apps = IAM Identity Center. Permission sets = IAM roles in target accounts.
- Control Tower = governed multi-account on top of Organizations. Preventive = SCP, Detective = Config.

## [[26 - Security & Encryption]]
- Server-side = server handles keys. Client-side = server cannot decrypt the contents. HTTPS = TLS certificate = encryption in flight.
- Auditing key use -> CloudTrail. AWS managed keys auto-rotate yearly. Cross-account sharing needs a customer managed key + custom key policy. A KMS key never leaves its region.
- Rotation period for customer managed keys: 90 to 2,560 days. Decrypt does not need you to name the key. Customer managed key = $1/month.
- Multi-Region keys: same key ID + material, but not global; independent policies. Used for client-side encryption of specific attributes with Global Tables / Global Aurora.
- SSE-KMS replication: explicit opt-in + target KMS key + IAM role with decrypt (source) and encrypt (target).
- Launch permission on the AMI + share the KMS key + IAM permissions (DescribeKey, ReEncrypt, CreateGrant, Decrypt) in the target account.
- Parameter Store = cheap/free config + secrets with KMS, hierarchy and versioning. TTL (parameter policies) = advanced tier.
- "Secrets" or "RDS/Aurora credential rotation" -> Secrets Manager (not Parameter Store).
- CloudFront / edge-optimized API Gateway certs live in us-east-1; regional API Gateway certs in the API's region. DNS validation for auto-renew. Imported certs do not auto-renew. HTTP to HTTPS redirect is done on the ALB.
- Need to manage your own keys in dedicated single-tenant hardware, or Oracle TDE / SSL acceleration -> CloudHSM. Integrate with KMS via a custom key store.
- WAF = Layer 7, ALB/API Gateway/CloudFront/AppSync/Cognito, never NLB. Fixed IP + WAF = Global Accelerator + ALB + WAF.
- Shield Standard = free L3/L4. Shield Advanced = paid, DRT access, cost protection, auto WAF rules for L7.
- Multi-account / Organization-wide firewall policy (WAF, Shield Advanced, SGs, Network Firewall, DNS Firewall) -> Firewall Manager.
- Think in layers: edge (CloudFront / Global Accelerator / Route 53 + Shield) -> ELB + ASG -> WAF -> hide the backend with SGs/NACLs and API Gateway/CloudFront.
- Threat detection / crypto-mining / compromised EC2 via DNS -> GuardDuty. Reacting to findings -> EventBridge.
- Inspector = vulnerabilities in running EC2, ECR images and Lambda (CVE + EC2 network reachability). Not for S3 or general account threats.
- Find PII/sensitive data in S3 -> Macie.

## [[27 - Networking - VPC]]
- /32 = one IP, /0 = all IPs; /24 = 256, /16 = 65,536. Know the three private ranges.
- VPC CIDR: /28 to /16, max 5 CIDRs, 5 VPCs per region (soft). Overlapping CIDRs cannot be connected (peering etc.).
- Need 29 usable IPs? `/27` = 32 - 5 = 27 (too few). Choose `/26` = 64 - 5 = 59.
- Public subnet = route table entry `0.0.0.0/0 -> IGW`. The `local` route covers traffic inside the VPC CIDR.
- NAT instance: public subnet + Elastic IP + source/destination check disabled + route table. NAT gateway is the recommended replacement.
- NAT gateway = managed, IPv4 only, needs IGW, HA per AZ, no SG. NAT instance can act as bastion, NAT gateway cannot.
- SG = stateful, allow-only, instance. NACL = stateless, allow+deny, subnet, numbered, first match wins. Block a single IP -> NACL. Default NACL allows everything; custom NACL denies everything.
- Peering = non-overlapping CIDRs + route tables on both sides + non-transitive. 3 VPCs fully connected = 3 peerings.
- Gateway endpoint = S3 and DynamoDB only, free, route table. Interface endpoint = ENI + SG, PrivateLink, paid, everything else (and S3 for on-premises/cross-VPC).
- Inbound ACCEPT but outbound REJECT = NACL problem (stateless). Flow logs -> S3 + Athena for analysis; -> CloudWatch Logs for alarms/metric filters.
- VPN = encrypted over the internet; needs VGW + CGW; enable route propagation; CGW behind NAT -> use NAT device public IP; ICMP in SG to ping; CloudHub for multiple sites.
- DX = private, not encrypted, > 1 month lead time; DX Gateway for multi-region/VPC; public VIF for S3, private VIF for VPC; need encryption -> add VPN.
- Cheap failover for Direct Connect = Site-to-Site VPN backup.
- Transit Gateway = transitive hub; IP multicast; ECMP to boost VPN throughput; share via RAM; share Direct Connect across accounts.
- Cannot launch EC2 in an IPv6 VPC -> out of IPv4 addresses in the subnet -> add an IPv4 CIDR.
- IPv6 outbound-only from private subnet = egress-only IGW (`::/0`). IPv4 = NAT gateway (`0.0.0.0/0`).
- Private IP > public IP; same AZ is free; ingress is free, egress costs; Gateway endpoint (free) beats NAT gateway for S3 access; CloudFront in front of S3 is cheaper than serving from S3.
- VPC-wide, sophisticated L3-L7 filtering (domain, protocol, IPS) = AWS Network Firewall (managed by Firewall Manager).

## [[28 - Disaster Recovery & Migrations]]
- Scenario questions ask which strategy to pick: Backup and Restore (cheap, slow), Pilot Light (core only), Warm Standby (scaled-down full), Multi-Site/Hot Site (fastest, costliest). RPO = data loss, RTO = downtime.
- Different engines = DMS + SCT. Same engine = DMS only. Continuous replication = CDC. DMS runs on an EC2 replication instance; Multi-AZ gives a standby.
- Central, automated backup with cross-region/cross-account copy = AWS Backup. Cannot-be-deleted backups, even by root = Vault Lock (WORM).
- Lift-and-shift / rehost servers = MGN. Plan and map dependencies = Application Discovery Service. Track = Migration Hub.
- One-off, hundreds of TB, slow link: Snowball. Ongoing: VPN, Direct Connect, DMS, DataSync.

## [[29 - More Solution Architectures]]
- SQS DLQ is on the queue; Lambda async DLQ is on the Lambda. Fan-out = SNS + SQS. Need advanced filtering, archive/replay or many targets from S3 = EventBridge.
- Deny a specific IP = NACL (security groups cannot deny). Behind CloudFront, filter at CloudFront (WAF / Geo Restriction), not the NACL.
- Know ENA vs EFA vs ENI. HPC networking = cluster placement group + EFA. HPC file system = FSx for Lustre. ParallelCluster is used with EFA.
- ASG with min=max=desired=1 across 2 AZs = self-healing single instance; EBS is AZ-bound, so move data via snapshots + lifecycle hooks.

## [[30 - Other Services]]
- CloudFormation = infrastructure as code; use it to repeat an architecture in different environments, regions or AWS accounts.
- Update preview = **change set**; some changes cause **replacement** of the resource. Delete a stack to remove all its resources.
- CloudFormation service role + `iam:PassRole` = least privilege for stack deployment.
- Need shell access with no inbound ports, no keys, no bastion -> **SSM Session Manager** (instance role with SSM permissions).
- Config non-compliance -> remediation via SSM Automation; patching -> Patch Manager (+ Maintenance Window); run commands at scale without SSH -> Run Command.
- Visualize/analyze cost over time, forecast, and get Savings Plan recommendations -> Cost Explorer.
- ML-based unusual-spend detection without setting thresholds -> Cost Anomaly Detection.
- Native AWS services/APIs running in your own data center, managed by AWS, for low latency or data residency -> Outposts.
- Long-running or large-disk batch jobs in containers on Spot/EC2 -> Batch; short event-driven serverless code -> Lambda.
- Stop/start EC2 and RDS on a schedule to cut cost -> Instance Scheduler on AWS (CloudFormation + DynamoDB + Lambda).

## [[31 - WhitePapers and Architectures]]
- Know the six pillar names (not details). They are a synergy: improving operational excellence likely improves cost optimization; sustainability tends to improve performance efficiency.
- Trusted Advisor: 6 categories; full checks + Support API require Business/Enterprise support.

## [[32 - Preparing for the Exam]]
- 65 questions / 130 min / 720 pass / 15 unscored / retake after 14 days.

