# Section 37I: DNS, IPv6 & Edge Infrastructure

## The idea

These are smaller AWS networking and infrastructure services that usually appear as **specific-use-case questions**.

> **Requirement → unique keyword → service**

| Requirement / keyword                             | Answer                                  |
| ------------------------------------------------- | --------------------------------------- |
| IPv6 outbound-only Internet access                | **Egress-Only Internet Gateway**        |
| On-premises → AWS DNS queries                     | **Route 53 Resolver inbound endpoint**  |
| AWS → on-premises DNS queries                     | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your own data center        | **Outposts**                            |
| Very low latency for a specific metropolitan area | **Local Zones**                         |
| 5G / mobile edge                                  | **Wavelength**                          |

---

# Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

It allows IPv6 instances to initiate outbound Internet connections while preventing **unsolicited inbound connections**.

### Signal

> **IPv6 + outbound Internet + block unsolicited inbound**

→ **Egress-Only Internet Gateway**

### Mental model

```text
IPv6 instance
      ↓
Egress-Only Internet Gateway
      ↓
Internet
```

Allowed:

```text
EC2 → Internet
```

Not allowed:

```text
Internet → EC2
```

### Important distinction

**Internet Gateway (IGW)** = normal Internet connectivity.

**Egress-Only Internet Gateway** = specifically for **IPv6 outbound-only access**.

### Memory

> **Egress-Only IGW = IPv6 OUTBOUND ONLY**

---

# Route 53 Resolver

Route 53 Resolver endpoints provide DNS resolution between AWS and external/on-premises DNS environments.

```text
On-premises → AWS DNS
→ Inbound endpoint

AWS → On-premises DNS
→ Outbound endpoint
```

The direction is based on **where the DNS query starts**.

---

# Route 53 Resolver Inbound Endpoint

Allows DNS queries from **on-premises into AWS**.

### Signal

> **On-premises servers need to resolve private AWS DNS names.**

→ **Route 53 Resolver inbound endpoint**

### Pattern

```text
On-premises
     ↓
DNS query
     ↓
Resolver inbound endpoint
     ↓
AWS / Route 53 private DNS
```

### Example

```text
On-premises application
        ↓
DNS query
        ↓
Route 53 Resolver inbound endpoint
        ↓
AWS private DNS
```

---

# Route 53 Resolver Outbound Endpoint

Allows DNS queries originating in **AWS** to be forwarded to external DNS servers, such as corporate/on-premises DNS servers.

### Signal

> **AWS resources need to resolve internal corporate DNS names.**

→ **Route 53 Resolver outbound endpoint**

### Pattern

```text
AWS workload
     ↓
DNS query
     ↓
Resolver outbound endpoint
     ↓
Corporate / on-premises DNS
```

---

# Inbound vs Outbound

```text
INBOUND
= query comes IN to AWS
= On-premises → AWS
```

```text
OUTBOUND
= query goes OUT from AWS
= AWS → On-premises
```

### Pocket memory

```text
On-prem → AWS DNS
→ Resolver inbound endpoint

AWS → On-prem DNS
→ Resolver outbound endpoint
```

Do not interpret inbound/outbound from the corporate network's perspective.

Think from **AWS's perspective**.

---

# AWS Outposts

**AWS Outposts = AWS infrastructure physically installed in your data center.**

Use it when:

* data must remain on-premises
* very low latency to local systems is required
* local workloads must use AWS APIs/services
* workloads must physically run in your own data center

### Pattern

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS-style infrastructure
```

### Signal

> **AWS infrastructure on-premises → Outposts**

### Example

> "A company must keep workloads in its own data center but wants to use AWS infrastructure and APIs."

→ **AWS Outposts**

---

# AWS Local Zones

**AWS Local Zones = AWS infrastructure placed closer to users in a metropolitan area.**

Use it when an application needs **very low latency for users in a specific city or metropolitan area**.

### Signal

> **City-level low latency → Local Zones**

### Pattern

```text
Main AWS Region
       ↓
Local Zone
       ↓
Users in the nearby metropolitan area
```

### Example

> "An application requires very low latency for users in a specific metropolitan area."

→ **AWS Local Zones**

### Memory

> **Local Zones = AWS CLOSER TO A CITY**

---

# AWS Wavelength

**AWS Wavelength = AWS infrastructure inside telecom 5G networks.**

It places AWS compute and storage at the edge of a telecommunications provider's 5G network.

Use it for:

* mobile applications
* 5G applications
* extremely low-latency mobile workloads
* workloads that process data close to mobile users

### Signal

> **5G → Wavelength**

### Pattern

```text
Mobile device
      ↓
5G network
      ↓
Wavelength Zone
      ↓
AWS resources at the telecom edge
```

This minimizes network distance between mobile users and the application.

### Example

> "A mobile application needs extremely low-latency processing over a 5G network."

→ **AWS Wavelength**

---

# Outposts vs Local Zones vs Wavelength

| Service         | AWS infrastructure location       | Main signal                    |
| --------------- | --------------------------------- | ------------------------------ |
| **Outposts**    | Your own data center              | AWS **on-premises**            |
| **Local Zones** | Near users in a metropolitan area | Very low latency to a **city** |
| **Wavelength**  | Inside a telecom 5G network       | **5G / mobile edge**           |

### Easy memory

```text
Outposts
= AWS IN YOUR DATA CENTER

Local Zones
= AWS CLOSER TO A CITY

Wavelength
= AWS ON 5G
```

---

# Network & Edge Decision Tree

### Infrastructure location

```text
Your own data center?
→ Outposts
```

```text
Near users in a city?
→ Local Zones
```

```text
Inside a 5G network?
→ Wavelength
```

### DNS direction

```text
On-premises → AWS
→ Resolver inbound endpoint
```

```text
AWS → On-premises
→ Resolver outbound endpoint
```

### Internet connectivity

```text
IPv6
+
Outbound Internet
+
Block unsolicited inbound
→ Egress-Only Internet Gateway
```

---

# Common Question Patterns

> **"IPv6 instances need outbound Internet access but must block unsolicited inbound connections."**

→ **Egress-Only Internet Gateway**

> **"On-premises servers need to resolve private AWS DNS names."**

→ **Route 53 Resolver inbound endpoint**

> **"AWS resources need to resolve internal corporate DNS names."**

→ **Route 53 Resolver outbound endpoint**

> **"Workloads must run in the company's own data center but use AWS infrastructure/services."**

→ **AWS Outposts**

> **"Need very low latency for users in a specific metropolitan area."**

→ **AWS Local Zones**

> **"Application needs extremely low-latency processing over a 5G network."**

→ **AWS Wavelength**

---

# Important SAA Traps

## Egress-Only vs Internet Gateway

The key clue is **IPv6 outbound-only**.

```text
IPv6 + outbound-only
→ Egress-Only Internet Gateway
```

Do not choose it simply because IPv6 appears; the **outbound-only** requirement matters.

---

## Resolver inbound vs outbound

Look at **where the DNS query starts**:

```text
On-prem → AWS
→ Inbound
```

```text
AWS → On-prem
→ Outbound
```

Think from **AWS's perspective**.

---

## Outposts vs Local Zones

```text
AWS infrastructure in your own facility
→ Outposts

AWS infrastructure near users in a metro area
→ Local Zones
```

---

## Local Zones vs Wavelength

```text
Specific city / metropolitan low latency
→ Local Zones

5G / telecom / mobile edge
→ Wavelength
```

---

# Pocket Card

| Keyword                                | Answer                                  |
| -------------------------------------- | --------------------------------------- |
| IPv6 + outbound-only Internet          | **Egress-Only Internet Gateway**        |
| On-prem → AWS DNS queries              | **Route 53 Resolver inbound endpoint**  |
| AWS → on-prem DNS queries              | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your data center | **Outposts**                            |
| Low latency to a specific city         | **Local Zones**                         |
| 5G / mobile edge                       | **Wavelength**                          |
