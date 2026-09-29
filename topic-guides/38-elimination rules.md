# AWS SAA-C03 — 20 Last-Minute Elimination Rules

Use these as **elimination rules**, not absolute "always choose X" rules.

The goal is to quickly eliminate answers that violate the scenario, then choose the simplest solution that satisfies the exact requirements.

---

## 1. LEAST operational overhead

**Eliminate:**

* EC2 that you must patch/manage
* Custom infrastructure-management scripts
* Manual failover
* Manual scaling
* Self-managed servers

**Think:**

> Managed services / serverless / AWS-managed operations

Do **not** memorize:

> `Least operational overhead = Lambda`

The real rule is:

> **Remove infrastructure management whenever the workload allows it.**

---

## 2. Cost-effective + unpredictable or variable workload

**Distrust:**

* Permanently provisioned capacity
* Large always-running EC2 fleet
* Overprovisioning for peak traffic

**Think:**

> Serverless / autoscaling / pay-per-use

---

## 3. Predictable, steady workload for years

**Distrust:**

* Pure On-Demand when a long-term commitment is clearly appropriate

**Think:**

> Savings Plans / Reserved Instances

The key is **steady and predictable usage**.

---

## 4. High availability

**Eliminate:**

* Single AZ
* Single EC2 instance
* Single database instance
* Manual recovery

**Think:**

> Multi-AZ + redundancy + automatic failover

---

## 5. Fault tolerant

**Eliminate:**

* Solutions requiring humans to recover the system
* Single points of failure
* Manual replacement

**Think:**

> Automatic failure detection + automatic recovery/failover

---

## 6. Scalable / elastic

**Eliminate:**

* Fixed server count
* Manual scaling
* Architectures that cannot react automatically to demand

**Think:**

> ASG / managed scaling / serverless

---

## 7. Static website

**Eliminate:**

* EC2 web servers
* Self-managed web infrastructure

**Think:**

> S3 + CloudFront

Especially when the requirement is simply to serve static files.

---

## 8. EC2 needs private access to S3

**Eliminate:**

* NAT Gateway when the only requirement is private S3 access
* Internet Gateway/public internet path

**Think:**

> **S3 Gateway VPC Endpoint**

---

## 9. Private access to an AWS service

**Eliminate:**

* Unnecessary public internet routing
* NAT Gateway when a suitable VPC endpoint solves the requirement

**Think:**

> VPC Endpoint

Then distinguish:

* **Gateway endpoint** → S3, DynamoDB
* **Interface endpoint** → most other AWS services

---

## 10. Shared filesystem across Linux EC2 instances

**Eliminate:**

* EBS as shared storage between multiple instances

**Think:**

> **EFS**

Remember:

> EBS = block storage for an instance
> EFS = shared file storage

---

## 11. Windows shared file system

**Eliminate:**

* EFS when Windows-specific SMB functionality is required

**Think:**

> **FSx for Windows File Server**

---

## 12. Relational data / SQL / joins / transactions

**Eliminate:**

* DynamoDB when the application genuinely needs relational capabilities

**Think:**

> RDS / Aurora

Look for:

* SQL
* joins
* relational schema
* complex queries
* transactions

---

## 13. Massive key-value scale / very low latency

**Eliminate:**

* RDS/Aurora when relational features are unnecessary

**Think:**

> DynamoDB

Typical clues:

* Key-value access
* Massive scale
* Very high request volume
* Single-digit millisecond performance
* Serverless database

---

## 14. Decouple producer and consumer

**Eliminate:**

* Direct synchronous communication
* Producer calling consumer directly

**Think:**

> **SQS**

Typical architecture:

```text
Producer → SQS → Consumer
```

The queue absorbs traffic spikes and allows asynchronous processing.

---

## 15. One message → many subscribers

**Eliminate:**

* A single consumer/queue when multiple independent subscribers need the message

**Think:**

> **SNS fan-out**

Typical architecture:

```text
             → SQS
Producer → SNS → Lambda
             → SQS
```

---

## 16. Event routing based on rules

**Eliminate:**

* Custom Lambda routing when AWS already provides event routing
* Complex SNS-only architecture when event rules/filtering are the requirement

**Think:**

> **EventBridge**

Typical clues:

* Event bus
* Event rules
* AWS service events
* SaaS events
* Route events to different targets

---

## 17. HTTP path/host routing

**Eliminate:**

* NLB when Layer 7 HTTP routing is explicitly required

**Think:**

> **ALB**

Typical clues:

* `/api/*`
* `/images/*`
* Host-based routing
* HTTP headers
* HTTP redirects

---

## 18. Static IPs required for the load balancer

**Eliminate:**

* ALB when fixed/static IP addresses are explicitly required

**Think:**

> **NLB**

NLB supports static IP addresses and can use Elastic IPs.

---

## 19. Global entry point + static anycast IP + traffic acceleration

**Eliminate:**

* Route 53 alone
* Regional load balancer alone

**Think:**

> **Global Accelerator**

Typical clues:

* Global users
* Static anycast IPs
* Faster path to regional applications
* Automatic traffic routing/failover

---

## 20. Who did what?

**Eliminate:**

* CloudWatch when the question asks who made an AWS API change
* AWS Config when the question is about API activity

**Think:**

> **CloudTrail**

Remember the three-way distinction:

| Service        | Main question                                          |
| -------------- | ------------------------------------------------------ |
| **CloudWatch** | How is it performing?                                  |
| **CloudTrail** | Who did what?                                          |
| **AWS Config** | What is the resource's configuration/compliance state? |

---

# High-Value Elimination Patterns

## Storage

Memorize:

> **Object → S3**
> **Block → EBS**
> **Shared file → EFS**
> **Windows file → FSx for Windows**
> **HPC filesystem → FSx for Lustre**

If an answer uses EBS as shared storage across multiple EC2 instances, be suspicious.

---

## Database

Memorize:

> **SQL / joins / relational transactions → Aurora/RDS**
> **Key-value / massive scale → DynamoDB**
> **Cache → ElastiCache**

Do not choose DynamoDB simply because the question says **high performance**.

---

## Networking

Memorize:

> **Layer 7 HTTP routing → ALB**
> **Layer 4 TCP/UDP + static IP → NLB**
> **Global static anycast IP → Global Accelerator**
> **DNS routing → Route 53**
> **Web application filtering → WAF**
> **DDoS protection → Shield**

These services can all appear in the same question as distractors.

---

## Security / Identity

### Users need AWS access

> **IAM Identity Center / federation**

### Roles must never exceed a maximum permission set

> **Permissions boundary**

### Organization-wide guardrail

> **SCP**

### Who created/deleted/changed something?

> **CloudTrail**

### What is the current resource configuration?

> **AWS Config**

### CPU / memory / disk / application metrics and logs

> **CloudWatch**

For EC2 memory and filesystem utilization:

> **CloudWatch Agent**

---

# The "Unnecessary Architecture" Rule

When two answers both technically work, eliminate the one that adds unnecessary components.

Example:

Requirement:

> Encrypt S3 objects at rest.

Do not automatically choose:

```text
S3
+ KMS
+ custom key policies
+ Lambda
+ key rotation logic
```

when a simpler AWS-managed encryption option satisfies the requirement.

Think:

> **Use the simplest architecture that fully satisfies the requirement.**

---

# 10-Second Exam Process

## Step 1 — Read the last sentence

Look for:

* LEAST
* MOST
* COST
* OPERATIONAL OVERHEAD
* HIGH AVAILABILITY
* FAULT TOLERANCE
* MINIMAL CHANGES
* MOST SECURE
* LOWEST LATENCY
* COST OPTIMIZATION

The last sentence often tells you **what to optimize for**.

---

## Step 2 — Find the hard requirement

Ask:

> What requirement would make an option impossible?

Examples:

* Static IP?
* Shared filesystem?
* SQL?
* Asynchronous processing?
* Global?
* Private?
* Multi-AZ?
* Minimal changes?
* Cheapest?

---

## Step 3 — Eliminate contradictions

Do not ask:

> "Which service do I remember?"

Ask:

> "Which answers clearly violate the requirement?"

---

## Step 4 — Remove unnecessary complexity

Among the survivors:

> Prefer the solution that satisfies the requirement with fewer unnecessary components and less management.

---

# Final Mental Model

When stuck between two answers, think:

```text
Does it satisfy the exact requirement?
        ↓
Does it introduce unnecessary management?
        ↓
Does it introduce unnecessary components?
        ↓
Is there a more AWS-managed solution?
        ↓
Does it meet the required scale / availability / security?
        ↓
Choose the remaining answer.
```

## One rule to remember above everything else

> **SAA questions usually aren't asking "What AWS service can do this?"**
>
> They are asking:
>
> **"Which architecture satisfies these exact requirements with the requested trade-off?"**
