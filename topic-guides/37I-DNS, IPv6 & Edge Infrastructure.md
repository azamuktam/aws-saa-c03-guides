# Section 37I: DNS, IPv6 & Edge Infrastructure

## The idea

These are smaller AWS networking and infrastructure services that usually appear as **specific-use-case questions**.

> **Requirement → unique keyword → service**

| Requirement / keyword                             | Answer                                  |
| ------------------------------------------------- | --------------------------------------- |
| IPv6 outbound-only Internet access                | **Egress-Only Internet Gateway**        |
| On-premises → AWS DNS queries                     | **Route 53 Resolver inbound endpoint**  |
| AWS → on-premises DNS queries                     | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your own data center        | **AWS Outposts**                        |
| Very low latency for a specific metropolitan area | **AWS Local Zones**                     |
| 5G / mobile edge                                  | **AWS Wavelength**                      |

---

# Route 53 Resolver

## The idea

**Route 53 Resolver connects DNS between your AWS VPC and your on-premises/corporate network.**

You need it when:

```text
AWS VPC  ↔  On-premises / Corporate network
```

and one side needs to resolve DNS names that are known only by the other side.

---

## First: What problem does it solve?

DNS converts a hostname into an IP address:

```text
database.company.local
        ↓
    10.20.30.40
```

Your company may have a **corporate DNS server** that knows internal names such as:

```text
payroll.company.local
git.company.local
database.company.local
```

AWS has its own DNS resolution inside a VPC.

Now imagine AWS and your corporate network are connected through:

```text
VPN
or
Direct Connect
```

You can have:

```text
Corporate network
        ↕
   VPN / Direct Connect
        ↕
      AWS VPC
```

The networks can communicate, but **DNS still needs to know where to send queries**.

That's where Route 53 Resolver endpoints come in.

---

# There are TWO directions

The easiest way to understand Resolver is to look at **where the DNS query starts**.

```text
On-premises → AWS
```

uses:

> **Inbound Resolver endpoint**

```text
AWS → On-premises
```

uses:

> **Outbound Resolver endpoint**

---

# Inbound Resolver Endpoint

## What does it do?

It allows **DNS queries coming from on-premises into AWS**.

Example:

Your AWS VPC has a private service:

```text
database.internal
        ↓
   10.20.5.30
```

Your on-premises server wants to know:

```text
"What IP address is database.internal?"
```

Your corporate DNS does not know the AWS private name.

The query can go:

```text
On-premises
      ↓
Corporate DNS
      ↓
Inbound Resolver Endpoint
      ↓
AWS DNS / Route 53
      ↓
10.20.5.30
```

### Why is it called "inbound"?

The DNS query is **coming into AWS**:

```text
ON-PREM → AWS
```

Therefore:

> **Inbound Resolver Endpoint**

---

# Outbound Resolver Endpoint

## What does it do?

It allows **DNS queries originating in AWS to reach on-premises DNS servers**.

Example:

Your company has:

```text
payroll.company.local
        ↓
    10.10.5.20
```

Your corporate DNS knows this name.

An EC2 instance in AWS asks:

```text
"What IP is payroll.company.local?"
```

AWS's normal DNS does not know the corporate name.

The query can go:

```text
EC2
 ↓
AWS DNS
 ↓
Outbound Resolver Endpoint
 ↓
Corporate DNS
 ↓
10.10.5.20
```

### Why is it called "outbound"?

The DNS query is **leaving AWS**:

```text
AWS → ON-PREM
```

Therefore:

> **Outbound Resolver Endpoint**

---

# The most important Resolver diagram

```text
                    AWS
             ┌─────────────────┐
             │                 │
             │      VPC        │
             │                 │
             │      EC2        │
             │        ↓        │
             │   AWS DNS       │
             │                 │
             └─────────────────┘
                ↑           ↓
                │           │
          INBOUND        OUTBOUND
                │           │
                ↓           ↓
        ┌─────────────────────────┐
        │   Corporate Network     │
        │                         │
        │   Corporate DNS         │
        └─────────────────────────┘
```

The simple version:

```text
On-prem → AWS
= INBOUND

AWS → On-prem
= OUTBOUND
```

---

# What does Resolver actually transfer?

The Resolver endpoint forwards **DNS queries**.

It is not primarily for:

* transferring application data
* routing normal network traffic
* replacing VPN
* replacing Direct Connect

Think of it as a **DNS bridge**.

```text
Network connectivity
        +
DNS Resolver endpoints
        =
AWS and on-prem can resolve each other's names
```

---

# Resolver does NOT create network connectivity

You still need network connectivity such as:

```text
VPN
```

or:

```text
Direct Connect
```

For example:

```text
          VPN
AWS  <------------>  On-prem
 │                       │
 │                       │
 └── Resolver ───────────┘
       DNS queries
```

The VPN or Direct Connect allows the networks to communicate.

The Resolver endpoint handles the **DNS part**.

---

# Common Resolver scenarios

## Scenario 1

> On-premises servers need to resolve private DNS names in AWS.

Think:

```text
On-prem → AWS
```

Answer:

> **Route 53 Resolver inbound endpoint**

---

## Scenario 2

> EC2 instances need to resolve internal corporate DNS names.

Think:

```text
AWS → On-prem
```

Answer:

> **Route 53 Resolver outbound endpoint**

---

## Scenario 3

> AWS and on-premises are connected through Direct Connect, but AWS workloads cannot resolve internal corporate hostnames.

Think:

```text
AWS → Corporate DNS
```

Answer:

> **Resolver outbound endpoint**

---

## Scenario 4

> On-premises applications cannot resolve private Route 53 names hosted in AWS.

Think:

```text
On-prem → AWS
```

Answer:

> **Resolver inbound endpoint**

---

# Don't confuse DNS direction with network ownership

The names **inbound/outbound are from AWS's perspective**.

So:

```text
On-prem → AWS
= INBOUND to AWS
```

```text
AWS → On-prem
= OUTBOUND from AWS
```

Always ask:

> **"Is the DNS query entering AWS or leaving AWS?"**

---

# Route 53 Hosted Zones vs Resolver

These are different things.

## Route 53 Hosted Zone

Stores DNS records such as:

```text
www.example.com → 10.20.30.40
```

Think:

> **Where are the DNS records?**

## Route 53 Resolver

Handles DNS queries and forwarding between DNS environments.

Think:

> **Where should this DNS query go?**

---

# Quick comparison

| Requirement                            | Answer                         |
| -------------------------------------- | ------------------------------ |
| Store/manage DNS records               | **Route 53 Hosted Zone**       |
| On-prem DNS → AWS DNS                  | **Resolver inbound endpoint**  |
| AWS DNS → on-prem DNS                  | **Resolver outbound endpoint** |
| Connect AWS network to on-prem network | **VPN / Direct Connect**       |

---

# Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

It allows IPv6 instances to initiate outbound Internet connections while preventing **unsolicited inbound connections**.

## Signal

> **IPv6 + outbound Internet + block unsolicited inbound**

→ **Egress-Only Internet Gateway**

## Mental model

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

## Important distinction

**Internet Gateway (IGW)** = normal Internet connectivity.

**Egress-Only Internet Gateway** = specifically for **IPv6 outbound-only access**.

### Memory

> **Egress-Only IGW = IPv6 OUTBOUND ONLY**

---

# AWS Outposts

**AWS Outposts = AWS infrastructure physically installed in your own data center.**

Use it when:

* workloads must physically remain on-premises
* very low latency to local systems is required
* local workloads need AWS APIs/services
* the company wants AWS-style infrastructure in its own facility

## Pattern

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS infrastructure
```

## Signal

> **AWS infrastructure on-premises → Outposts**

## Example

> "A company must keep workloads in its own data center but wants to use AWS infrastructure and APIs."

→ **AWS Outposts**

---

# AWS Local Zones

**AWS Local Zones = AWS infrastructure placed closer to users in a specific metropolitan area.**

Use it when an application needs **very low latency for users in a particular city or metropolitan area**.

## Signal

> **City-level low latency → Local Zones**

## Pattern

```text
Main AWS Region
       ↓
Local Zone
       ↓
Users in nearby metropolitan area
```

## Example

> "An application requires very low latency for users in a specific metropolitan area."

→ **AWS Local Zones**

### Memory

> **Local Zones = AWS CLOSER TO A CITY**

---

# AWS Wavelength

**AWS Wavelength = AWS infrastructure inside a telecommunications provider's 5G network.**

It places AWS compute and storage at the edge of a telecom 5G network.

Use it for:

* mobile applications
* 5G applications
* extremely low-latency mobile workloads
* processing data close to mobile users

## Signal

> **5G → Wavelength**

## Pattern

```text
Mobile device
      ↓
5G network
      ↓
Wavelength Zone
      ↓
AWS resources at telecom edge
```

This minimizes network distance between mobile users and the application.

## Example

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

## DNS

```text
On-premises → AWS DNS
→ Resolver inbound endpoint
```

```text
AWS → On-premises DNS
→ Resolver outbound endpoint
```

---

## Internet connectivity

```text
IPv6
+
Outbound Internet
+
Block unsolicited inbound
→ Egress-Only Internet Gateway
```

---

## Infrastructure location

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

---

# Common Question Patterns

> **"On-premises servers need to resolve private AWS DNS names."**

→ **Route 53 Resolver inbound endpoint**

> **"AWS resources need to resolve internal corporate DNS names."**

→ **Route 53 Resolver outbound endpoint**

> **"IPv6 instances need outbound Internet access but must block unsolicited inbound connections."**

→ **Egress-Only Internet Gateway**

> **"Workloads must run in the company's own data center but use AWS infrastructure/services."**

→ **AWS Outposts**

> **"Need very low latency for users in a specific metropolitan area."**

→ **AWS Local Zones**

> **"Application needs extremely low-latency processing over a 5G network."**

→ **AWS Wavelength**

---

# Important SAA Traps

## Resolver inbound vs outbound

Look at **where the DNS query starts**:

```text
On-prem → AWS
→ INBOUND
```

```text
AWS → On-prem
→ OUTBOUND
```

Think from **AWS's perspective**.

---

## Egress-Only vs Internet Gateway

The key clue is:

```text
IPv6
+
outbound-only
```

→ **Egress-Only Internet Gateway**

Do not choose it simply because IPv6 appears. The **outbound-only** requirement matters.

---

## Outposts vs Local Zones

```text
AWS infrastructure in your own facility
→ Outposts
```

```text
AWS infrastructure near users in a metropolitan area
→ Local Zones
```

---

## Local Zones vs Wavelength

```text
Specific city / metropolitan low latency
→ Local Zones
```

```text
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

---

