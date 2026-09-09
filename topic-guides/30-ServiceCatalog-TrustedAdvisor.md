# Section 30: Service Catalog & Trusted Advisor

## The idea

This section is about **controlling what users can deploy** and **checking whether an AWS environment follows best practices**.

The key idea:

**AWS Service Catalog** = let organizations create and offer a **catalog of approved AWS products/resources** that users can deploy themselves.

**AWS Trusted Advisor** = analyze your AWS environment and give **recommendations** for improving cost, performance, security, fault tolerance, and service limits.

## Core concepts

**Service Catalog** — the company controls *what can be deployed*, while developers/users can deploy approved configurations without building them from scratch.

**Products** are approved IT resources/templates made available to users. A product can be based on an **AWS CloudFormation template**.


## Trusted Advisor's five pillars

| Pillar | Example checks |
|---|---|
| **Cost Optimization** | Idle EC2 instances, unattached Elastic IPs, underused RDS |
| **Performance** | Overutilized instances, high-latency configs |
| **Security** | Security groups open to 0.0.0.0/0, **MFA missing on root**, public S3 buckets |
| **Fault Tolerance** | Missing Multi-AZ, snapshots not taken, single-AZ exposure |
| **Service Limits** | You're approaching a **service quota** (e.g., near your VPC or EC2 limit) |

THE trap (tested nuance): **the free tier of Trusted Advisor gives only a handful of core checks** (basic security + service limits). To unlock the **full set of checks — including all Cost Optimization checks — you need a Business or Enterprise support plan.** If a question asks "why can't the team see the cost optimization checks?", the answer is their support plan, not a permissions problem.

Also worth one neuron each:
- Trusted Advisor **recommends**; it doesn't fix. Pair it with EventBridge/CloudWatch for alerting on check status changes.
- Distinguish from neighbors: **Config** = "is this resource *compliant with my rules*?"; **Inspector** = software vulnerabilities on workloads; **Trusted Advisor** = broad best-practice checks across the five pillars, including cost and limits.

## Question patterns

> *"Non-technical teams must deploy infrastructure, but only pre-approved configurations, without granting them broad IAM permissions"* → **Service Catalog** (vending machine: products in portfolios, launch-on-behalf)

> *"Identify idle instances and unattached Elastic IPs to reduce the bill"* → **Trusted Advisor** (Cost Optimization pillar)

> *"Team can only see a few Trusted Advisor checks — why?"* → **Support plan** (full checks need Business/Enterprise)

> *"Get warned before hitting an AWS service quota"* → **Trusted Advisor Service Limits check** (limits = its own pillar)

> *"Flag security groups allowing unrestricted access and missing root MFA"* → **Trusted Advisor** (Security pillar)

> *"Central IT curates standard CloudFormation-based products for business units"* → **Service Catalog portfolios** (products live in portfolios, shared to users/accounts)

## Pocket card

| Keyword | Answer |
|---|---|
| Self-service approved stacks, minimal IAM | Service Catalog |
| Product / portfolio | Service Catalog terms (CFN template / group of products) |
| Idle / unused / unattached resources | Trusted Advisor (Cost) |
| Approaching quota warning | Trusted Advisor (Service Limits) |
| Open SGs, root MFA check | Trusted Advisor (Security) |
| Full checks locked | Business/Enterprise support plan |
| Five pillars | Cost, Performance, Security, Fault Tolerance, Service Limits |

Trusted Advisor tells you the bill *could* be smaller — the cost management tool zoo in the next section is how you actually see, alert on, and shrink it.
