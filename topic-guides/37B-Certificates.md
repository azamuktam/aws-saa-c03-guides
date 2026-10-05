# Section 37B: Certificates & TLS

## The idea

This section covers AWS certificate management and TLS/HTTPS patterns that commonly appear as **specific-use-case questions**.

> **Requirement → keyword → answer**

| Requirement / keyword                                | Answer                                   |
| ---------------------------------------------------- | ---------------------------------------- |
| Need a new public TLS / HTTPS certificate            | **ACM**                                  |
| TLS / HTTPS certificates                             | **ACM**                                  |
| Third-party certificate that must be imported        | **ACM**                                  |
| Third-party certificate import alternative           | **IAM certificate store**                |
| CloudFront TLS certificate                           | **ACM in us-east-1**                     |
| ALB TLS certificate                                  | **ACM in the same Region as the ALB**    |
| Regional API Gateway TLS certificate                 | **ACM in the same Region as the API**    |
| Edge-optimized API Gateway certificate               | **ACM in us-east-1**                     |
| Multiple unrelated domains on one ALB HTTPS listener | **Multiple ACM certificates + SNI**      |
| Multiple subdomains of one domain                    | **Wildcard certificate**                 |
| Multiple names in one certificate                    | **SAN certificate**                      |
| Certificate expiring soon                            | **ACM expiration events / DaysToExpiry** |

---

# ACM — AWS Certificate Manager

**ACM = TLS/SSL certificate management for AWS services.**

ACM can:

* **Request and issue new public certificates**
* **Import existing certificates** from third-party Certificate Authorities (CAs)
* Manage certificates for supported AWS services

Common integrations:

* Application Load Balancer (ALB)
* CloudFront
* API Gateway

### Common signal

> **HTTPS / TLS certificate → ACM**

---

## ACM can request / issue certificates

ACM can **request a new public TLS certificate** for your domain.

Typical flow:

```text
You
 ↓
ACM: Request certificate
 ↓
Prove domain ownership
 ↓
ACM issues certificate
 ↓
Use with ALB / CloudFront / API Gateway
```

Domain ownership can be validated using:

* **DNS validation**
* **Email validation**

### Important

> **ACM can create/issue a new public certificate for you.**

So don't think of ACM only as a place where you import certificates.

It can either:

```text
Need a new certificate
→ ACM requests/issues it
```

or:

```text
Already have a certificate from another CA
→ Import it into ACM
```

---

## Public ACM certificates

Public ACM certificates are available at **no additional certificate charge**.

ACM can automatically renew eligible **ACM-issued certificates** when the renewal requirements are met.

---

## Importing third-party certificates

You can import a certificate obtained from an external Certificate Authority (CA), such as a commercial third-party CA, into ACM.

When importing, you provide:

* Certificate
* Private key
* Certificate chain, when applicable

Example:

```text
Third-party CA
      ↓
Certificate + private key + chain
      ↓
     ACM
      ↓
ALB / other supported AWS service
```

### Important

> **Imported certificates are not automatically renewed by ACM.**

You must obtain the renewed certificate from the CA and import the new certificate.

This is different from an ACM-issued public certificate, which ACM can automatically renew when the certificate is eligible.

---

## IAM certificate store

AWS IAM also provides a **server certificate store** where third-party certificates can be uploaded.

For SAA purposes:

> **ACM = modern/default certificate management choice**

> **IAM certificate store = older/alternative certificate store**

A question may therefore list both:

* **AWS Certificate Manager**
* **AWS IAM certificate store**

as valid places where a third-party certificate can be imported.

For an ALB, ACM is normally the preferred AWS-native solution.

---

## S3 is not an ALB certificate store

You could technically store certificate files in an S3 bucket, but that does **not** make S3 a certificate management service.

For example:

```text
certificate.pem
private-key.pem
certificate-chain.pem
```

can exist as S3 objects, but an ALB does not simply use an S3 object as its HTTPS listener certificate.

Typical pattern:

```text
Third-party certificate
        ↓
       ACM
        ↓
ALB HTTPS listener
```

Do not choose S3 merely because the question says the certificate must be stored securely.

---

# ACM and EC2

You generally cannot use an ACM public certificate's private key directly on an EC2 server in the same way that you would with a certificate file you manage yourself.

For an ALB / CloudFront / API Gateway architecture, the common pattern is:

```text
Internet
   ↓
ALB / CloudFront / API Gateway
   ↓
ACM certificate
   ↓
Application
```

The TLS certificate is associated with the AWS service that terminates HTTPS.

---

# ACM Regional behavior

ACM certificates are generally **Regional resources**.

For services such as ALB:

```text
ALB in us-east-1
→ ACM certificate in us-east-1
```

A regional service normally uses a certificate in the **same Region**.

---

# CloudFront special case

A certificate used by **CloudFront** must be in:

```text
us-east-1
```

Even if the rest of your infrastructure is deployed in another Region.

Common exam trap:

```text
ALB
→ ACM certificate in the SAME REGION as the ALB

CloudFront
→ ACM certificate in us-east-1
```

---

## API Gateway certificate region

API Gateway has an important **Regional vs Edge-optimized** distinction.

### Regional API Gateway

```text
Regional API in us-east-2
→ ACM certificate in us-east-2
```

The certificate must be in the **same Region as the Regional API**.

### Edge-optimized API Gateway

An **Edge-optimized API Gateway endpoint uses an API Gateway-managed CloudFront distribution**.

Therefore:

```text
Edge-optimized API Gateway
→ API Gateway-managed CloudFront
→ ACM certificate in us-east-1
```

### SAA memory

```text
ALB
→ ACM same Region as ALB

Regional API Gateway
→ ACM same Region as API

CloudFront
→ ACM us-east-1

Edge-optimized API Gateway
→ ACM us-east-1
```

Do not use the broad rule:

> **"API Gateway → ACM us-east-1"**

First check whether the API is **Regional** or **Edge-optimized**.

---

# ACM certificate types

## ACM-issued public certificate

ACM requests and issues the certificate.

```text
ACM
 ↓
Public certificate
```

ACM can automatically renew eligible certificates.

---

## ACM private certificate

Used for internal/private PKI scenarios.

```text
AWS Private CA
      ↓
Private certificate
      ↓
ACM / supported AWS service
```

---

## Imported certificate

You already obtained the certificate from another CA.

```text
DigiCert / other CA
        ↓
   Import into ACM
        ↓
Use with supported AWS service
```

Important:

> **Imported certificate → you handle renewal and re-import.**

---

# Certificate expiration monitoring

When the requirement is:

> **"Notify me before an ACM certificate expires."**

Think of two important AWS patterns.

### 1. ACM expiration events → EventBridge

ACM provides certificate expiration events through **Amazon EventBridge**.

Typical pattern:

```text
ACM certificate
      ↓
Approaching-expiration event
      ↓
Amazon EventBridge
      ↓
SNS / Lambda / other target
```

The event includes information such as **`DaysToExpiry`**.

This is the most direct **event-driven** approach.

---

### 2. `DaysToExpiry` metric → scheduled checking

ACM also publishes the **`DaysToExpiry`** CloudWatch metric.

It indicates how many days remain before a certificate expires.

Possible pattern:

```text
ACM
 ↓
DaysToExpiry
 ↓
Scheduled daily checking
 ↓
Find certificates reaching 30 days
 ↓
SNS notification
```

### Important distinction

Do not think:

> **EventBridge itself is a CloudWatch metric alarm.**

A scheduled EventBridge rule provides the **trigger** for a process that checks the metric and then sends the notification.

For SAA questions, the intended concept is:

> **ACM expiration events → EventBridge**

or

> **DaysToExpiry metric → periodic monitoring/checking**

---

### AWS Health

AWS Health can also provide ACM-related renewal/expiration events, particularly around **renewal status and situations requiring customer action**.

Useful exam distinction:

```text
ACM EventBridge expiration event
→ approaching certificate expiration

AWS Health
→ renewal / renewal-status / action-required events
```

---

### Exam memory

```text
Need notification before ACM certificate expires
        ↓
ACM expiration events → EventBridge → SNS
        OR
DaysToExpiry metric → scheduled checking → SNS
```

---

# ALB HTTPS Certificates & SNI

**SNI (Server Name Indication)** allows an ALB HTTPS listener to use **multiple certificates on the same listener**.

The client includes the requested hostname during the TLS handshake, allowing the ALB to select the appropriate certificate.

Example:

```text
ALB :443
 ├── i-love-manila.com      → Certificate A
 ├── i-love-boracay.com     → Certificate B
 ├── i-love-cebu.com        → Certificate C
 └── another-domain.com     → Certificate D
```

Useful when **multiple unrelated domains** share one ALB.

A new domain can be supported by adding another certificate to the listener rather than replacing the existing certificate.

### Exam pattern

> **Many unrelated domains + one ALB HTTPS listener → SNI**

---

# Wildcard vs SAN vs SNI

| Requirement                                    | Best fit                        |
| ---------------------------------------------- | ------------------------------- |
| Multiple subdomains of one domain              | **Wildcard certificate**        |
| Multiple names in one certificate              | **SAN certificate**             |
| Multiple unrelated domains on one ALB listener | **Multiple certificates + SNI** |

---

## Wildcard certificate

A wildcard covers multiple subdomains under the same domain.

Example:

```text
*.example.com
```

Can cover:

```text
www.example.com
api.example.com
shop.example.com
```

A wildcard is **not** intended for unrelated domains such as:

```text
i-love-manila.com
i-love-boracay.com
i-love-cebu.com
```

---

## SAN certificate

A **SAN (Subject Alternative Name)** certificate can contain multiple domain names in one certificate.

Useful when multiple names should be covered by **one certificate**.

---

## SNI

SNI is different from SAN.

**SNI allows the ALB listener to have multiple certificates**, with the ALB selecting the correct certificate based on the hostname requested by the client.

Think:

```text
Wildcard
→ one certificate covers many subdomains

SAN
→ one certificate contains many names

SNI
→ one listener uses multiple certificates
```

---

# Common Question Patterns

> **"Need a new public HTTPS/TLS certificate."**

→ **AWS Certificate Manager (ACM)**

> **"Need HTTPS certificate for an Application Load Balancer."**

→ **ACM**

Use the ACM certificate in the **same Region as the ALB**.

> **"Need a TLS certificate for CloudFront."**

→ **ACM in us-east-1**

> **"A Regional API Gateway needs a custom HTTPS domain."**

→ **ACM certificate in the same Region as the API**

> **"An Edge-optimized API Gateway needs a custom HTTPS domain."**

→ **ACM certificate in us-east-1**

> **"A company obtained a certificate from a third-party CA and needs to import it into AWS."**

→ **ACM**
→ **IAM certificate store** can also be a valid certificate-store answer depending on the choices.

> **"Multiple unrelated domains share one ALB HTTPS listener."**

→ **Multiple certificates + SNI**

> **"Several subdomains belong to one domain."**

→ **Wildcard certificate**

> **"One certificate must cover multiple domain names."**

→ **SAN certificate**

> **"Security team wants an alert 30 days before ACM certificates expire."**

→ **ACM expiration events → EventBridge → SNS**

or

→ **`DaysToExpiry` metric → scheduled checking → SNS**

---

# Important SAA Traps

## ACM Region trap

```text
ALB
→ ACM certificate in same Region

Regional API Gateway
→ ACM certificate in same Region

CloudFront
→ ACM certificate in us-east-1

Edge-optimized API Gateway
→ ACM certificate in us-east-1
```

---

## ACM issue vs import

```text
Need a new public certificate
→ ACM requests/issues it

Already have a third-party certificate
→ Import into ACM
```

---

## Imported certificate trap

```text
Third-party certificate
→ Import into ACM
→ ACM does NOT automatically renew it
→ Renew it externally and import the replacement
```

---

## Certificate expiration trap

```text
Approaching ACM expiration
→ EventBridge expiration events

Days remaining as a metric
→ CloudWatch DaysToExpiry

Need notification
→ SNS

Need a daily metric-based check
→ scheduled process / EventBridge schedule
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

# Certificate Pocket Card

| Keyword                                   | Answer                             |
| ----------------------------------------- | ---------------------------------- |
| Need a new public TLS certificate         | **ACM**                            |
| TLS / SSL certificate                     | **ACM**                            |
| HTTPS                                     | **ACM**                            |
| Third-party certificate import            | **ACM**                            |
| Third-party certificate store alternative | **IAM certificate store**          |
| Imported certificate renewal              | **Manual / re-import**             |
| ALB certificate                           | **ACM in same Region as ALB**      |
| Regional API Gateway certificate          | **ACM in same Region as API**      |
| CloudFront certificate                    | **ACM in us-east-1**               |
| Edge-optimized API Gateway certificate    | **ACM in us-east-1**               |
| Certificate expiring soon                 | **ACM EventBridge / DaysToExpiry** |
| Alert before certificate expiry           | **EventBridge → SNS**              |
| Metric showing days remaining             | **`DaysToExpiry`**                 |
| Multiple unrelated domains on one ALB     | **SNI + multiple certificates**    |
| Multiple subdomains of one domain         | **Wildcard certificate**           |
| Multiple names in one certificate         | **SAN certificate**                |
| S3 certificate storage                    | **Not an ALB certificate store**   |

---

# Questions

### Q1 — Certificate expiration notification

**Scenario:**
A company uses ACM certificates on ALBs and wants to notify the security team **30 days before expiration**. Which approaches can satisfy the requirement?

**Correct concepts:**

**1. ACM certificate expiration events → EventBridge → SNS**

```text
ACM
 ↓
Approaching-expiration event
 ↓
EventBridge
 ↓
SNS
```

**2. `DaysToExpiry` → scheduled checking → SNS**

```text
ACM
 ↓
DaysToExpiry metric
 ↓
Scheduled daily check
 ↓
SNS
```

### Exam trap

Do not automatically choose:

```text
AWS Config
Trusted Advisor
Private CA
```

when the question is simply asking for **ACM expiration notification**.

---

### Q2 — Which certificate location?

**Scenario:**
An ALB is deployed in `eu-west-1` and needs an ACM certificate.

**Answer:**

```text
ALB: eu-west-1
→ ACM certificate: eu-west-1
```

---

### Q3 — CloudFront certificate

**Scenario:**
A CloudFront distribution needs an ACM certificate.

**Answer:**

```text
CloudFront
→ ACM in us-east-1
```

---

### Q4 — Multiple domains on one ALB

**Scenario:**
One ALB must serve several unrelated domain names over HTTPS.

**Answer:**

```text
One ALB HTTPS listener
+
Multiple ACM certificates
+
SNI
```

---

### Q5 — Several subdomains

**Scenario:**
A company needs HTTPS for:

```text
api.example.com
www.example.com
shop.example.com
```

**Answer:**

```text
Wildcard certificate
→ *.example.com
```

---

### Q6 — One certificate, many names

**Scenario:**
A single certificate must contain several different domain names.

**Answer:**

```text
SAN certificate
```
