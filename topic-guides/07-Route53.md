# Section 7: Route 53

## The idea

**Amazon Route 53** is AWS's managed **DNS (Domain Name System)** service.

DNS translates a domain name into information such as an IP address or an AWS resource.

```text
User
  ↓
www.example.com
  ↓
Route 53
  ↓
IP address / AWS resource
  ↓
User connects to the destination
```

The most important thing to understand:

> **Route 53 chooses where the client should connect. It does not carry the application's traffic.**

Compare:

```text
Route 53
→ DNS
→ decides where to connect

ALB / NLB
→ load balancing
→ sits in the traffic path
→ distributes requests to targets
```

A common architecture is:

```text
User
  ↓
Route 53
  ↓
ALB
  ↓
EC2 instances
```

Route 53 chooses the destination. The load balancer then distributes traffic among instances.

---

# DNS records

The most important SAA record types are:

| Record    | Maps                          | Important point                 |
| --------- | ----------------------------- | ------------------------------- |
| **A**     | Name → IPv4 address           | Standard IPv4 DNS record        |
| **AAAA**  | Name → IPv6 address           | IPv6                            |
| **CNAME** | Name → another DNS name       | Cannot be used at the root/apex |
| **Alias** | Name → supported AWS resource | Can be used at the root/apex    |

## A record

Example:

```text
example.com
    ↓
192.0.2.10
```

Use an A record when you have an IPv4 address.

## AAAA record

Example:

```text
example.com
    ↓
2001:db8::1
```

Use an AAAA record for IPv6.

## CNAME

A CNAME points one DNS name to another DNS name.

```text
www.example.com
       ↓
app.example.com
```

A CNAME **cannot be used at the DNS zone apex**:

```text
example.com
```

### Exam signal

> "Point the root domain to an AWS resource."

→ **Alias**, not CNAME.

---

# Route 53 Alias records

An Alias record is an AWS-specific DNS feature that can point a domain name to supported AWS resources.

Common examples:

* Application Load Balancer
* Network Load Balancer
* CloudFront
* API Gateway
* S3 static website endpoint

Alias records have two important SAA advantages:

* Can be used at the **root/apex domain**
* No Route 53 charge for Alias queries

### Example

```text
example.com
      ↓
Alias
      ↓
ALB
```

### Exam signal

> "Point `example.com` directly to an ALB."

→ **Alias record**

Do not use CNAME at the apex.

---

# Hosted zones

A **hosted zone** contains the DNS records for a domain.

There are two main types.

## Public hosted zone

Answers DNS queries from the public Internet.

```text
Internet users
      ↓
Public hosted zone
      ↓
DNS records
```

### Exam signal

> "Public website must resolve on the Internet."

→ **Public hosted zone**

## Private hosted zone

Answers DNS queries only from associated VPCs.

Example:

```text
db.internal.example.com
        ↓
Private hosted zone
        ↓
Private IP
```

### Exam signal

> "Private DNS names should only resolve inside the VPC."

→ **Private hosted zone**

---

# Routing policies

Route 53 routing policies determine **which record is returned to the client**.

This is one of the most important SAA Route 53 topics.

| Routing policy         | Main purpose                                              | Key signal                    |
| ---------------------- | --------------------------------------------------------- | ----------------------------- |
| **Simple**             | Basic DNS response                                        | One normal answer             |
| **Weighted**           | Split traffic by percentage                               | A/B testing, canary           |
| **Latency**            | Send users to the lowest-latency Region                   | Best performance              |
| **Failover**           | Primary + secondary                                       | Active-passive DR             |
| **Geolocation**        | Route based on user location                              | Country/continent rules       |
| **Geoproximity**       | Route based on resource/user geographic distance and bias | Shift traffic using **bias**  |
| **Multi-Value Answer** | Return multiple healthy records                           | Simple DNS-level distribution |

---

# Simple routing

Simple routing returns a single record.

It is the basic/default routing behavior.

It does not provide traffic percentage splitting or active-passive failover by itself.

### Exam signal

> "A normal DNS record with no special routing requirement."

→ **Simple routing**

---

# Weighted routing

Weighted routing divides traffic according to percentages.

Example:

```text
90% → Version A
10% → Version B
```

This is useful for:

* A/B testing
* Canary deployments
* Gradually introducing a new version

### Exam signal

> "Send 10% of traffic to the new application."

→ **Weighted routing**

Think:

**Percentage → Weight**

---

# Latency-based routing

Latency routing sends the user to the Region that Route 53 determines will provide the **lowest latency**.

Example:

```text
User in Europe
      ↓
Lowest latency Region
      ↓
eu-west-1

User in Asia
      ↓
Lowest latency Region
      ↓
ap-southeast-1
```

The decision is about **performance**, not geographic rules.

### Exam signal

> "Global users should be sent to the Region with the best performance."

→ **Latency routing**

### Important distinction

A user in Germany does **not necessarily** go to a German/European Region.

Route 53 chooses based on latency.

---

# Failover routing

Failover routing provides a **primary and secondary** setup.

Typical use:

```text
Primary Region
      ↓
Healthy?
      │
   Yes → Primary
      │
   No
      ↓
Secondary / DR Region
```

A Route 53 health check determines whether the primary is healthy.

### Exam signal

* Active-passive
* Primary + secondary
* Disaster recovery
* Use standby only if primary fails

→ **Failover routing**

### Important

Route 53 failover is still DNS-based.

Clients and DNS resolvers may cache answers according to TTL, so failover is not necessarily instantaneous.

---

# Geolocation routing

Geolocation routing makes the decision based on **where the user is located**.

Example:

```text
Germany users
     ↓
German application

US users
     ↓
US application
```

Typical use cases:

* Legal requirements
* Content restrictions
* Language or regional content
* Country-specific applications

### Exam signal

> "Users in Germany must receive the German website."

→ **Geolocation routing**

This is a **location rule**, not a performance decision.

### Default record

Geolocation routing should have a **default record** for users whose location does not match one of the configured locations.

---

# Geoproximity routing

Geoproximity routing uses the geographic relationship between users and your AWS resources and lets you modify the traffic distribution using **bias**.

A positive or negative bias changes how much geographic area is associated with a resource.

### Exam signal

> "Gradually shift more traffic toward Region A."

> "Increase the traffic share using bias."

→ **Geoproximity routing**

### Remember

```text
Geolocation
→ Where is the USER?

Geoproximity
→ Geographic distance + BIAS
```

The word **bias** is one of the strongest exam clues for Geoproximity.

---

# Multi-Value Answer routing

Multi-Value Answer routing can return multiple healthy records in response to a DNS query.

Route 53 can return up to **8 healthy records**.

It can provide simple DNS-level distribution, but it is **not a replacement for an ELB**.

### Exam signal

> "Return several healthy IP addresses and let the client choose."

→ **Multi-Value Answer**

---

# Latency vs Geolocation vs Geoproximity

These three are easy to confuse.

| Policy           | Question being answered                                                      |
| ---------------- | ---------------------------------------------------------------------------- |
| **Latency**      | Which destination gives this user the best network performance?              |
| **Geolocation**  | Where is the user, and what rule applies to that location?                   |
| **Geoproximity** | How should traffic be distributed geographically, and how can bias shift it? |

### Easy memory

```text
Fastest
→ Latency

Country / continent rule
→ Geolocation

Bias / shift geographic traffic
→ Geoproximity
```

---

# Health checks

Route 53 health checks can monitor endpoints and influence routing decisions.

For example:

```text
Route 53
   ↓
Health check
   ↓
Application endpoint
```

If the endpoint becomes unhealthy, a routing policy such as **Failover** can stop returning it.

## Important limitation

Route 53 health checkers operate from the public AWS/Internet infrastructure.

They cannot directly check a normal **private IP endpoint inside a VPC**.

For a private resource:

```text
Private resource
      ↓
CloudWatch metric/alarm
      ↓
Route 53 health check based on alarm
```

### Exam signal

> "Route 53 must health-check a private resource."

→ **CloudWatch alarm-based health check**

---

# Calculated health checks

A calculated health check combines other health checks using logic.

You can combine many child health checks and define whether the overall result should be healthy based on the configured logic.

Example idea:

```text
Check A ──┐
Check B ──┼──► Calculated health check
Check C ──┘
```

### Exam signal

> "Combine multiple Route 53 health checks."

→ **Calculated health check**

---

# Route 53 vs Load Balancer

This distinction is extremely important.

## Route 53

Works at the **DNS level**.

```text
User
 ↓
Route 53
 ↓
Choose destination
```

Typical decisions:

* Region
* DR site
* AWS resource
* Geographic destination

## Load Balancer

Works in the **traffic path**.

```text
User
 ↓
Load Balancer
 ↓
Instance A
Instance B
Instance C
```

Typical decisions:

* Which EC2 instance?
* Which container?
* Which target?

### Exam rule

```text
Choose between Regions / sites
→ Route 53

Choose between instances / targets
→ ELB
```

---

# Route 53 failover vs ELB failover

Consider where the failure occurs.

### Instance failure

```text
ALB
 ↓
EC2-A ❌
 ↓
EC2-B ✅
```

The load balancer stops sending traffic to the unhealthy target.

→ **ELB**

### Regional failure

```text
Route 53
 ├── Region A ❌
 └── Region B ✅
```

→ **Route 53 failover**

The important architectural idea:

```text
ELB
→ inside a Region

Route 53
→ can choose between Regions
```

---

# TTL

**TTL (Time To Live)** tells DNS resolvers how long they may cache a DNS answer.

Example:

```text
TTL = 60 seconds
```

A resolver can cache the answer for roughly 60 seconds before asking DNS for a new answer.

### Why TTL matters

Lower TTL:

* Faster changes/failover visibility
* More DNS queries

Higher TTL:

* More caching
* Fewer DNS queries
* Slower propagation of changes

### Exam signal

> "DNS changes need to be reflected quickly."

→ **Lower TTL**

Remember:

> Route 53 cannot force every client to immediately forget a cached DNS answer.

---

# Global Accelerator vs Route 53

Both can help global applications, but they work at different layers.

## Route 53

DNS-based.

```text
User
 ↓
DNS lookup
 ↓
Route 53
 ↓
Destination
```

The client then connects directly to the destination.

## Global Accelerator

Uses static anycast IP addresses and stays in the network path.

```text
User
 ↓
Global Accelerator
 ↓
AWS global network
 ↓
Healthy regional endpoint
```

This allows traffic to move to another healthy endpoint without waiting for normal DNS TTL behavior.

### Exam signal

> "Need very fast global failover and static IP addresses."

→ **Global Accelerator**

> "Need DNS-based routing between Regions."

→ **Route 53**

---

# Question patterns

> **"Point the root domain to an ALB."**
> → **Alias record**

> **"Point a subdomain to another DNS name."**
> → **CNAME**

> **"Send 10% of traffic to a new application version."**
> → **Weighted routing**

> **"Send global users to the Region with the lowest latency."**
> → **Latency routing**

> **"Users in Germany must receive the German application."**
> → **Geolocation routing**

> **"Gradually shift more geographic traffic toward a Region."**
> → **Geoproximity + bias**

> **"Primary Region should receive traffic unless it becomes unhealthy."**
> → **Failover routing + health check**

> **"Return multiple healthy IP addresses."**
> → **Multi-Value Answer**

> **"Private VPC endpoint must be health-checked by Route 53."**
> → **CloudWatch alarm-based health check**

> **"Combine several health checks."**
> → **Calculated health check**

> **"DNS changes need to propagate faster."**
> → **Lower TTL**

> **"Need fast global failover without waiting for DNS caching."**
> → **Global Accelerator**

> **"Private DNS names should resolve only inside a VPC."**
> → **Private hosted zone**

---

# Pocket card

| Keyword                              | Answer                                  |
| ------------------------------------ | --------------------------------------- |
| IPv4 DNS record                      | **A**                                   |
| IPv6 DNS record                      | **AAAA**                                |
| Name → another name                  | **CNAME**                               |
| Root domain → AWS resource           | **Alias**                               |
| Public DNS                           | **Public hosted zone**                  |
| Internal VPC-only DNS                | **Private hosted zone**                 |
| Percentage / 10% / A-B test          | **Weighted**                            |
| Lowest latency / best performance    | **Latency**                             |
| Primary + DR / active-passive        | **Failover**                            |
| User country / continent             | **Geolocation**                         |
| Bias / geographic traffic shift      | **Geoproximity**                        |
| Multiple healthy IPs                 | **Multi-Value Answer**                  |
| Private endpoint health check        | **CloudWatch alarm-based health check** |
| Combine health checks                | **Calculated health check**             |
| Faster DNS changes                   | **Lower TTL**                           |
| Choose between Regions               | **Route 53**                            |
| Choose between instances             | **ELB**                                 |
| Instant global failover / static IPs | **Global Accelerator**                  |

---

# Core mental model

When a Route 53 question appears, first ask **what decision needs to be made**.

```text
What domain record is needed?
        ↓
A / AAAA / CNAME / Alias

Who should receive the traffic?
        ↓
Routing policy

Percentage
        ↓
Weighted

Fastest Region
        ↓
Latency

Primary / DR
        ↓
Failover

User location
        ↓
Geolocation

Geographic traffic shift / bias
        ↓
Geoproximity

Several healthy addresses
        ↓
Multi-Value Answer
```

Then ask whether health matters:

```text
Need endpoint health
        ↓
Route 53 Health Check

Private endpoint
        ↓
CloudWatch alarm
        ↓
Route 53 health check
```

Finally, ask whether DNS itself is the right tool:

```text
DNS-based regional routing
        ↓
Route 53

Fast global failover + static IPs
        ↓
Global Accelerator
```

The most important SAA distinctions are:

**Alias = AWS resource + root domain.**

**Weighted = percentage.**

**Latency = fastest Region.**

**Geolocation = user location rule.**

**Geoproximity = geographic distribution + bias.**

**Failover = primary/secondary DR.**

**Multi-Value = multiple healthy answers.**

**Route 53 = DNS decision.**

**ELB = traffic distribution inside the destination.**

**Global Accelerator = fast global traffic routing without waiting for DNS caching.**
