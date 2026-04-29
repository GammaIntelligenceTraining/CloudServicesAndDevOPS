# AWS Cloud Technologies — Student Materials
---

# Module 01 — Introduction to cloud technologies

## What is cloud computing?

> On-demand delivery of IT resources over the internet with pay-as-you-go pricing.

Instead of buying and maintaining physical hardware, you rent computing power, storage, and networking from a provider (AWS, Azure, GCP) and pay only for what you use.

## The five NIST characteristics

Every cloud service has these five properties. If any is missing, it's not truly "cloud."

| Characteristic | What it means | Example |
|---|---|---|
| **On-demand self-service** | Get resources instantly, no human approval needed | Click "Launch instance" → server in 60 seconds |
| **Broad network access** | Available over the internet from any device | Manage AWS from a laptop, phone, or tablet |
| **Resource pooling** | Hardware shared across customers, logically isolated | Your EC2 runs on a server with other tenants; you never see them |
| **Rapid elasticity** | Scale up and down automatically with demand | 2 servers at night, 200 during a flash sale, back to 2 |
| **Measured service** | Usage is metered; you see exactly what you pay for | Billed per second of compute, per GB of storage |

## On-premises vs cloud

```
┌────────────────────────────────┬──────────────────────────────┐
│        ON-PREMISES             │           CLOUD              │
├────────────────────────────────┼──────────────────────────────┤
│ You buy hardware (CapEx)       │ You rent by the hour (OpEx)  │
│ You own the data center        │ Provider owns the building   │
│ You maintain everything        │ Provider maintains hardware  │
│ Fixed capacity (plan ahead)    │ Elastic capacity (scale now) │
│ Weeks to deploy                │ Minutes to deploy            │
│ Full control                   │ Control within provider menu │
│ High upfront cost              │ No upfront cost              │
└────────────────────────────────┴──────────────────────────────┘
```

## Cloud deployment models

| Model | Who uses it | Example |
|---|---|---|
| **Public** | Anyone can sign up | AWS, Azure, GCP |
| **Private** | One organization only | On-prem OpenStack, AWS Outposts |
| **Hybrid** | Public + private connected | Database on-prem, website on AWS |

## AWS global infrastructure

```
AWS Cloud
├── Partition (aws, aws-cn, aws-us-gov, aws-eusc)
│   ├── Region (e.g., eu-central-1 Frankfurt)
│   │   ├── Availability Zone a (1+ data centers)
│   │   ├── Availability Zone b (1+ data centers)
│   │   └── Availability Zone c (1+ data centers)
│   └── Region (e.g., us-east-1 N. Virginia)
│       ├── AZ a
│       ├── AZ b
│       ├── AZ c
│       ├── AZ d
│       ├── AZ e
│       └── AZ f
└── Edge locations (750+ worldwide for CloudFront CDN)
```

**Key numbers (April 2026):**

| Fact | Value |
|---|---|
| Regions | ~39 |
| Availability Zones | ~123 |
| Minimum AZs per Region | 3 |
| Edge locations | 750+ |
| Distance between AZs in a Region | Up to ~100 km |
| Latency between AZs in same Region | 1–2 ms |

## AWS account structure

```
Tenant (your organization)
└── AWS Account (12-digit ID)
    ├── Root user (email + password, MFA required)
    │   └── Use ONLY for account-level tasks
    └── IAM users (your daily login)
        └── Permissions controlled by policies
```

**Root user — only use for:**
- Changing account email / phone / name
- Closing the account
- Changing the support plan
- Activating IAM access to Billing

**Everything else → use your IAM user.**

## AWS service categories (the big four)

| Category | What it does | Key services |
|---|---|---|
| **Compute** | Run code / applications | EC2, Lambda, Fargate, Elastic Beanstalk |
| **Storage** | Store data | S3, EBS, EFS |
| **Networking** | Connect things | VPC, Route 53, CloudFront, ELB |
| **Security** | Control access | IAM, KMS, Secrets Manager, GuardDuty |

## AWS Free Tier (accounts created after July 15, 2025)

| | Free Account Plan | Paid Account Plan |
|---|---|---|
| Credits | $100 signup + up to $100 earnable | Same |
| Duration | 6 months (then auto-closes if not upgraded) | 6 months of credits, then pay-as-you-go |
| On credit exhaustion | Account suspended | Normal billing begins |
| Service access | Restricted (some large instances blocked) | Full |

**Always Free services (never expire):**

| Service | Monthly free allowance |
|---|---|
| Lambda | 1M requests + 400K GB-seconds |
| DynamoDB | 25 GB + 25 WCU + 25 RCU |
| CloudFront | 1 TB out + 10M requests |
| CloudWatch | 10 metrics, 10 alarms, 5 GB logs |

**Day-1 safety setup:**
1. Enable MFA on root user
2. Create IAM user with AdministratorAccess
3. Activate IAM access to Billing (root-only, one time)
4. Create Zero-Spend Budget ($0.01 alert)
5. Create $5 monthly forecasted budget
6. Enable Free Tier usage alerts

## Key terms — Module 01

| Term | Definition |
|---|---|
| Cloud | On-demand IT resources over the internet, pay-as-you-go |
| Region | Geographic cluster of AWS data centers (e.g., Frankfurt) |
| Availability Zone | 1+ isolated data centers within a Region, independent power/cooling |
| AWS Account | Container for resources, users, and billing (12-digit ID) |
| Root user | Original email-based identity; unrestricted; lock it away |
| IAM user | Named identity inside an account with policy-controlled permissions |
| Console | AWS web GUI at console.aws.amazon.com |
| Free Tier | Set of free allowances — credits + Always Free services |
| CapEx | Capital expenditure — buy upfront (on-prem) |
| OpEx | Operational expenditure — pay as you go (cloud) |

---

# Module 02 — Service models and basic architecture

## Service models: IaaS / PaaS / SaaS / FaaS

```
          You manage MORE ◄──────────────────► Provider manages MORE

┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ ON-PREM  │  │  IaaS    │  │  PaaS    │  │  FaaS    │  │  SaaS    │
├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤
│ App   YOU│  │ App   YOU│  │ App   YOU│  │ Func  YOU│  │          │
│ Data  YOU│  │ Data  YOU│  │ Data  YOU│  │ Data  YOU│  │ Data  YOU│
│ Runtime  │  │ Runtime  │  │          │  │          │  │          │
│ OS       │  │ OS    YOU│  │          │  │          │  │          │
│ Virtual  │  │          │  │          │  │          │  │          │
│ Server   │  │          │  │          │  │          │  │          │
│ Storage  │  │          │  │          │  │          │  │          │
│ Network  │  │          │  │          │  │          │  │          │
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
  All YOU       AWS below     AWS below     AWS below     AWS below
                  the OS      the runtime   the function  the app
```

**AWS examples for each model:**

| Model | You manage | AWS manages | AWS service examples |
|---|---|---|---|
| **IaaS** | OS, apps, data | Hardware, virtualization, networking | EC2, EBS, VPC |
| **PaaS** | Code and data | OS, runtime, platform | Elastic Beanstalk, RDS, App Runner |
| **FaaS** | Function code and data | Everything else | Lambda, Fargate |
| **SaaS** | Just use the app | Everything | WorkMail, Chime, Amazon Connect |

**Pizza as a Service (the classic analogy):**

| | Homemade (On-Prem) | Take & Bake (IaaS) | Delivery (PaaS) | Dining Out (SaaS) |
|---|---|---|---|---|
| Dining table | You | You | You | Restaurant |
| Oven + gas | You | You | Restaurant | Restaurant |
| Dough + toppings | You | Restaurant | Restaurant | Restaurant |
| Cooking | You | You | Restaurant | Restaurant |
| Serving | You | You | Restaurant | Restaurant |

**Layered reality:** Salesforce is SaaS to you — but Salesforce itself runs on AWS (IaaS). Service models stack.

## Shared Responsibility Model

```
┌─────────────────────────────────────────────────────────┐
│               CUSTOMER RESPONSIBILITY                   │
│         "Security IN the cloud"                         │
│                                                         │
│  Data · IAM · OS patches (EC2) · App code ·             │
│  Encryption choices · Network config (SGs, NACLs) ·     │
│  Client-side data protection                            │
├─────────────────────────────────────────────────────────┤
│                  AWS RESPONSIBILITY                     │
│         "Security OF the cloud"                         │
│                                                         │
│  Physical data centers · Hardware · Hypervisor ·        │
│  Networking infrastructure · Managed service platforms ·│
│  Global infrastructure · Edge locations                 │
└─────────────────────────────────────────────────────────┘
```

**How the split changes by service type:**

| Layer | EC2 (IaaS) | RDS (PaaS) | Lambda (FaaS) | S3 |
|---|---|---|---|---|
| Hardware | AWS | AWS | AWS | AWS |
| Hypervisor | AWS | AWS | AWS | AWS |
| Guest OS | **You** | AWS | AWS | AWS |
| DB engine patches | n/a | AWS | n/a | n/a |
| App code | **You** | n/a | **You** | n/a |
| Data | **You** | **You** | **You** | **You** |
| IAM / access | **You** | **You** | **You** | **You** |
| Encryption | **You** | **You** | **You** | **You** |

> **Remember:** data and access configuration are ALWAYS your responsibility, no matter the service model.

## High availability and fault tolerance

```
┌─────────────────── Region: eu-central-1 ───────────────────┐
│                                                            │
│   AZ-a                    AZ-b                    AZ-c     │
│  ┌───────┐               ┌───────┐              ┌──────┐   │
│  │ EC2   │               │ EC2   │              │ EC2  │   │
│  │ RDS   │◄─── sync ────►│ RDS   │              │      │   │
│  │primary│    replication│standby│              │      │   │
│  └───────┘               └───────┘              └──────┘   │
│                                                            │
│         AZ-a fails? → AZ-b takes over.                     │
│         This is Multi-AZ.                                  │
└────────────────────────────────────────────────────────────┘
```

| Concept | Definition | User experience on failure |
|---|---|---|
| **High Availability (HA)** | Fast recovery / failover | Brief blip (seconds to minutes) |
| **Fault Tolerance (FT)** | No interruption at all | Nothing visible — failure absorbed |

**Rule:** single AZ = dev/test. Multi-AZ = production. Always.

## SLAs and the nines

| Availability | Downtime / year | Downtime / month |
|---|---|---|
| 99% (2 nines) | 3.65 days | 7.3 hours |
| 99.9% (3 nines) | 8.77 hours | 44 minutes |
| 99.95% | 4.38 hours | 22 minutes |
| **99.99% (4 nines)** | **52.6 minutes** | **4.4 minutes** |
| 99.999% (5 nines) | 5.26 minutes | 26 seconds |

**Key AWS SLAs to know:**

| Service | SLA |
|---|---|
| EC2 (Multi-AZ, region-level) | 99.99% |
| EC2 (single instance) | 99.5% |
| RDS Multi-AZ | 99.95% |
| S3 Standard (availability) | 99.9% |
| S3 Standard (durability) | 99.999999999% (11 nines) |
| Route 53 | 100% |
| Lambda | 99.95% |
| DynamoDB | 99.99% |

> **Availability** = "is it responding right now?" · **Durability** = "is my data still there?"

## Scaling: vertical vs horizontal

```
Vertical (scale UP)              Horizontal (scale OUT)
┌────────┐                       ┌────┐ ┌────┐ ┌────┐
│        │                       │ EC2│ │ EC2│ │ EC2│
│ BIGGER │  ← stop, resize,      │    │ │    │ │    │
│  EC2   │    restart            └──┬─┘ └──┬─┘ └──┬─┘
│        │                          │      │      │
└────────┘                       ┌──┴──────┴──────┴──┐
                                 │  Load Balancer    │
 • Requires downtime             └───────────────────┘
 • Has a ceiling
 • Simple                        • No downtime to add
                                 • Near-unlimited scale
                                 • App must be stateless
```

**Auto Scaling Group (ASG):** manages a fleet of EC2 instances across multiple AZs. Three numbers define it:

| Setting | Meaning |
|---|---|
| **Minimum** | Never fewer than this (e.g., 2) |
| **Desired** | Target right now (e.g., 4) |
| **Maximum** | Never more than this (e.g., 20) |

CloudWatch metrics trigger scaling policies → ASG adds or removes instances → ALB distributes traffic.

## 3-tier reference architecture on AWS

```
                     Users
                       │
                       ▼
              ┌────────────────┐
              │   Route 53     │  DNS (100% SLA)
              └───────┬────────┘
                      ▼
              ┌────────────────┐
              │  CloudFront    │  CDN + WAF + Shield
              └───────┬────────┘
                      ▼
    ┌─────────────── VPC ──────────────────┐
    │                                      │
    │  ┌─────────┐        ┌─────────┐      │
    │  │ Public  │        │ Public  │      │   ← ALB nodes
    │  │ subnet  │◄──────►│ subnet  │      │
    │  │  AZ-a   │        │  AZ-b   │      │
    │  └────┬────┘        └────┬────┘      │
    │       ▼                  ▼           │
    │  ┌─────────┐        ┌─────────┐      │
    │  │ Private │        │ Private │      │   ← EC2 / ECS
    │  │ subnet  │        │ subnet  │      │     (Auto Scaling)
    │  │  AZ-a   │        │  AZ-b   │      │
    │  └────┬────┘        └────┬────┘      │
    │       ▼                  ▼           │
    │  ┌─────────┐  sync  ┌─────────┐      │
    │  │  RDS    │◄──────►│  RDS    │      │   ← Multi-AZ DB
    │  │ primary │        │ standby │      │
    │  └─────────┘        └─────────┘      │
    │                                      │
    └──────────────────────────────────────┘
                      │
              ┌───────┴────────┐
              │  S3 (assets,   │  Object storage
              │  backups, logs)│  (11 nines durability)
              └────────────────┘
```

**Request flow (simplified):**
1. User → Route 53 resolves DNS
2. → CloudFront edge serves cached content (fast, cheap)
3. → Cache miss → ALB in VPC distributes to healthy EC2
4. → EC2 checks cache (ElastiCache) → if miss, queries RDS
5. → Files read/written to S3 via VPC Endpoint (free, private)
6. → Response returns: EC2 → ALB → CloudFront (caches it) → user

**AZ failure scenario:** ALB routes away from failed AZ → ASG launches replacements → RDS fails over to standby → users see a brief blip → site stays up.

## Well-Architected Framework — six pillars

| Pillar | One-liner | Mnemonic letter |
|---|---|---|
| **Operational Excellence** | Deploy, observe, recover with confidence | **O** |
| **Security** | Right people, right access, right things | **S** |
| **Reliability** | Survive failure, keep working | **R** |
| **Performance Efficiency** | Right tool, right size, evolve with tech | **P** |
| **Cost Optimization** | Pay for what you use, not what you might | **C** |
| **Sustainability** | Minimize environmental impact | **S** |

> **Mnemonic:** "**O**ur **S**ystems **R**un **P**retty **C**ost-effectively and **S**ustainably"

## Key terms — Module 02

| Term | Definition |
|---|---|
| IaaS | You manage OS and up; provider manages hardware (EC2) |
| PaaS | You manage code and data; provider manages platform (Beanstalk, RDS) |
| SaaS | You just use the app; provider manages everything (WorkMail) |
| FaaS | Upload a function; runs on events; pay per invocation (Lambda) |
| Multi-AZ | Deploy across ≥2 AZs for high availability |
| SLA | Contractual uptime commitment; breach = service credits |
| Availability | Is the service reachable right now? |
| Durability | Is my data still there, uncorrupted? |
| HA | High availability — minimizes downtime (brief blip OK) |
| FT | Fault tolerance — zero perceivable interruption |
| ASG | Auto Scaling Group — fleet of EC2s, self-healing, auto-scaling |
| ALB | Application Load Balancer — distributes HTTP traffic across targets |
| Shared Responsibility | AWS secures OF the cloud; you secure IN the cloud |
| Well-Architected | Six-pillar framework for cloud best practices |

---

*AWS Cloud Technologies Course · Modules 01–02 Student Reference*
