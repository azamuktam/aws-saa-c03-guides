# Section 37B: Certificates & TLS

## The idea

This section covers AWS certificate management and TLS/HTTPS patterns that commonly appear as **specific-use-case questions**.

> **Requirement → keyword → answer**

| Requirement / keyword                                | Answer                                |
| ---------------------------------------------------- | ------------------------------------- |
| TLS / HTTPS certificates                             | **ACM**                               |
| Third-party certificate that must be imported        | **ACM**                               |
| Third-party certificate import alternative           | **IAM certificate store**             |
| CloudFront TLS certificate                           | **ACM in us-east-1**                  |
| ALB TLS certificate                                  | **ACM in the same Region as the ALB** |
| Multiple unrelated domains on one ALB HTTPS listener | **Multiple ACM certificates + SNI**   |
| Multiple subdomains of one domain                    | **Wildcard certificate**              |
| Multiple names in one certificate                    | **SAN certificate**                   |

---

# ACM — AWS Certificate Manager

**ACM = TLS/SSL certificate management for AWS services.**

Common integrations:

* Application Load Balancer (ALB)
* CloudFront
* API Gateway

### Common signal

> **HTTPS / TLS certificate → ACM**

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

You generally cannot export an **ACM public certificate's private key** for direct installation on an EC2 server.

If the private key must be directly accessible by an EC2-hosted application, an ACM public certificate is not the normal solution.

Common AWS-native architecture:

```text
Internet
   ↓
ALB / CloudFront / API Gateway
   ↓
ACM certificate
   ↓
Application
```

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

# ACM certificate types

## ACM-issued public certificate

ACM issues the certificate.

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

> **"Need HTTPS certificate."**

→ **AWS Certificate Manager (ACM)**

> **"Need HTTPS certificate for an Application Load Balancer."**

→ **ACM**

Use the ACM certificate in the **same Region as the ALB**.

> **"Need a TLS certificate for CloudFront."**

→ **ACM in us-east-1**

> **"A company obtained a certificate from a third-party CA and needs to import it into AWS."**

→ **ACM**
→ **IAM certificate store** can also be a valid certificate-store answer depending on the choices.

> **"Multiple unrelated domains share one ALB HTTPS listener."**

→ **Multiple certificates + SNI**

> **"Several subdomains belong to one domain."**

→ **Wildcard certificate**

> **"One certificate must cover multiple domain names."**

→ **SAN certificate**

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

## Imported certificate trap

```text
Third-party certificate
→ Import into ACM
→ ACM does NOT automatically renew it
→ Renew it externally and import the replacement
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

| Keyword                                   | Answer                           |
| ----------------------------------------- | -------------------------------- |
| TLS / SSL certificate                     | **ACM**                          |
| HTTPS                                     | **ACM**                          |
| Third-party certificate import            | **ACM**                          |
| Third-party certificate store alternative | **IAM certificate store**        |
| Imported certificate renewal              | **Manual / re-import**           |
| ALB certificate                           | **ACM in same Region as ALB**    |
| CloudFront certificate                    | **ACM in us-east-1**             |
| Multiple unrelated domains on one ALB     | **SNI + multiple certificates**  |
| Multiple subdomains of one domain         | **Wildcard certificate**         |
| Multiple names in one certificate         | **SAN certificate**              |
| S3 certificate storage                    | **Not an ALB certificate store** |
