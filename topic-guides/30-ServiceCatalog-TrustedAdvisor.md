# Section 30: Service Catalog & Trusted Advisor

## The idea

This section is about **controlling what users can deploy** and **checking whether an AWS environment follows best practices**.

The key idea:

**AWS Service Catalog** = let organizations create and offer a **catalog of approved AWS products/resources** that users can deploy themselves.

**AWS Trusted Advisor** = analyze your AWS environment and give **recommendations** for improving cost, performance, security, fault tolerance, service limits, and operational excellence.

> **Important:** Trusted Advisor helps identify problems and recommends actions. It does not automatically fix the problem.

---

# AWS Service Catalog

**AWS Service Catalog = a company-approved catalog of AWS resources/products.**

Instead of allowing users to create arbitrary infrastructure, administrators create a set of **approved products** that users can launch themselves.

```text
Company
  ↓
Service Catalog
  ├── Approved EC2 environment
  ├── Approved RDS database
  ├── Approved VPC
  └── Other approved products
```

This gives the organization:

* **Standardization** → users deploy approved configurations
* **Governance** → central teams control what can be deployed
* **Self-service** → developers/users can launch approved products without building everything from scratch

## Core concepts

**Service Catalog** — the company controls *what can be deployed*, while developers/users can deploy approved configurations without building them from scratch.

**Products** are approved IT resources/templates made available to users.

A product can be based on an **AWS CloudFormation template**.

**Portfolios** are collections of products that can be shared with users, groups, or accounts.

```text
Portfolio
  ├── Product A
  ├── Product B
  └── Product C
```

### Easy memory

> **Service Catalog = company-approved AWS resource catalog**

Think:

> **"AWS App Store for company-approved infrastructure."**

---

# Trusted Advisor

**AWS Trusted Advisor = checks your AWS environment and recommends improvements.**

It looks for common AWS best-practice issues involving:

* Cost
* Performance
* Security
* Fault tolerance
* Service limits / quotas
* Operational excellence

---

# Trusted Advisor Categories

The **classic five categories** are:

| Category              | Example checks                                                                  |
| --------------------- | ------------------------------------------------------------------------------- |
| **Cost Optimization** | Idle EC2 instances, unattached Elastic IPs, underused resources                 |
| **Performance**       | Overutilized instances, high-latency configurations                             |
| **Security**          | Security groups open to `0.0.0.0/0`, **MFA missing on root**, public S3 buckets |
| **Fault Tolerance**   | Missing Multi-AZ, snapshots not taken, single-AZ exposure                       |
| **Service Limits**    | You're approaching a **service quota**                                          |

Current Trusted Advisor documentation also lists **Operational Excellence** as a category.

So for current AWS knowledge:

```text
Trusted Advisor
├── Cost Optimization
├── Performance
├── Security
├── Fault Tolerance
├── Service Limits
└── Operational Excellence
```

For older SAA material, you will often still see the **five classic categories**.

---

# Trusted Advisor — Service Limits / Service Quotas

This is especially important for SAA.

**Service limits = service quotas.**

Trusted Advisor's **Service Limits** checks monitor usage against supported AWS service quotas.

The check reports:

* **Service**
* **Region**
* **Limit Amount**
* **Current Usage**
* **Status**

For Trusted Advisor Service Limits checks:

```text
Yellow
= 80% of quota reached

Red
= 100% of quota reached
```

The data is based on a snapshot, and Trusted Advisor documentation says quota/usage data can take **up to 24 hours** to reflect changes.

### Example

Suppose an AWS quota is:

```text
VPC limit = 100
Current usage = 82
```

Trusted Advisor can show:

```text
82 / 100
= 82%
= Yellow
```

### Signal

> **Monitor AWS resource usage against supported service quotas → Trusted Advisor Service Limits**

---

# Trusted Advisor vs Service Quotas

This distinction is important.

## Service Quotas

**Service Quotas = view/manage quotas.**

Use it to:

* View quotas
* Check whether a quota is adjustable
* Request quota increases
* Manage supported quotas

```text
Service Quotas
= manage the quota itself
```

## Trusted Advisor Service Limits

**Trusted Advisor = monitor usage against supported quotas and recommend action.**

```text
Trusted Advisor
= "You are getting close to the quota."
```

### Easy memory

```text
Need to MANAGE / INCREASE the quota
→ Service Quotas

Need to MONITOR USAGE against the quota
→ Trusted Advisor Service Limits
```

---

# Trusted Advisor quota alerts

Trusted Advisor Service Limits checks use:

```text
80%
→ Yellow

100%
→ Red
```

AWS recommends requesting an increase from the **Service Quotas console** when you expect to exceed a quota.

---

# Trusted Advisor and notifications

Trusted Advisor itself is primarily a **recommendation/checking service**.

For automation and notifications, it can integrate with services such as:

```text
Trusted Advisor
      ↓
EventBridge / CloudWatch
      ↓
SNS / other target
      ↓
Notification
```

Trusted Advisor check status can be monitored through Amazon EventBridge, and Trusted Advisor also exposes metrics such as `RedResources`, `YellowResources`, and the service-quota metric `ServiceLimitUsage`.

### Memory

> **Trusted Advisor recommends; other services can alert/automate around the result.**

---

# Trusted Advisor Support Plans

The availability of Trusted Advisor checks depends on the AWS support plan.

Current AWS documentation states:

* **Basic / Developer Support** → all **Service Limits** checks plus selected Security and Fault Tolerance checks
* **Business Support+ / Enterprise Support / Unified Operations** → full Trusted Advisor checks and programmatic access through the Trusted Advisor API

For SAA-style questions, the classic pattern remains:

> **Full Trusted Advisor checks, especially full Cost Optimization coverage → higher support plan**

### Important

Do **not** think:

> "No cost optimization checks = IAM permission problem."

It may simply be the **support plan**.

---

# Trusted Advisor vs Config vs Inspector

These services are easy to confuse.

## Trusted Advisor

```text
"Are there AWS best-practice improvements I should make?"
```

Broad recommendations across AWS.

Examples:

* idle resources
* security weaknesses
* fault-tolerance issues
* service quota usage

---

## AWS Config

```text
"Does this resource comply with MY rules?"
```

Examples:

* S3 bucket must not be public
* EBS volumes must be encrypted
* Security group must not allow SSH from the internet

> **Config = compliance against rules**

---

## Amazon Inspector

```text
"Does my workload have software vulnerabilities?"
```

Examples:

* package vulnerabilities
* software vulnerabilities
* EC2/container/Lambda vulnerability findings

> **Inspector = workload vulnerability scanning**

---

# Trusted Advisor vs CloudWatch

Another useful distinction:

### Trusted Advisor

Checks AWS best-practice conditions.

```text
Idle EC2?
Quota nearly exhausted?
Security best practice violated?
```

→ **Trusted Advisor**

### CloudWatch

Monitors application/infrastructure metrics.

```text
CPU = 90%
Latency = 500 ms
Requests = 10,000/min
```

→ **CloudWatch**

---

# Trusted Advisor vs Service Quotas Automatic Management

AWS now provides a newer feature called:

# Service Quotas Automatic Management

This is important for **modern AWS questions**.

**Service Quotas Automatic Management** monitors supported service-quota utilization and can notify you before quotas are exhausted. AWS currently provides thresholds at:

```text
80%
95%
```

It has two modes:

* **Notify Only**
* **Notify and Auto-Adjust**

---

## Notify Only

The service monitors supported quotas and sends notifications.

```text
Quota usage
   ↓
80%
   ↓
Notification

95%
   ↓
Notification
```

Notifications appear through AWS Health, with optional notification channels such as email/chat/app notifications. EventBridge integration is also available for automation.

---

## Notify and Auto-Adjust

In addition to notifying you, AWS can automatically submit quota increase requests for supported adjustable quotas.

```text
Quota usage
   ↓
Threshold exceeded
   ↓
AWS automatically requests quota increase
```

Important:

> **Auto-adjust does not guarantee approval.**

It only automates the quota increase request for supported quotas.

---

# Service Quotas Automatic Management thresholds

| Utilization | Automatic Management                                                   |
| ----------- | ---------------------------------------------------------------------- |
| **80%**     | Notify                                                                 |
| **95%**     | Notify                                                                 |
| **95%+**    | Notify and, in Auto-Adjust mode, request increase for supported quotas |

There is an important mode difference:

| Mode                       | 80% notification | 95% notification | Auto quota increase |
| -------------------------- | ---------------: | ---------------: | ------------------: |
| **Notify Only**            |                ✅ |                ✅ |                   ❌ |
| **Notify and Auto-Adjust** |                ❌ |                ✅ |                   ✅ |

AWS documents that the exact automatic-adjustment behavior depends on the quota supporting automated adjustment.

---

# Important: Automatic Management vs Trusted Advisor

These can both appear in questions about quotas.

## Trusted Advisor

```text
Monitor supported quota usage
→ Service Limits check
→ Yellow at 80%
→ Red at 100%
```

The information is snapshot-based and can take up to 24 hours to reflect changes.

## Service Quotas Automatic Management

```text
Monitor supported quota usage
→ 80% / 95%
→ notifications
→ optional automatic quota increase requests
```

It is specifically designed to proactively monitor supported quotas and notify you before you run out.

### Modern exam memory

```text
Need broad AWS best-practice checks
→ Trusted Advisor

Need dedicated modern quota monitoring/automatic management
→ Service Quotas Automatic Management
```

---

# Important limitation of Service Quotas Automatic Management

It does **not** automatically cover every AWS quota.

Only quotas with the required **usage metrics** are supported for Automatic Management.

It can monitor both:

* adjustable quotas
* non-adjustable quotas

But only a subset of adjustable quotas support **automatic adjustment**.

So:

> **Supported quotas only → Automatic Management**

---

# Trusted Advisor vs Service Quotas Automatic Management

| Requirement                             | Service                                 |
| --------------------------------------- | --------------------------------------- |
| Broad AWS best-practice recommendations | **Trusted Advisor**                     |
| Cost optimization recommendations       | **Trusted Advisor**                     |
| Security best-practice checks           | **Trusted Advisor**                     |
| Fault-tolerance checks                  | **Trusted Advisor**                     |
| Service-limit/quota checks              | **Trusted Advisor Service Limits**      |
| Dedicated quota monitoring              | **Service Quotas Automatic Management** |
| 80% / 95% quota notifications           | **Service Quotas Automatic Management** |
| Automatically request quota increases   | **Service Quotas Automatic Management** |
| Manually request quota increase         | **Service Quotas**                      |

---

# Service Catalog vs Trusted Advisor

Do not confuse these.

```text
Service Catalog
= control what users CAN deploy

Trusted Advisor
= analyze what you HAVE deployed
```

Example:

```text
Service Catalog
→ only allow approved RDS/EC2/VPC products

Trusted Advisor
→ identify idle resources / security issues / quota problems
```

---

# Question Patterns

> *"Non-technical teams must deploy infrastructure, but only pre-approved configurations, without granting them broad IAM permissions."*

→ **Service Catalog**

Think:

```text
vending machine
→ approved products
→ self-service deployment
```

---

> *"Identify idle instances and unattached Elastic IPs to reduce the bill."*

→ **Trusted Advisor — Cost Optimization**

---

> *"Team can only see a few Trusted Advisor checks — why?"*

→ **Support plan**

Full Trusted Advisor access requires a higher support plan.

---

> *"Get warned before hitting an AWS service quota."*

For traditional SAA questions:

→ **Trusted Advisor Service Limits**

For modern AWS quota-management functionality:

→ **Service Quotas Automatic Management**

---

> *"Flag security groups allowing unrestricted access and missing root MFA."*

→ **Trusted Advisor — Security**

---

> *"Central IT curates standard CloudFormation-based products for business units."*

→ **Service Catalog portfolios**

Products live in portfolios and can be shared with users/accounts.

---

> *"Automatically monitor supported quotas and notify administrators at 80% and 95%."*

→ **Service Quotas Automatic Management**

---

> *"Automatically request quota increases when supported quotas approach their thresholds."*

→ **Service Quotas Automatic Management — Notify and Auto-Adjust**

---

# Pocket Card

| Keyword                                     | Answer                                                           |
| ------------------------------------------- | ---------------------------------------------------------------- |
| Self-service approved stacks, minimal IAM   | **Service Catalog**                                              |
| Product / portfolio                         | **Service Catalog terms**                                        |
| Product based on CloudFormation             | **Service Catalog**                                              |
| Idle / unused / unattached resources        | **Trusted Advisor — Cost**                                       |
| Approaching quota warning                   | **Trusted Advisor — Service Limits**                             |
| Full Trusted Advisor checks                 | **Higher support plan**                                          |
| Open SGs, root MFA check                    | **Trusted Advisor — Security**                                   |
| Five classic Trusted Advisor pillars        | **Cost, Performance, Security, Fault Tolerance, Service Limits** |
| Current additional Trusted Advisor category | **Operational Excellence**                                       |
| Dedicated quota monitoring                  | **Service Quotas Automatic Management**                          |
| 80% / 95% quota notifications               | **Service Quotas Automatic Management**                          |
| Automatic quota increase requests           | **Service Quotas Automatic Management**                          |
| Manually request quota increase             | **Service Quotas**                                               |

---

# Final Memory

```text
Service Catalog
= WHAT USERS CAN DEPLOY

Trusted Advisor
= WHAT COULD BE IMPROVED

Service Quotas
= VIEW / MANAGE QUOTAS

Service Quotas Automatic Management
= MONITOR QUOTA USAGE
= 80% / 95% NOTIFICATIONS
= OPTIONAL AUTO-ADJUST
```

### Trusted Advisor

```text
Cost
Performance
Security
Fault Tolerance
Service Limits
Operational Excellence
```

### Service Quotas Automatic Management

```text
Supported quota
      ↓
Monitor utilization
      ↓
80% / 95%
      ↓
Notify
      ↓
Optional Auto-Adjust
```

# The Golden Rule

```text
Approved infrastructure catalog
→ Service Catalog

AWS best-practice recommendations
→ Trusted Advisor

Cost optimization recommendations
→ Trusted Advisor

Security best-practice checks
→ Trusted Advisor

Service quota / service-limit checks
→ Trusted Advisor

Dedicated modern quota monitoring
→ Service Quotas Automatic Management

80% / 95% quota notifications
→ Service Quotas Automatic Management

Automatic quota increase requests
→ Service Quotas Automatic Management

View / request quota increases manually
→ Service Quotas
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

```text
Approved products      → Service Catalog
Idle resources         → Trusted Advisor
Security checks        → Trusted Advisor
Service limits         → Trusted Advisor
Quota management       → Service Quotas
80% / 95% monitoring   → Automatic Management
Auto quota adjustment  → Automatic Management
```
