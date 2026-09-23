# Section 37B: Certificates, DNS, IPv6 & Edge Infrastructure

## The idea

These are smaller AWS networking and infrastructure services that usually appear as **specific-use-case questions**.

> **Requirement → unique keyword → service**

| Requirement / keyword                                | Answer                                  |
| ---------------------------------------------------- | --------------------------------------- |
| TLS / HTTPS certificates                             | **ACM**                                 |
| CloudFront TLS certificate                           | **ACM in us-east-1**                    |
| Multiple unrelated domains on one ALB HTTPS listener | **Multiple ACM certificates + SNI**     |
| IPv6 outbound-only Internet access                   | **Egress-Only Internet Gateway**        |
| On-premises → AWS DNS queries                        | **Route 53 Resolver inbound endpoint**  |
| AWS → on-premises DNS queries                        | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your own data center           | **Outposts**                            |
| Very low latency for a specific metropolitan area    | **Local Zones**                         |
| 5G / mobile edge                                     | **Wavelength**                          |

---

# ACM — AWS Certificate Manager

**ACM = TLS/SSL certificates for AWS services.**

Common integrations:

* Application Load Balancer (ALB)
* CloudFront
* API Gateway

### Common signal

> **HTTPS / TLS certificate → ACM**

---

## Important facts

### Public certificates

* Public ACM certificates are provided at **no additional charge**.
* ACM can **automatically renew** certificates that meet the renewal requirements.

### Regional behavior

ACM certificates are generally **Regional resources**.

For services such as ALB:

```text
ALB in us-east-1
→ ACM certificate in us-east-1
```

A regional service normally uses a certificate in the **same Region**.

### CloudFront special case

A certificate used by **CloudFront** must be in:

```text
us-east-1
```

Common exam trap:

```text
ALB
→ ACM certificate in the SAME REGION as the ALB

CloudFront
→ ACM certificate in us-east-1
```

---

## ACM and EC2

You generally cannot export an **ACM public certificate's private key** for direct installation on an EC2 server.

If the certificate must be installed directly on EC2 and the private key must be accessible, ACM public certificates are not the normal solution.

Common AWS-native pattern:

```text
Internet
   ↓
ALB / CloudFront / API Gateway
   ↓
ACM certificate
```

---

## ALB HTTPS Certificates & SNI

**SNI (Server Name Indication)** allows an ALB HTTPS listener to use **multiple SSL/TLS certificates on the same listener**.

The client sends the requested hostname during the TLS handshake, and the ALB selects the matching certificate.

Example:

```text
ALB :443
 ├── i-love-manila.com      → Certificate A
 ├── i-love-boracay.com     → Certificate B
 ├── i-love-cebu.com        → Certificate C
 └── another-domain.com     → Certificate D
```

Useful when **multiple unrelated domains** share the same ALB.

A new domain can be supported by **adding another certificate** to the listener instead of replacing the existing certificates.

### Certificate choices

| Requirement                                    | Best fit                        |
| ---------------------------------------------- | ------------------------------- |
| Multiple subdomains of one domain              | **Wildcard certificate**        |
| Multiple names in one certificate              | **SAN certificate**             |
| Multiple unrelated domains on one ALB listener | **Multiple certificates + SNI** |

Example:

```text
*.example.com
→ www.example.com
→ api.example.com
→ shop.example.com
```

A wildcard is **not** for unrelated domains such as:

```text
i-love-manila.com
i-love-boracay.com
i-love-cebu.com
```

### Exam pattern

> **Many unrelated domains + one ALB HTTPS listener → SNI**

---

## Example

> "A company needs to configure HTTPS for an Application Load Balancer."

→ **AWS Certificate Manager (ACM)**

---

## CloudFront example

> "A company needs to configure a TLS certificate for a CloudFront distribution."

→ **ACM in us-east-1**

---

# Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

Allows IPv6 instances to initiate outbound Internet connections while preventing **unsolicited inbound connections**.

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
On-premises → AWS
→ Inbound endpoint

AWS → On-premises
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

Do not interpret inbound/outbound from the corporate network's perspective. Think from **AWS's perspective**.

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

> "A company must keep workloads in its own data center because of local requirements but wants to use AWS infrastructure and APIs."

→ **AWS Outposts**

### Important distinction

```text
Your own data center
→ Outposts

AWS infrastructure near a city
→ Local Zones

AWS infrastructure inside a 5G network
→ Wavelength
```

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

Places AWS compute and storage at the edge of a telecommunications provider's 5G network.

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

### Certificates

```text
TLS / HTTPS
→ ACM
```

---

# Common Question Patterns

> **"Need HTTPS certificate."**

→ **AWS Certificate Manager (ACM)**

> **"Need HTTPS certificate for an ALB."**

→ **ACM**

Use the ACM certificate in the **same Region as the ALB**.

> **"Need HTTPS certificate for CloudFront."**

→ **ACM in us-east-1**

> **"Multiple unrelated domains share one ALB HTTPS listener."**

→ **Multiple certificates + SNI**

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

## ACM Region trap

```text
ALB
→ ACM certificate in same Region

CloudFront
→ ACM certificate in us-east-1
```

---

## ALB certificate trap

```text
Many unrelated domains
+
One ALB HTTPS listener
→ Multiple certificates + SNI
```

Do not confuse:

```text
Subdomains of one domain
→ Wildcard

Multiple names in one certificate
→ SAN

Multiple unrelated domains on one ALB
→ SNI
```

---

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

| Keyword                                  | Answer                                  |
| ---------------------------------------- | --------------------------------------- |
| TLS/SSL certificates                     | **ACM**                                 |
| HTTPS                                    | **ACM**                                 |
| ALB certificate                          | **ACM in same Region as ALB**           |
| CloudFront certificate                   | **ACM in us-east-1**                    |
| Multiple unrelated domains on one ALB    | **SNI + multiple certificates**         |
| Multiple subdomains of one domain        | **Wildcard certificate**                |
| Multiple domain names in one certificate | **SAN certificate**                     |
| IPv6 outbound-only Internet              | **Egress-Only Internet Gateway**        |
| On-prem → AWS DNS queries                | **Route 53 Resolver inbound endpoint**  |
| AWS → on-prem DNS queries                | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your data center   | **Outposts**                            |
| Low latency to a specific city           | **Local Zones**                         |
| 5G edge                                  | **Wavelength**                          |

---
