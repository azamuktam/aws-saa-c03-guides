# Section 8: CloudFront & Global Accelerator

## The idea

CloudFront and Global Accelerator both improve application performance for users around the world, but they solve **different problems**.

### CloudFront

**CloudFront = content delivery and HTTP/HTTPS acceleration.**

It can:

* Cache content at edge locations
* Reduce latency for users
* Reduce requests reaching the origin
* Accelerate dynamic HTTP/HTTPS requests even when content is not cacheable
* Protect private S3 content with Origin Access Control (OAC)
* Run custom logic at edge locations using Lambda@Edge

### Global Accelerator

**Global Accelerator = network traffic acceleration and availability.**

It can:

* Provide static anycast IP addresses
* Route traffic onto the AWS global network close to users
* Support TCP and UDP applications
* Improve availability across AWS Regions
* Perform health checks and route new connections to healthy endpoints
* Avoid DNS-based failover delays

### Main distinction

```text
CloudFront
→ Content delivery
→ HTTP/HTTPS
→ Caching
→ Lambda@Edge for edge processing

Global Accelerator
→ Network traffic acceleration
→ TCP/UDP
→ No caching
```

---

# CloudFront

**Amazon CloudFront is a Content Delivery Network (CDN).**

It distributes content through AWS edge locations around the world.

Users connect to the CloudFront edge location that serves their request, and CloudFront retrieves content from the origin when necessary.

### Origins

CloudFront can use origins such as:

* Amazon S3
* Application Load Balancer
* EC2/custom HTTP server
* Other HTTP origins

Example:

```text
User
  │
  ▼
CloudFront
  │
  ▼
Origin
```

### Main use cases

* Static websites
* Images and files
* Videos
* Software downloads
* APIs
* HTTP/HTTPS applications
* Dynamic content acceleration

---

# CloudFront caching

CloudFront can cache content at edge locations.

Example:

```text
First request
User → CloudFront → Origin
                    ↓
                 Content
                    ↓
                 Cached

Later requests
User → CloudFront
          ↓
       Cache hit
```

When CloudFront serves a cache hit, the request does not need to retrieve the object from the origin again.

### Why caching matters

Caching can:

* reduce latency
* reduce origin requests
* reduce origin load
* reduce data transfer from the origin

### Exam clue

> **"Reduce the load on the origin server behind CloudFront."**

→ **Increase the cache hit ratio / use an appropriate longer TTL when suitable**

---

# TTL

**TTL (Time To Live)** determines how long CloudFront considers an object fresh in the cache before it needs to check the origin again.

In general:

```text
Longer TTL
→ Objects remain cached longer
→ Fewer origin requests
→ Lower origin load
```

However, longer TTLs also mean content changes may take longer to appear unless you use versioning or invalidation.

---

# CloudFront Invalidation

An **invalidation** tells CloudFront to remove cached objects before their normal TTL expires.

Example:

```text
Invalidation
→ /images/logo.png
```

or:

```text
Invalidation
→ /*
```

### Important

Invalidations can incur charges after the included allowance.

A common alternative is **versioned filenames**:

```text
app.js
```

becomes:

```text
app-v2.js
```

CloudFront treats the new filename as a new object.

### Exam clue

> **"The company needs to force CloudFront to refresh cached content before the TTL expires."**

→ **CloudFront invalidation**

---

# CloudFront dynamic content

CloudFront is not limited to static content.

Dynamic HTTP/HTTPS requests can also benefit from CloudFront because the user's request enters the AWS network at an edge location and can use optimized connectivity to the origin.

However:

> **Caching is not required for CloudFront to provide acceleration.**

### Important distinction

```text
Static/cacheable content
→ CloudFront caching + acceleration

Dynamic/non-cacheable HTTP content
→ CloudFront can still provide network acceleration
```

---

# Lambda@Edge

**Lambda@Edge** lets you run custom Lambda code at **CloudFront edge locations**.

It is used when the application needs to **process or modify requests/responses at the edge**.

Typical uses:

* Modify viewer requests or responses
* Modify origin requests or responses
* Authentication/authorization
* Redirects
* Header manipulation
* Personalization
* Custom request/response processing

Example:

```text
User
  │
  ▼
CloudFront Edge Location
  │
  ▼
Lambda@Edge
  │
  ▼
Origin
```

### Exam clue

> **"Execute custom application logic at CloudFront edge locations."**

→ **Lambda@Edge**

---

# CloudFront + Lambda@Edge + Kinesis

For globally distributed applications that need **real-time streaming data processing** together with edge processing:

```text
Users
  │
  ▼
CloudFront
  │
  ▼
Lambda@Edge
  │
  ▼
Kinesis
  │
  ▼
S3
```

Use:

* **CloudFront** → global HTTP/HTTPS delivery
* **Lambda@Edge** → execute custom logic at the edge
* **Kinesis** → real-time streaming data
* **S3** → durable storage

Typical data:

* Clickstreams
* User activity
* Application events
* Real-time telemetry

### Exam clue

> **"Process user activity in real time and execute processing close to global users."**

→ **CloudFront + Lambda@Edge + Kinesis**

### Important distinction

```text
Route 53 latency-based routing
→ routes users to an endpoint

Lambda@Edge
→ executes code at CloudFront edge locations

Kinesis
→ processes real-time streaming data
```

---

# CloudFront Origin Access Control (OAC)

**Origin Access Control (OAC)** allows CloudFront to securely access an S3 bucket while keeping the bucket private.

With OAC:

```text
User
  │
  ▼
CloudFront
  │
  │ authenticated request
  ▼
Private S3 bucket
```

The S3 bucket policy grants the CloudFront service permission to access the bucket.

Users can access the content through CloudFront without needing direct public access to the S3 bucket.

### OAC and SigV4

OAC can use **AWS Signature Version 4 (SigV4)** to sign requests from CloudFront to S3.

The recommended configuration is to have CloudFront sign requests to the S3 origin.

### Important exam clue

> **"S3 content must be accessible only through CloudFront and the S3 bucket must remain private."**

→ **CloudFront + OAC**

### OAI vs OAC

**OAI (Origin Access Identity)** is the older mechanism.

**OAC (Origin Access Control)** is the recommended approach for new S3 origins.

OAC supports capabilities that OAI does not, including:

* S3 buckets in all AWS Regions
* SSE-KMS
* authenticated dynamic requests such as `PUT`, `POST`, and `DELETE`

### Important S3 condition

OAC is designed for a **regular S3 bucket origin**, not an S3 static website endpoint.

### Memory rule

```text
Private S3 bucket + CloudFront
→ OAC
```

---

# CloudFront signed URLs

**Signed URLs** provide temporary access to CloudFront content.

They are commonly used when access needs to be controlled for a **specific file or resource**.

Example:

```text
Customer
   ↓
Signed URL
   ↓
Specific report/video/file
```

Typical use cases:

* One private download
* One protected video
* Temporary access to a specific object

### Exam clue

> **"Give a customer temporary access to one private file."**

→ **Signed URL**

---

# CloudFront signed cookies

**Signed cookies** also provide temporary access to private CloudFront content, but they are useful when a user needs access to **multiple files**.

Example:

```text
Subscriber
    ↓
Signed cookie
    ↓
Private video library
    ├── video1
    ├── video2
    ├── video3
    └── video4
```

Typical use cases:

* HLS video streams consisting of many files
* Premium content libraries
* Multiple private resources without changing the URLs

### Exam clue

> **"Subscribers need access to many files in a private content library."**

→ **Signed Cookies**

### Signed URL vs Signed Cookies

|               | Signed URL             | Signed Cookies           |
| ------------- | ---------------------- | ------------------------ |
| Typical scope | Specific file/resource | Multiple files/resources |
| URL changes   | Yes                    | No                       |
| Example       | One private download   | Premium video library    |

---

# CloudFront Geo Restriction

**Geo Restriction** controls content access at the **country level**.

You can use:

* **Allow list** → only selected countries can access content
* **Block list** → selected countries cannot access content

Typical use case:

> Licensing agreements prohibit distribution in certain countries.

→ **CloudFront Geo Restriction**

### Important

CloudFront's built-in geographic restriction works at the **country level**.

### Exam clue

> **"Block users from country X from accessing the content."**

→ **CloudFront Geo Restriction**

---

# CloudFront custom SSL/TLS certificate

For a custom domain on CloudFront, the ACM certificate used by CloudFront must be in:

> **`us-east-1` (US East — N. Virginia)**

This applies even if the application's origin is in another AWS Region.

Example:

```text
Application origin
→ eu-west-1

CloudFront ACM certificate
→ us-east-1
```

### Exam clue

> **"Attach a custom ACM certificate to CloudFront."**

→ **ACM certificate in `us-east-1`**

---

# CloudFront Field-Level Encryption

**Field-level encryption** encrypts selected fields in an HTTPS request at the CloudFront edge.

Example:

```text
POST request

name
email
credit_card_number  ← encrypted
```

The sensitive field remains encrypted as it travels through the application stack.

Only the application that has the corresponding private key can decrypt the field.

### Important

Field-level encryption is for **specific fields**, not necessarily the entire request.

### Exam clue

> **"Encrypt a credit card number at the edge so intermediate application components cannot read it."**

→ **CloudFront Field-Level Encryption**

---

# CloudFront Origin Groups

An **origin group** contains:

* Primary origin
* Secondary origin

If the primary origin returns a configured failure response, CloudFront can retry the request against the secondary origin.

Example:

```text
CloudFront
    │
    ▼
Primary origin
    │
    ✕ failure
    │
    ▼
Secondary origin
```

Typical use case:

> High availability through origin failover.

CloudFront origin failover is configured for specific HTTP status codes and applies to viewer requests using supported methods such as `GET`, `HEAD`, and `OPTIONS`.

### Exam clue

> **"CloudFront should automatically use a secondary origin when the primary origin fails."**

→ **CloudFront Origin Group**

---

# CloudFront Price Classes

**Price Class** determines which CloudFront edge locations are used by your distribution.

This is primarily a **cost optimization setting**.

In general:

```text
More edge locations
→ broader geographic coverage
→ potentially lower latency
→ higher cost

Fewer edge locations
→ lower cost
→ potentially higher latency for some users
```

### Exam clue

> **"Reduce CloudFront costs and accept potentially higher latency in some geographic regions."**

→ **Use a lower CloudFront Price Class**

Price Class controls the locations used for CloudFront delivery; it does not determine which origin CloudFront uses.

---

# Global Accelerator

**AWS Global Accelerator** improves the performance and availability of applications by providing static anycast IP addresses and routing traffic through the AWS global network.

It does **not cache application content**.

### Typical architecture

```text
User
 │
 ▼
Global Accelerator
 │
 ▼
AWS global network
 │
 ▼
Application endpoint
```

Supported standard accelerator endpoints include:

* Application Load Balancers
* Network Load Balancers
* EC2 instances
* Elastic IP address endpoints

---

# Global Accelerator Static IP Addresses

A standard Global Accelerator provides **two static IPv4 addresses**.

These addresses are **Anycast** addresses.

The same addresses can be used by users around the world.

Traffic enters the AWS global network at a suitable edge location and is then routed toward the application endpoint.

### Why static IPs matter

This is especially useful when clients require fixed IP addresses for:

* Firewall allow-listing
* Network ACLs
* Corporate security policies
* Partner integrations

### Exam clue

> **"Customers require a small set of fixed IP addresses to whitelist."**

→ **Global Accelerator**

---

# Global Accelerator protocols

Global Accelerator can accelerate:

* **TCP**
* **UDP**

This makes it suitable for applications that are not limited to HTTP/HTTPS.

Examples:

* Gaming
* VoIP
* IoT
* MQTT
* Other TCP/UDP applications

### Exam clue

> **"Global application uses UDP and requires lower latency."**

→ **Global Accelerator**

---

# Global Accelerator does not cache

Unlike CloudFront:

> **Global Accelerator does not cache application content.**

It accelerates the **network path** to the application.

```text
CloudFront
→ caching + HTTP/HTTPS acceleration

Global Accelerator
→ network acceleration + availability
```

---

# Global Accelerator health checks and failover

A standard Global Accelerator continuously checks the health of application endpoints.

Health checks can use:

* TCP
* HTTP
* HTTPS

When an endpoint becomes unhealthy, Global Accelerator can route **new connections** to healthy endpoints.

This is different from DNS-based failover because clients continue using the same Global Accelerator IP addresses.

### Important distinction

Do not memorize:

> "Global Accelerator always fails over in exactly 30 seconds."

The health-check interval and threshold are configurable, so the detection and failover time depends on the configuration.

### Exam clue

> **"Multi-Region application needs fast failover without waiting for DNS caches to expire."**

→ **Global Accelerator**

---

# CloudFront vs Global Accelerator

| Feature                                 | **CloudFront**                               | **Global Accelerator**                                  |
| --------------------------------------- | -------------------------------------------- | ------------------------------------------------------- |
| Main purpose                            | Content delivery and HTTP/HTTPS acceleration | Network traffic acceleration and availability           |
| Main layer                              | HTTP/HTTPS                                   | TCP/UDP                                                 |
| Caching                                 | ✅ Yes                                        | ❌ No                                                    |
| Static anycast IPs                      | ❌                                            | ✅ Two static IPv4 addresses                             |
| Static content                          | ✅                                            | ❌                                                       |
| Dynamic HTTP/HTTPS                      | ✅                                            | ✅                                                       |
| UDP                                     | ❌                                            | ✅                                                       |
| Global application failover             | Limited/origin failover                      | ✅                                                       |
| DNS cache dependency for accelerator IP | —                                            | ❌                                                       |
| Typical use                             | Websites, videos, downloads, APIs            | Gaming, VoIP, IoT, fixed IPs, multi-Region applications |

---

# CloudFront vs S3 Transfer Acceleration vs Global Accelerator

This is a common SAA distinction.

### CloudFront

Used when users are **retrieving HTTP/HTTPS content**.

```text
Users worldwide
      ↓
CloudFront
      ↓
Origin
```

Typical:

> Website, video, images, downloads, APIs

---

### S3 Transfer Acceleration

Used when users are **transferring files to or from S3 over long distances** and need improved transfer performance.

It uses CloudFront's globally distributed edge locations to route data to S3 over an optimized network path.

Typical clue:

> **"Users around the world upload large files to a central S3 bucket."**

→ **S3 Transfer Acceleration**

---

### Global Accelerator

Used for applications that need **network-level acceleration**, including TCP/UDP workloads.

Typical clue:

> **"Global users access a TCP/UDP application."**

→ **Global Accelerator**

---

# The three-way direction and protocol trap

Read the **direction and protocol** carefully:

```text
Global users retrieve HTTP/HTTPS content
→ CloudFront

Global users transfer files to/from S3
→ S3 Transfer Acceleration

Global users use a TCP/UDP application
→ Global Accelerator
```

---

# CloudFront vs Global Accelerator vs S3 Transfer Acceleration

| Requirement                                | Answer                       |
| ------------------------------------------ | ---------------------------- |
| Global website/content delivery            | **CloudFront**               |
| Cache static content near users            | **CloudFront**               |
| Dynamic HTTP/HTTPS acceleration            | **CloudFront**               |
| Private S3 origin behind CloudFront        | **CloudFront + OAC**         |
| Upload large files globally to S3          | **S3 Transfer Acceleration** |
| TCP application                            | **Global Accelerator**       |
| UDP application                            | **Global Accelerator**       |
| Fixed global IP addresses                  | **Global Accelerator**       |
| Multi-Region failover without changing DNS | **Global Accelerator**       |

---

# Question patterns

> **"Serve a static website to users worldwide with low latency; the S3 bucket must not be publicly accessible."**

→ **CloudFront + S3 origin + OAC**

---

> **"A private S3 bucket should only be accessible through a CloudFront distribution."**

→ **CloudFront OAC**

---

> **"Paying subscribers should access many videos in a premium library."**

→ **CloudFront Signed Cookies**

---

> **"Send a customer a temporary link to download one private report."**

→ **CloudFront Signed URL**

---

> **"A multiplayer game uses UDP and has high latency for global users."**

→ **Global Accelerator**

---

> **"Enterprise clients require fixed IP addresses to whitelist."**

→ **Global Accelerator**

---

> **"A multi-Region application needs traffic redirected to healthy endpoints without waiting for DNS TTL/cache changes."**

→ **Global Accelerator**

---

> **"Licensing restrictions prevent users from certain countries from accessing content."**

→ **CloudFront Geo Restriction**

---

> **"Reduce requests reaching the origin."**

→ **Improve CloudFront caching / increase appropriate TTL**

---

> **"Force CloudFront to retrieve updated files before the TTL expires."**

→ **CloudFront Invalidation**

---

> **"Users around the world upload large files to one S3 bucket."**

→ **S3 Transfer Acceleration**

---

> **"Attach a custom ACM certificate to CloudFront."**

→ **ACM certificate in `us-east-1`**

---

> **"CloudFront should use another origin when the primary origin fails."**

→ **CloudFront Origin Group**

---

> **"Reduce CloudFront costs and accept potentially higher latency in some regions."**

→ **Lower CloudFront Price Class**

---

> **"Encrypt a credit card field at the CloudFront edge so it remains encrypted through the application stack."**

→ **CloudFront Field-Level Encryption**

---

> **"Process global user activity in real time at the edge and stream the data for processing."**

→ **CloudFront + Lambda@Edge + Kinesis**

---

# Important SAA traps

## OAC vs Signed URL

These solve different problems.

```text
OAC
→ CloudFront → private S3 origin

Signed URL
→ controls which viewer can access CloudFront content
```

You can use both together.

For example:

```text
User
 │
 │ signed URL
 ▼
CloudFront
 │
 │ OAC
 ▼
Private S3
```

The signed URL controls viewer access, while OAC protects the S3 origin.

---

## OAI vs OAC

```text
OAC
→ recommended

OAI
→ legacy
```

When the question presents both for a new CloudFront + S3 design:

→ **OAC**

---

## CloudFront vs Global Accelerator

```text
HTTP/HTTPS content
→ CloudFront

TCP/UDP application
→ Global Accelerator
```

If caching is mentioned:

→ **CloudFront**

If fixed static IPs are required:

→ **Global Accelerator**

If UDP is required:

→ **Global Accelerator**

---

## CloudFront vs Lambda@Edge

```text
CloudFront
→ delivers/caches HTTP/HTTPS content

Lambda@Edge
→ executes custom code at CloudFront edge locations
```

If the question says:

> **"Execute custom logic at the edge."**

→ **Lambda@Edge**

If it says:

> **"Real-time streaming data."**

→ **Kinesis**

If both are required:

→ **CloudFront + Lambda@Edge + Kinesis**

---

## CloudFront vs S3 Transfer Acceleration

```text
Download/view web content
→ CloudFront

Upload/transfer files to S3
→ S3 Transfer Acceleration
```

---

## Origin Group vs Geo Restriction

```text
Primary origin fails
→ Origin Group

Country restrictions
→ Geo Restriction
```

---

## Price Class vs Origin

```text
Reduce CloudFront cost
→ Price Class

Change where CloudFront gets content from
→ Origin configuration
```

---

# Pocket card

| Keyword in question                               | Answer                                       |
| ------------------------------------------------- | -------------------------------------------- |
| Global static content, low latency                | **CloudFront**                               |
| HTTP/HTTPS content delivery                       | **CloudFront**                               |
| Execute custom code at CloudFront edge            | **Lambda@Edge**                              |
| Real-time streaming data                          | **Kinesis**                                  |
| Edge processing + real-time streaming             | **CloudFront + Lambda@Edge + Kinesis**       |
| Private S3 bucket through CloudFront              | **CloudFront OAC**                           |
| OAI vs OAC                                        | **OAC**                                      |
| One private file                                  | **Signed URL**                               |
| Many private files                                | **Signed Cookies**                           |
| Block/allow countries                             | **Geo Restriction**                          |
| CloudFront custom certificate                     | **ACM in `us-east-1`**                       |
| Encrypt specific sensitive fields                 | **Field-Level Encryption**                   |
| Reduce origin load                                | **Improve caching / appropriate longer TTL** |
| Force-refresh cached content                      | **Invalidation**                             |
| Primary/secondary CloudFront origins              | **Origin Group**                             |
| Reduce CloudFront cost                            | **Price Class**                              |
| UDP application                                   | **Global Accelerator**                       |
| TCP application requiring global acceleration     | **Global Accelerator**                       |
| Fixed IPs for allow-listing                       | **Global Accelerator**                       |
| Multi-Region traffic failover without DNS changes | **Global Accelerator**                       |
| Global file transfers to S3                       | **S3 Transfer Acceleration**                 |

---
