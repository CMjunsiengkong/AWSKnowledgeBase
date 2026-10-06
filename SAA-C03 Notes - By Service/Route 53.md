---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Route 53
version: C (by service)
source_chapters: [10]
related: [EC2, ELB, CloudFront & Global Accelerator, CloudWatch, VPC, VPC Connectivity, Elastic Beanstalk]
tags: [aws, saa-c03, route53, dns, routing-policies, health-checks, alias, resolver]
---

# Amazon Route 53

Concept-only note merged from chapter 10. Lab steps (registering a domain, creating records, dig/nslookup demos, health check and policy demos, cleanup) are in [[10 - Route 53]] (Version B). Related: [[ELB]], [[CloudFront & Global Accelerator]], [[CloudWatch]], [[VPC]].

## 1. DNS fundamentals
(src: 10/01-What is a DNS -)
- **DNS** translates human-friendly hostnames into IP addresses; hierarchical naming (root, TLD, second-level domain, subdomain).
- Terminology: **domain registrar** (where you register names: Route 53, GoDaddy...), **DNS records** (A, AAAA, CNAME, NS...), **zone file** (contains the records), **name servers** (resolve the queries), **TLD** (.com, .us, .in, .gov, .org), **second-level domain** (amazon.com).
- Example `http://api.www.example.com.`: trailing dot = **root**; `.com` = TLD; `example.com` = second-level domain; `www.example.com` = subdomain; `api.www.example.com` = **FQDN**; `http` = protocol; the whole thing = URL.
- **Resolution flow**: browser asks the **local DNS server** (company/ISP) -> if unknown, it asks the **Root DNS server** (ICANN), which returns the NS for `.com` -> asks the **TLD server** (.com), which returns the NS of `example.com` -> asks the **second-level domain server** (managed by the registrar/DNS service, e.g. Route 53), which returns the **A record** (e.g. 9.10.11.12). The local DNS server **caches** the answer and returns it to the browser.

## 2. Route 53 overview
(src: 10/02-Route 53 Overview)
- **Highly available, scalable, fully managed and authoritative DNS** (authoritative = you can update the records). Also a **domain registrar**, and can **check the health of resources**.
- **The only AWS service with a 100% availability SLA.** [verify] (unconfirmed - no source fetched)
- Name "53" = the traditional DNS port.
- Each record contains: domain/subdomain name, **record type**, **value**, **routing policy** (how Route 53 responds to queries), **TTL** (time records are cached at resolvers).

### Record types
| Type | Maps | Notes |
|---|---|---|
| **A** | hostname -> IPv4 | |
| **AAAA** | hostname -> IPv6 | |
| **CNAME** | hostname -> another hostname | target may itself be A/AAAA; **not allowed at the zone apex** (no CNAME for `example.com`, fine for `www.example.com`) |
| **NS** | name servers of the hosted zone | controls how traffic is routed to the domain |

(Other record types exist; only A, AAAA, CNAME, NS are needed for the exam.)

### Hosted zones
- A **hosted zone** = container of records defining how to route traffic to a domain and its subdomains.

| | Public hosted zone | Private hosted zone |
|---|---|---|
| Answers | queries from the **public internet** | queries only from **within your VPC(s)** |
| Domain | public domain you own (`app1.mypublicdomain.com`) | private names (`app1.company.internal`, `database.example.internal`) |
| Use | public records | identify private resources (EC2, DB) by private names, resolving to private IPs |

- **Cost**: **$0.50 per month per hosted zone**; domain registration **from ~$12 per year** (varies by TLD; ~$13 shown in the demo). Route 53 is not free.
- A newly registered domain gets a hosted zone with **NS** and **SOA** records; the NS records mean Route 53 is the source of truth for the records. Domain registration can take minutes to hours; keep **auto-renew on** if you want to keep the domain; enable **privacy protection** to hide contact data.

## 3. TTL
(src: 10/06-Route 53 - TTL)
- TTL tells clients/resolvers to **cache** the answer for that many seconds, so they do not query DNS again until it expires.

| TTL | Effect |
|---|---|
| **High** (e.g. 24 h) | less traffic and **lower cost** on Route 53, but clients may hold **outdated** records for a long time |
| **Low** (e.g. 60 s) | more queries (**more cost**, billed per query), but record changes propagate quickly |

- Change strategy: lower the TTL ahead of time (e.g. 24 h before), change the record once all caches have the low TTL, then raise the TTL again.
- A record change is not seen by clients until their cached copy expires.
- **TTL is mandatory on every record except Alias records.**

> [!tip] Exam
> TTL mandatory except for Alias records. High TTL = cheaper but stale; low TTL = costlier but fast changes.

## 4. CNAME vs Alias
(src: 10/07-Route 53 CNAME vs Alias)
- AWS resources (load balancer, CloudFront...) expose a hostname; to map it to your own domain you use CNAME or Alias.

| | CNAME | Alias |
|---|---|---|
| Points to | any hostname | **AWS resource** only (Route 53 extension to DNS) |
| Zone apex (`example.com`) | **No** (rejected: "CNAME is not permitted at apex") | **Yes** |
| Cost | normal query charges | **free of charge** (queries to Alias records for AWS resources) |
| Health check | no native | **native health check** (Evaluate Target Health) |
| Record type | CNAME | always **A or AAAA** |
| TTL | you set it | **cannot be set**; set automatically by Route 53 |
| IP changes of target | - | automatically recognized (e.g. ALB IP changes) |

- **Alias targets**: Elastic Load Balancers, CloudFront distributions, API Gateway, Elastic Beanstalk environments, **S3 websites** (not plain buckets), VPC interface endpoints, Global Accelerator, **Route 53 records in the same hosted zone**.
- **Not possible**: an Alias for an **EC2 DNS name**.

> [!tip] Exam
> Root domain pointing to an ELB/CloudFront -> **Alias record (A/AAAA)**, never CNAME. Alias = free, native health check, no TTL. EC2 DNS name cannot be an Alias target.

## 5. Routing policies
(src: 10/08-Routing Policy - Simple, 10/09-Routing Policy - Weighted, 10/10-Routing Policy - Latency, 10/13-Routing Policy - Failover, 10/14-Routing Policy - Geolocation, 10/15-Routing Policy - Geoproximity, 10/16-Routing Policy - IP-based, 10/17-Routing Policy - Multi Value)
- **"Routing" here is DNS-level**: Route 53 only **answers DNS queries**; traffic does not pass through it (unlike a load balancer). Clients then connect to the returned endpoint.
- Policies: simple, weighted, latency, failover, geolocation, geoproximity, IP-based, multi-value.

| Policy | How it answers | Health checks | Key facts |
|---|---|---|---|
| **Simple** | one resource, or multiple values in one record | **No** | with multiple values the **client picks one at random**; with Alias only **one AWS resource** target |
| **Weighted** | by relative weight: weight / sum of weights | Yes | records must share **same name and type**; weights need not sum to 100; weight **0** = stop sending traffic; if **all weights are 0**, all records returned equally; uses: regional load balancing, testing a new version with small traffic |
| **Latency** | lowest latency between user and the AWS region of the record | Yes | you must specify the **region** of each record (IPs alone give no location); latency measured from user to AWS regions |
| **Failover** | primary / secondary (active-passive) | **Primary: mandatory**; secondary optional | exactly **one primary and one secondary**; if primary unhealthy, secondary is returned |
| **Geolocation** | by **user location**: continent, country, or US state (most precise match wins) | Yes | create a **Default** record for no match; uses: localization, content restriction, load balancing |
| **Geoproximity** | by geographic location of users **and resources**, adjustable with a **bias** | Yes | needs **Route 53 Traffic Flow** (advanced); see below |
| **IP-based** | by **client IP / CIDR** list mapped to locations | - | uses: optimize performance, **reduce network costs**, route a known ISP's CIDRs to a specific endpoint |
| **Multi-value** | returns **up to 8 healthy records** | Yes (only healthy returned) | **client-side load balancing**; **not a substitute for ELB**; safer than simple (which may return unhealthy IPs) |

### Latency vs Geolocation
- Latency = **fastest**, may send a German user to the US if that is quicker. Geolocation = **where the user is**, regardless of speed.

### Geoproximity and bias
- Route users to resources by geographic distance; a **bias** changes the size of a resource's geographic area.
  - **Bias increased (positive)** -> area expands -> more traffic attracted to that resource.
  - **Bias decreased (negative)** -> area shrinks -> less traffic.
- Resources: **AWS resources** -> specify the AWS Region; **non-AWS resources** (e.g. on-premises) -> specify **latitude and longitude**.
- Example: us-west-1 and us-east-1 at bias 0 split the US by a line down the middle; setting us-east-1 to +50 moves the dividing line toward the west, so more users go to us-east-1.

> [!tip] Exam
> Shift traffic between regions -> **Geoproximity (bias)**. Per-country content/compliance -> **Geolocation** (+ Default). Lowest latency -> **Latency**. Active-passive -> **Failover** (primary needs health check). Known client CIDRs -> **IP-based**. Up to 8 healthy answers -> **Multi-value**. Percent split / canary -> **Weighted**.

## 6. Health checks
(src: 10/11-Route 53 - Health Checks, 10/12 concepts, 10/13)
- Purpose: monitor mainly **public** resources and drive **automated DNS failover** by associating health checks with records. Health check metrics are available in **CloudWatch**.
- **Three types**:
  1. **Endpoint** health checks (application, server, other AWS resource; public endpoint).
  2. **Calculated** health checks: combine results of **child** health checks (OR, AND, NOT); up to **255 child health checks**; you set how many must pass. Use case: perform maintenance without making the whole check fail.
  3. **CloudWatch alarm** health checks: health follows an alarm state; the way to monitor **private resources**.
- **Endpoint check mechanics**:
  - About **15 global health checkers** send requests; healthy if endpoint answers **2xx or 3xx** (or the configured response).
  - Interval: **30 seconds** (standard) or **10 seconds** (fast, higher cost).
  - Protocols: **HTTP, HTTPS, TCP**.
  - Healthy if **more than 18%** of the checkers report healthy [verify] (unconfirmed - the AWS page fetched did not state this threshold); you can choose which checker locations to use.
  - **String matching**: for text responses, checkers inspect the **first 5,120 bytes**. [verify] (unconfirmed)
  - You must **allow inbound requests from the Route 53 health checker IP ranges** (published by AWS) in security groups/firewalls; a failed check due to a blocked port shows a connection timeout.
- **Private resources**: health checkers sit on the public internet, outside your VPC, so they **cannot reach private endpoints**. Create a **CloudWatch metric + alarm** on the private resource and attach a health check that monitors the alarm.

> [!warning] Correction [note]
> AWS documents the same three types plus an Application Recovery Controller (ARC) routing control type. For alarm-based checks, Route 53 does not wait for the alarm to enter ALARM; it evaluates the same data stream, only same-account alarms with standard-resolution metrics are supported. Source: [Types of Amazon Route 53 health checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-types.html).

> [!warning] Correction [note]
> Multi-value: AWS confirms Route 53 returns **up to eight healthy records** and that it "isn't a substitute for a load balancer"; records without a health check are always considered healthy. Source: [Multivalue answer routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-multivalue.html).

> [!tip] Exam
> Private resource health check = CloudWatch alarm. Failover primary record needs a health check. Allow health checker IPs in the firewall.

## 7. Third-party registrar with Route 53 DNS
(src: 10/18-3rd Party Domains & Route 53)
- **Registrar and DNS service are different things**, though registrars usually bundle DNS.
- You can register a domain at GoDaddy (or any registrar) and still use **Route 53 as the DNS service**: create a **public hosted zone** in Route 53, then replace the **NS (name server) records** at the registrar with the **four Route 53 name servers** of that zone.
- The reverse also works: Route 53 registrar with another DNS provider.

> [!tip] Exam
> Third-party domain + Route 53 DNS = public hosted zone + update NS records at the registrar.

## 8. Route 53 Resolver and hybrid DNS
(src: 10/19-Route 53 Resolvers & Hybrid DNS)
- By default the **Route 53 Resolver** answers queries for local VPC/EC2 names, **private hosted zone** records and public records.
- **Hybrid DNS** (resolve names between AWS and on-premises) needs connectivity (**VPN or Direct Connect**) plus **Resolver endpoints**:

| Endpoint | Direction | Use |
|---|---|---|
| **Inbound endpoint** | on-premises -> AWS | on-premises DNS resolvers forward queries to it to resolve AWS names (e.g. private hosted zone) |
| **Outbound endpoint** | AWS -> on-premises | Route 53 Resolver forwards queries for on-premises names (e.g. `web.onpremise.private`) to on-premises DNS resolvers |

> [!tip] Exam
> Two-way DNS between AWS and a data center = Resolver **inbound + outbound** endpoints over VPN/Direct Connect. See [[VPC Connectivity]].

## Not included here
- Hands-on narration: domain registration, first records, EC2/ALB setup for the demos, TTL/Simple/Weighted/Latency/Geolocation/Failover/Multi-value demos (VPN tests), health check creation, section cleanup (10/03, 10/04, 10/05, 10/12, 10/20) -> [[10 - Route 53]].
- Concepts inside those labs are merged above (cost of domain/hosted zone, health check options, apex CNAME rejection).
- All 20 lectures have transcripts.
