# Section 37B: Certificates, DNS, IPv6 & Edge Infrastructure

## The idea

These are smaller AWS networking and infrastructure services that usually appear in SAA questions as **specific use cases**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
TLS / HTTPS certificates
→ ACM

CloudFront TLS certificate
→ ACM in us-east-1

IPv6 outbound-only Internet access
→ Egress-Only Internet Gateway

On-premises → AWS DNS queries
→ Route 53 Resolver inbound endpoint

AWS → on-premises DNS queries
→ Route 53 Resolver outbound endpoint

AWS infrastructure in your own data center
→ Outposts

Very low latency for a specific metropolitan area
→ Local Zones

5G / mobile edge
→ Wavelength
```

---

# ACM — AWS Certificate Manager

**ACM = TLS/SSL certificates for AWS services.**

Use it when you need HTTPS.

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
* ACM can automatically renew certificates that meet the renewal requirements.

### Regional behavior

ACM certificates are generally **Regional resources**.

For services such as an ALB:

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

This is a very common SAA exam trap.

So:

```text
HTTPS on ALB
→ ACM certificate in the SAME REGION as the ALB

HTTPS on CloudFront
→ ACM certificate in us-east-1
```

---

## ACM and EC2

You generally cannot export an **ACM public certificate's private key** for installation on an EC2 server.

So if a question says the certificate must be installed directly on an EC2 instance and requires access to the private key, ACM public certificates are not the normal solution.

The common AWS-native pattern is to terminate TLS on an AWS service such as:

```text
Internet
   ↓
ALB / CloudFront / API Gateway
   ↓
ACM certificate
```

---

## Example

> "A company needs to configure HTTPS for an Application Load Balancer."

→ **AWS Certificate Manager (ACM)**

---

## CloudFront example

> "A company needs to configure a TLS certificate for a CloudFront distribution."

→ **ACM**

But remember:

> **CloudFront certificate → ACM in us-east-1**

---

# Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

It allows instances using IPv6 to initiate outbound Internet connections, while preventing unsolicited inbound connections from the Internet.

### Signal

> **IPv6 instances need outbound Internet access but must block unsolicited inbound connections.**

→ **Egress-Only Internet Gateway**

---

## Mental model

```text
IPv6 instance
      ↓
Egress-Only Internet Gateway
      ↓
Internet
```

Outbound connections are allowed:

```text
EC2 → Internet
```

But unsolicited inbound connections are not allowed:

```text
Internet → EC2
```

---

## Important distinction

An **Internet Gateway (IGW)** is the normal gateway for Internet connectivity.

An **Egress-Only Internet Gateway** is specifically designed for **IPv6 outbound-only access**.

The key exam signal is:

```text
IPv6
+
Outbound Internet
+
No unsolicited inbound
→ Egress-Only Internet Gateway
```

### Memory

> **Egress-Only IGW = IPv6 OUTBOUND ONLY**

---

# Route 53 Resolver

AWS provides **Route 53 Resolver endpoints** for DNS resolution between AWS and external/on-premises DNS environments.

There are two important directions to remember:

```text
On-premises → AWS
→ Inbound endpoint

AWS → On-premises
→ Outbound endpoint
```

The direction is based on **where the DNS query starts**.

---

# Route 53 Resolver Inbound Endpoint

A **Route 53 Resolver inbound endpoint** allows DNS queries from on-premises networks to be sent into AWS.

### Signal

> **On-premises servers need to resolve private AWS DNS names.**

→ **Route 53 Resolver inbound endpoint**

### Mental model

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

Suppose an on-premises application needs to resolve an internal AWS hostname.

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

A **Route 53 Resolver outbound endpoint** allows DNS queries originating in AWS to be forwarded to DNS servers outside AWS, such as corporate/on-premises DNS servers.

### Signal

> **AWS resources need to resolve internal corporate DNS names.**

→ **Route 53 Resolver outbound endpoint**

### Mental model

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

This is one of the easiest ways to memorize it:

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

---

# AWS Outposts

**AWS Outposts = AWS infrastructure physically installed in your data center.**

It extends AWS infrastructure and services into your own on-premises environment.

Use it when:

* data must remain on-premises
* you need very low latency to local systems
* local workloads must use AWS APIs/services
* the workload must physically run in your own data center

### Typical pattern

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS-style infrastructure
```

### Signal

> **AWS infrastructure on-premises → Outposts**

---

## Example

> "A company must keep workloads in its own data center because of local requirements but wants to use AWS infrastructure and APIs."

→ **AWS Outposts**

---

## Important distinction

Outposts is different from services that simply provide low latency near users.

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

Use it when an application needs very low latency for users in a specific city or metropolitan area.

### Signal

> **City-level low latency → Local Zones**

---

## Typical scenario

A company has an application deployed in an AWS Region, but users in a particular metropolitan area need extremely low latency.

Instead of having the workload run only in the main AWS Region:

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

---

## Memory

> **Local Zones = AWS CLOSER TO A CITY**

---

# AWS Wavelength

**AWS Wavelength = AWS infrastructure inside telecom 5G networks.**

It places AWS compute and storage resources at the edge of a telecommunications provider's 5G network.

Use it for:

* mobile applications
* 5G applications
* extremely low-latency mobile workloads
* workloads that need to process data close to mobile users

### Signal

> **5G → Wavelength**

---

## Typical pattern

```text
Mobile device
      ↓
5G network
      ↓
Wavelength Zone
      ↓
AWS resources at the telecom edge
```

This minimizes the network distance between mobile users and the application.

---

## Example

> "A mobile application needs extremely low-latency processing over a 5G network."

→ **AWS Wavelength**

---

# Outposts vs Local Zones vs Wavelength

These three are easy to mix up.

| Service         | Where AWS infrastructure is located | Main signal                    |
| --------------- | ----------------------------------- | ------------------------------ |
| **Outposts**    | Your own data center                | AWS **on-premises**            |
| **Local Zones** | Near users in a metropolitan area   | Very low latency to a **city** |
| **Wavelength**  | Inside a telecom 5G network         | **5G / mobile edge**           |

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

When you see one of these questions, identify the location requirement first.

```text
Where does the infrastructure need to be?

                ┌───────────────────────┐
                │ Your own data center? │
                └───────────┬───────────┘
                            ↓
                         Outposts
```

```text
                ┌─────────────────────────┐
                │ Near users in a city?   │
                └────────────┬────────────┘
                             ↓
                        Local Zones
```

```text
                ┌─────────────────────────┐
                │ Inside a 5G network?    │
                └────────────┬────────────┘
                             ↓
                         Wavelength
```

For DNS:

```text
DNS query starts where?

On-premises
     ↓
AWS
→ Resolver inbound endpoint
```

```text
AWS
 ↓
On-premises
→ Resolver outbound endpoint
```

For Internet connectivity:

```text
IPv6
+
Outbound Internet
+
Block unsolicited inbound
→ Egress-Only Internet Gateway
```

For certificates:

```text
TLS / HTTPS
→ ACM
```

---

# Common Question Patterns

> **"Need HTTPS certificate."**

→ **AWS Certificate Manager (ACM)**

---

> **"Need HTTPS certificate for an ALB."**

→ **ACM**

Use the ACM certificate in the **same Region as the ALB**.

---

> **"Need HTTPS certificate for CloudFront."**

→ **ACM in us-east-1**

---

> **"IPv6 instances need outbound Internet access but must block unsolicited inbound connections."**

→ **Egress-Only Internet Gateway**

---

> **"On-premises servers need to resolve private AWS DNS names."**

→ **Route 53 Resolver inbound endpoint**

---

> **"AWS resources need to resolve internal corporate DNS names."**

→ **Route 53 Resolver outbound endpoint**

---

> **"Workloads must run in the company's own data center but use AWS infrastructure/services."**

→ **AWS Outposts**

---

> **"Need very low latency for users in a specific metropolitan area."**

→ **AWS Local Zones**

---

> **"Application needs extremely low-latency processing over a 5G network."**

→ **AWS Wavelength**

---

# Important SAA Traps

## ACM Region trap

A frequent exam mistake is assuming an ACM certificate can be used from any Region.

Remember:

```text
ALB
→ ACM certificate in same Region

CloudFront
→ ACM certificate in us-east-1
```

---

## Egress-Only vs Internet Gateway

The most important clue is **IPv6 outbound-only**.

```text
IPv6 + outbound-only
→ Egress-Only Internet Gateway
```

Do not choose it simply because IPv6 appears in the question. The outbound-only requirement matters.

---

## Resolver inbound vs outbound

Look at the direction of the query.

```text
On-prem → AWS
→ Inbound

AWS → On-prem
→ Outbound
```

Don't interpret "inbound" and "outbound" from the perspective of the corporate network. Think from the perspective of **AWS**.

---

## Outposts vs Local Zones

```text
Physical AWS infrastructure in your own facility
→ Outposts

Physical AWS infrastructure near users in a metro area
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
| TLS/SSL certificates                   | **ACM**                                 |
| HTTPS                                  | **ACM**                                 |
| ALB certificate                        | **ACM in same Region as ALB**           |
| CloudFront certificate                 | **ACM in us-east-1**                    |
| IPv6 outbound-only Internet            | **Egress-Only Internet Gateway**        |
| On-prem → AWS DNS queries              | **Route 53 Resolver inbound endpoint**  |
| AWS → on-prem DNS queries              | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your data center | **Outposts**                            |
| Low latency to a specific city         | **Local Zones**                         |
| 5G edge                                | **Wavelength**                          |

---

# Final Memory

```text
ACM
= HTTPS CERTIFICATES
= TLS / SSL
= CLOUDFRONT → us-east-1

Egress-Only Internet Gateway
= IPv6 OUTBOUND ONLY

Route 53 Resolver inbound endpoint
= ON-PREM → AWS DNS

Route 53 Resolver outbound endpoint
= AWS → ON-PREM DNS

Outposts
= AWS IN YOUR DATA CENTER

Local Zones
= AWS CLOSER TO A CITY

Wavelength
= AWS ON 5G
```

# The Golden Rule

```text
TLS / HTTPS
→ ACM

CloudFront certificate
→ ACM in us-east-1

IPv6 outbound-only
→ Egress-Only IGW

On-prem → AWS DNS
→ Resolver inbound

AWS → on-prem DNS
→ Resolver outbound

AWS in your own data center
→ Outposts

Low latency to a city
→ Local Zones

5G / mobile edge
→ Wavelength
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
TLS              → ACM
CloudFront       → ACM us-east-1
IPv6 outbound    → Egress-Only IGW
On-prem → AWS DNS → Resolver inbound
AWS → on-prem DNS → Resolver outbound
Own data center  → Outposts
Specific city    → Local Zones
5G               → Wavelength
```
