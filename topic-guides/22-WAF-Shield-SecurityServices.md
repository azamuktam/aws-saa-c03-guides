# Section 22: WAF, Shield & the Security Service Zoo

## The idea

This section is mainly a **service-matching game**. Identify the service from the scenario:

* **WAF** → Layer 7 HTTP filtering
* **Shield** → DDoS protection
* **Firewall Manager** → centralized security policies across AWS Organizations
* **Network Firewall** → VPC-wide traffic inspection
* **GuardDuty / Macie / Inspector / Security Hub / Detective / Artifact** → detection, discovery, scanning, aggregation, investigation, and compliance reports

### WAF — the Layer 7 firewall

**AWS WAF (Web Application Firewall)** works at **Layer 7** and reads HTTP requests: URLs, headers, query strings, and body content.

| Rule type               | Purpose                                                                   |
| ----------------------- | ------------------------------------------------------------------------- |
| **SQL injection rules** | Detect SQLi such as `' OR 1=1 --`                                         |
| **XSS rules**           | Detect malicious Cross-Site Scripting such as `<script>` input            |
| **Rate-based rules**    | Limit requests **per IP** over 5 minutes                                  |
| **Geo-match**           | Allow/block by country                                                    |
| **IP sets**             | Explicit IP allow/deny lists                                              |
| **Managed rule groups** | Pre-built rule sets from AWS/vendors, e.g. **OWASP Top 10** core rule set |

**Exam gold:** *"rate limit requests per IP"* → **WAF rate-based rule**.

**Count mode:** counts matching requests instead of blocking them. Use it to test a rule against production traffic **without impacting users**.

**Where can WAF attach?**

| WAF can attach to  | WAF cannot attach to |
| ------------------ | -------------------- |
| CloudFront         | NLB                  |
| ALB                | EC2 directly         |
| API Gateway        | Route 53             |
| AppSync            |                      |
| Cognito User Pools |                      |

**Trap:** *"Attach WAF to a Network Load Balancer"* → impossible. **NLB is Layer 4**; WAF requires an L7 front end: **CloudFront, ALB, API Gateway, AppSync, or Cognito**.

### Shield — DDoS protection

**DDoS (Distributed Denial of Service)** uses many machines to flood a service and make it unavailable.

* **Shield Standard**

  * **Free and automatic**
  * Protects against common **Layer 3/4** attacks
  * Examples: **SYN floods, UDP reflection**
  * No activation required

* **Shield Advanced**

  * **~$3,000/month**, 1-year commitment
  * Adds **Layer 7 DDoS protection**
  * **24/7 DDoS Response Team (DRT)**
  * **Cost protection**: AWS refunds scaling charges caused by DDoS attacks

**Memory hook:**

* **L7 DDoS / expert help / attack-related cost protection** → **Shield Advanced**
* **Common DDoS protection at no cost** → **Shield Standard**

### Firewall Manager — centralized security policy management

Use **AWS Firewall Manager** when security policies must be applied across many AWS accounts/resources.

Example: 50 accounts in an **AWS Organization**, and every ALB—including resources/accounts added later—must receive the same WAF rules.

Firewall Manager centrally manages and automatically applies:

* **WAF rules**
* **Shield Advanced**
* **Security Group rules**
* **Network Firewall rules**

It can automatically apply policies to **new accounts and new resources**.

**Prerequisites:**

* **AWS Organizations**
* **AWS Config** enabled

**Trap:** *"Automatically apply security policies across accounts, including future accounts"* → **Firewall Manager**, not WAF alone.

### Network Firewall — VPC-wide traffic inspection

**AWS Network Firewall** is a managed firewall for an entire **VPC**.

* Inspects traffic at **Layers 3–7**
* Supports **intrusion prevention**
* Supports **domain filtering**
* Supports **stateful rules**
* Can filter **all traffic entering/leaving the VPC**

**Scenario:** *"Inspect all traffic in the VPC"* or *"Filter outbound traffic to specific domains for the entire VPC"* → **Network Firewall**

### The detection zoo — one-liners

| Service          | One-liner                                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **GuardDuty**    | **ML threat detection** using **CloudTrail, VPC Flow Logs, DNS logs**; no agents. Detects **cryptomining, unusual API calls, compromised credentials** |
| **Macie**        | **PII / sensitive-data discovery in S3**; uses ML to identify data such as credit cards and SSNs .                                                      |
| **Inspector**    | **Vulnerability scanner** for **CVEs** on **EC2 (via SSM agent), ECR container images, Lambda**                                                        |
| **Security Hub** | **Aggregation dashboard** for findings from security services + compliance standards such as **CIS and PCI**                                           |
| **Detective**    | **Post-finding investigation**; builds relationship graphs to help identify **root cause**                                                             |
| **Artifact**     | Download **AWS compliance reports** such as **SOC, PCI, ISO** for auditors                                                                             |

**Flow:**

```text
GuardDuty / Macie / Inspector
            │
         findings
            ▼
       Security Hub
            │
      "Why did this happen?"
            ▼
         Detective
        root cause
```

**Trap: GuardDuty vs Inspector**

* **GuardDuty** → detects suspicious **behavior/activity** from logs; threats happening now
* **Inspector** → scans software/resources for **vulnerabilities**; weaknesses that could be exploited
* **Threat** → GuardDuty
* **CVE** → Inspector

**Trap:** *"Macie for EC2 or RDS"* → no. **Macie is S3-only.**
 Identify sensitive data using **Amazon Macie** and create an Amazon EventBridge (Amazon CloudWatch Events) rule to capture the **SensitiveData** event type.
 Set up an Amazon SNS topic as the target for an Amazon EventBridge (Amazon CloudWatch Events) rule that sends notifications when the error occurs again.

## Question patterns

> *"Block SQL injection attacks against an application behind an ALB"* → **WAF on the ALB** using L7 rules.

> *"Limit each client IP to 2,000 requests per 5 minutes"* → **WAF rate-based rule**.

> *"Protection against common DDoS attacks at no additional cost"* → **Shield Standard** (free, automatic, L3/L4).

> *"Company suffered a large DDoS, wants expert support during attacks and refunds for attack-related scaling costs"* → **Shield Advanced** (DRT + cost protection, ~**$3k/month**).

> *"Ensure WAF rules are applied to all ALBs across 50 accounts, including future accounts"* → **Firewall Manager** (org-wide automatic application; requires **Organizations + Config**).

> *"Identify S3 buckets containing personally identifiable information"* → **Macie**.

> *"Alert when EC2 instances are used for cryptocurrency mining or credentials are compromised"* → **GuardDuty**.

> *"Continuously scan EC2 instances and container images for software vulnerabilities (CVEs)"* → **Inspector**.

> *"Single pane of glass for security findings across all accounts and services"* → **Security Hub**.

> *"After a GuardDuty finding, analyze and identify the root cause of the incident"* → **Detective**.

> *"Auditor requests AWS's SOC 2 / PCI compliance reports"* → **Artifact**.

## Pocket card

| Keyword                                                  | Answer                                        |
| -------------------------------------------------------- | --------------------------------------------- |
| SQLi / XSS / HTTP filtering                              | **WAF**                                       |
| Rate limit per IP                                        | **WAF rate-based rule**                       |
| Test rule without blocking                               | **WAF Count mode**                            |
| WAF attach points                                        | **CloudFront, ALB, API GW, AppSync, Cognito** |
| WAF cannot attach to                                     | **NLB, EC2 directly, Route 53**               |
| Free automatic DDoS (L3/L4)                              | **Shield Standard**                           |
| L7 DDoS / DRT / cost protection                          | **Shield Advanced (~$3k/mo)**                 |
| Security policies across org, automatic for new accounts | **Firewall Manager (Organizations + Config)** |
| Inspect ALL VPC traffic L3–L7                            | **Network Firewall**                          |
| Cryptomining / odd API calls / no agents                 | **GuardDuty**                                 |
| PII in S3                                                | **Macie**                                     |
| CVEs on EC2/ECR/Lambda                                   | **Inspector**                                 |
| One findings dashboard + compliance checks               | **Security Hub**                              |
| Root cause after a finding                               | **Detective**                                 |
| SOC/PCI/ISO reports for auditors                         | **Artifact**                                  |

Next: **Cognito** — user identity and authentication.
