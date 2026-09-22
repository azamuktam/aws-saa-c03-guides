# Section 7: Route 53

## The idea

**Amazon Route 53** is AWS's managed **DNS service**.

DNS translates domain names into information such as IP addresses or AWS resources.

```text
User
 ↓
www.example.com
 ↓
Route 53
 ↓
Destination
```

> **Route 53 chooses where the client connects; it does not carry application traffic.**

```text
Route 53 → DNS / chooses destination
ALB/NLB   → traffic path / distributes requests
```

Typical architecture:

```text
User → Route 53 → ALB → EC2
```

---

# DNS records

| Record    | Maps                          | Key point              |
| --------- | ----------------------------- | ---------------------- |
| **A**     | Name → IPv4                   | IPv4                   |
| **AAAA**  | Name → IPv6                   | IPv6                   |
| **CNAME** | Name → DNS name               | Cannot be at zone apex |
| **Alias** | Name → supported AWS resource | Can be at zone apex    |

### A / AAAA

```text
A     → 192.0.2.10
AAAA  → 2001:db8::1
```

### CNAME

```text
www.example.com
 ↓
app.example.com
```

Cannot be used for:

```text
example.com
```

### Alias

Points to supported AWS resources such as:

* ALB
* NLB
* CloudFront
* API Gateway
* S3 website endpoint

Advantages:

* Works at the **root/apex**
* No Route 53 charge for Alias queries

> Root domain → AWS resource → **Alias**

---

# Route 53 + S3 static website

Route 53 can point a domain to an **S3 static website endpoint** using an Alias record.

Standard setup:

```text
Domain: www.example.com
Bucket: www.example.com
```

Requirements:

* Domain registered
* Bucket configured for **static website hosting**
* Bucket name matches the domain for the standard website setup
* Bucket and hosted zone do not need to be in the same Region
* CORS is not required for Route 53 → S3 website routing
* MX records are for email, not website routing

```text
User
 ↓
Route 53
 ↓
Alias
 ↓
S3 website endpoint
```

### HTTPS

Native S3 website endpoints do not provide HTTPS.

For HTTPS:

```text
User
 ↓
Route 53
 ↓
CloudFront
 ↓
S3
```

CloudFront can use an ACM certificate.

---

# Hosted zones

## Public hosted zone

Answers public Internet DNS queries.

> Public website → **Public hosted zone**

## Private hosted zone

Answers DNS queries only from associated VPCs.

```text
db.internal.example.com
 ↓
Private hosted zone
 ↓
Private IP
```

> VPC-only private DNS → **Private hosted zone**

---

# Route 53 Resolver endpoints

Used when DNS resolution crosses the **AWS/on-premises boundary**.

## Inbound

**On-premises → AWS**

```text
On-prem DNS
 ↓
Inbound Resolver endpoint
 ↓
Route 53 Resolver
 ↓
Private hosted zone
```

> On-prem DNS needs to resolve AWS private DNS → **Inbound endpoint**

## Outbound

**AWS → on-premises**

```text
EC2
 ↓
Route 53 Resolver
 ↓
Outbound Resolver endpoint
 ↓
On-prem DNS
```

Uses a **Resolver forwarding rule**.

> AWS resources need to resolve on-prem DNS → **Outbound endpoint + forwarding rule**

### Memory

```text
Inbound  → DNS comes IN → On-prem → AWS
Outbound → DNS goes OUT → AWS → On-prem
```

Normal VPC DNS already uses Route 53 Resolver; endpoints are needed for cross-network DNS resolution.

---

# Routing policies

| Policy                 | Main purpose                   | Signal              |
| ---------------------- | ------------------------------ | ------------------- |
| **Simple**             | Basic DNS response             | Normal DNS          |
| **Weighted**           | Percentage split               | A/B, canary         |
| **Latency**            | Lowest-latency Region          | Performance         |
| **Failover**           | Primary + secondary            | Active-passive DR   |
| **Geolocation**        | User location                  | Country/continent   |
| **Geoproximity**       | Geographic distribution + bias | Bias                |
| **Multi-Value Answer** | Multiple healthy records       | Several healthy IPs |

---

# Active-Active vs Active-Passive

This is an important multi-Region distinction.

### Active-Active

Multiple resources/Regions **serve traffic simultaneously**.

Can be implemented with routing policies such as:

* Weighted
* Latency
* Geolocation
* Multi-Value Answer

Health checks can remove unhealthy resources.

```text
            Route 53
           /        \
      Region A    Region B
       ACTIVE      ACTIVE
```

> Multiple Regions serving traffic at the same time → **Active-Active**

### Active-Passive

One resource/Region is **primary**; another is **standby**.

```text
Route 53
   ↓
Primary
   X
   ↓
Secondary / DR
```

Usually implemented with **Failover routing + health checks**.

> Primary + standby / DR → **Active-Passive**

### Key distinction

```text
Active-Active
→ Multiple active resources

Active-Passive
→ Primary + standby
```

---

# Simple routing

Returns a basic DNS answer.

> No special routing requirement → **Simple**

---

# Weighted routing

Splits traffic by percentage.

Example:

```text
90% → Version A
10% → Version B
```

Useful for:

* A/B testing
* Canary deployments
* Gradual rollouts

> Percentage → **Weighted**

---

# Latency-based routing

Sends users to the Region Route 53 determines will provide the **lowest latency**.

> Global users + best performance → **Latency**

This is based on latency, not simply geographic location.

---

# Failover routing

Provides **primary + secondary** routing.

```text
Primary
  ↓
Healthy? → Yes → Primary
  ↓ No
Secondary
```

Uses health checks.

Typical use:

* Active-passive
* Disaster recovery
* Primary/standby

> Primary + DR → **Failover + health check**

DNS caching means failover is not necessarily instantaneous.

---

# Geolocation routing

Routes based on **user location**.

Example:

```text
Germany → German application
US      → US application
```

Useful for:

* Legal requirements
* Regional content
* Language
* Country-specific applications

A **default record** should handle unmatched locations.

> User country/continent → **Geolocation**

---

# Geoproximity routing

Uses geographic relationship between users and resources and supports **bias** to shift traffic.

> Geographic traffic shift + **bias** → **Geoproximity**

Memory:

```text
Geolocation  → Where is the USER?
Geoproximity → Geography + BIAS
```

---

# Multi-Value Answer routing

Returns multiple healthy records; Route 53 can return up to **8 healthy records**.

Useful for simple DNS-level distribution.

Not a replacement for ELB.

> Several healthy IP addresses → **Multi-Value Answer**

---

# Health checks

Route 53 health checks can influence routing decisions.

If a resource becomes unhealthy, a routing policy can stop returning it.

### Private resources

Route 53 health checkers cannot directly check a normal private IP endpoint inside a VPC.

Use:

```text
Private resource
 ↓
CloudWatch metric/alarm
 ↓
Route 53 health check
```

> Private resource health check → **CloudWatch alarm-based health check**

---

# Calculated health checks

Combines multiple health checks.

```text
Check A ─┐
Check B ─┼→ Calculated health check
Check C ─┘
```

> Combine multiple health checks → **Calculated health check**

---

# Route 53 vs Load Balancer

## Route 53

DNS-level decision:

```text
User
 ↓
Route 53
 ↓
Destination
```

Can choose:

* Region
* DR site
* AWS resource
* Geographic destination

## Load Balancer

Traffic-path decision:

```text
User
 ↓
ALB/NLB
 ↓
Targets
```

Chooses:

* EC2 instance
* Container
* Target

### Exam rule

```text
Choose between Regions/sites
→ Route 53

Choose between instances/targets
→ ELB
```

---

# Route 53 failover vs ELB failover

### Instance failure

```text
ALB
 ↓
EC2-A ❌
EC2-B ✅
```

→ **ELB**

### Regional failure

```text
Route 53
 ├── Region A ❌
 └── Region B ✅
```

→ **Route 53**

> **ELB → inside a Region**

> **Route 53 → can choose between Regions**

---

# TTL

**TTL (Time To Live)** controls how long DNS resolvers may cache an answer.

Example:

```text
TTL = 60 seconds
```

Lower TTL:

* Faster DNS changes/failover visibility
* More DNS queries

Higher TTL:

* More caching
* Fewer queries
* Slower change visibility

> Need faster DNS changes → **Lower TTL**

Route 53 cannot force clients to immediately discard cached answers.

---

# Global Accelerator vs Route 53

## Route 53

DNS-based:

```text
User
 ↓
DNS lookup
 ↓
Route 53
 ↓
Destination
```

## Global Accelerator

Uses static anycast IPs and stays in the traffic path:

```text
User
 ↓
Global Accelerator
 ↓
AWS global network
 ↓
Healthy regional endpoint
```

This avoids waiting for normal DNS TTL behavior when switching endpoints.

> Fast global failover + static IPs → **Global Accelerator**

> DNS-based regional routing → **Route 53**

---

# Question patterns

> **Root domain → ALB**
> → **Alias**

> **Subdomain → another DNS name**
> → **CNAME**

> **S3 static website + Route 53**
> → **Static website hosting + matching bucket/domain + Alias**

> **10% traffic to new version**
> → **Weighted**

> **Lowest-latency Region**
> → **Latency**

> **German users → German application**
> → **Geolocation**

> **Shift geographic traffic using bias**
> → **Geoproximity**

> **Primary Region unless unhealthy**
> → **Failover + health check**

> **Multiple healthy IP addresses**
> → **Multi-Value Answer**

> **Private endpoint health check**
> → **CloudWatch alarm-based health check**

> **Combine health checks**
> → **Calculated health check**

> **Faster DNS changes**
> → **Lower TTL**

> **Fast global failover without waiting for DNS caching**
> → **Global Accelerator**

> **Private DNS only inside VPC**
> → **Private hosted zone**

> **On-prem DNS → AWS private DNS**
> → **Inbound Resolver endpoint**

> **AWS → on-prem DNS**
> → **Outbound Resolver endpoint + forwarding rule**

> **Multiple Regions serving traffic simultaneously**
> → **Active-Active**

> **Primary + standby/DR**
> → **Active-Passive / Failover**

---

# Pocket card

| Keyword                          | Answer                                  |
| -------------------------------- | --------------------------------------- |
| IPv4                             | **A**                                   |
| IPv6                             | **AAAA**                                |
| Name → DNS name                  | **CNAME**                               |
| Root domain → AWS resource       | **Alias**                               |
| S3 website + Route 53            | **Matching bucket/domain + Alias**      |
| Public DNS                       | **Public hosted zone**                  |
| VPC-only DNS                     | **Private hosted zone**                 |
| On-prem → AWS DNS                | **Inbound Resolver endpoint**           |
| AWS → on-prem DNS                | **Outbound Resolver endpoint**          |
| Percentage                       | **Weighted**                            |
| Lowest latency                   | **Latency**                             |
| Primary + DR                     | **Failover**                            |
| User location                    | **Geolocation**                         |
| Geographic bias                  | **Geoproximity**                        |
| Multiple healthy IPs             | **Multi-Value Answer**                  |
| Private health check             | **CloudWatch alarm-based health check** |
| Combine health checks            | **Calculated health check**             |
| Faster DNS changes               | **Lower TTL**                           |
| Choose between Regions           | **Route 53**                            |
| Choose between instances         | **ELB**                                 |
| Fast global failover + static IP | **Global Accelerator**                  |
| Multiple active Regions          | **Active-Active**                       |
| Primary + standby                | **Active-Passive**                      |

---
