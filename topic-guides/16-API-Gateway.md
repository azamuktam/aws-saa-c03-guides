# Section 16: API Gateway

## Big picture

**API Gateway = the entry point for HTTP APIs.**

It sits between the client and your backend.

```text
Client
  ↓
API Gateway
  ↓
Backend
```

The backend could be:

* Lambda
* an HTTP application
* another AWS service

API Gateway can handle things such as:

* authentication
* throttling
* routing
* caching
* request validation
* monitoring

### Common serverless architecture

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
DynamoDB
```

Use this pattern when you need a **serverless HTTP API** without managing servers.

---

# 1. API Gateway API Types

There are three important types:

| Type              | Main purpose                    | Remember                   |
| ----------------- | ------------------------------- | -------------------------- |
| **REST API**      | Full-featured API               | More features              |
| **HTTP API**      | Simple, cheaper API             | Lower cost, fewer features |
| **WebSocket API** | Real-time two-way communication | Persistent connection      |

---

## REST API

REST API has the most features.

Important features include:

* API keys
* Usage plans
* Caching
* WAF integration
* Request validation

### Use REST API when

The question specifically needs one of these features.

Example:

> "Customers have Basic, Pro, and Enterprise API plans with different request limits."

→ **REST API + API Keys + Usage Plans**

### Remember

> **Need advanced API Gateway features → REST API**

---

## HTTP API

HTTP API is simpler and cheaper.

It is good for:

* simple HTTP APIs
* Lambda integrations
* HTTP backend integrations
* JWT authentication

### Use it when

> "Expose a Lambda function through HTTP at the lowest cost."

→ **HTTP API**

### Important limitation

HTTP API does **not** have all the advanced REST API features.

For example, questions requiring:

* API keys
* usage plans
* API Gateway caching

→ use **REST API**, not HTTP API.

### Remember

> **Simple + cheap → HTTP API**

---

## WebSocket API

WebSocket API provides a **persistent connection** between the client and server.

Unlike normal HTTP requests, the server can send data to the client when needed.

```text
Client ←────────→ Server
       persistent
       connection
```

### Use it for

* real-time chat
* live notifications
* live dashboards
* real-time updates

### Remember

> **Real-time two-way communication → WebSocket API**

---

# REST vs HTTP vs WebSocket

| Requirement                      | Answer            |
| -------------------------------- | ----------------- |
| Advanced API Gateway features    | **REST API**      |
| Cheapest simple HTTP API         | **HTTP API**      |
| Simple Lambda + JWT              | **HTTP API**      |
| API keys / usage plans           | **REST API**      |
| API Gateway caching              | **REST API**      |
| Real-time chat                   | **WebSocket API** |
| Server pushes updates to clients | **WebSocket API** |

---

# 2. API Gateway Endpoint Types

This tells you **where the API is accessed from**.

| Endpoint           | Use when                               |
| ------------------ | -------------------------------------- |
| **Edge-Optimized** | Clients are globally distributed       |
| **Regional**       | Clients are mainly in one region       |
| **Private**        | API must only be accessible from a VPC |

---

## Edge-Optimized

Use when clients are distributed around the world.

```text
Users around the world
        ↓
Edge-Optimized API
        ↓
AWS Region
```

The API uses CloudFront's edge network.

### Remember

> **Global clients → Edge-Optimized**

---

## Regional

The API is accessed directly in its AWS Region.

Use it when:

* clients are mainly in the same region
* you want to use your **own CloudFront distribution**

### Remember

> **Same region / own CloudFront → Regional**

---

## Private

The API is accessible only from inside a VPC.

```text
VPC
 ↓
Interface VPC Endpoint
 ↓
Private API Gateway
```

It does not need to be publicly accessible over the internet.

### Remember

> **VPC-only API → Private endpoint**

---

# 3. API Authentication

The exam commonly gives you three choices:

* IAM
* Cognito
* Lambda Authorizer

The easiest way to remember them:

> **IAM = AWS callers**
> **Cognito = application users**
> **Lambda Authorizer = custom authentication**

---

## IAM Authorization

Use IAM when the caller is an AWS identity.

Examples:

* EC2
* Lambda
* AWS users
* AWS services

Requests are signed using **AWS Signature Version 4 (SigV4)**.

### Example

> "An EC2 instance needs to securely call an API."

→ **IAM authentication**

### Remember

> **AWS identity → IAM**

---

## Cognito User Pool Authorizer

Use Cognito when **real application users sign in**.

Example:

```text
User
 ↓
Cognito User Pool
 ↓
JWT token
 ↓
API Gateway
```

The application user gets a JWT, and API Gateway can validate it.

### Example

> "Users of a mobile application must log in before accessing the API."

→ **Cognito User Pool**

### Remember

> **App users → Cognito**

---

## Lambda Authorizer

Use this when authentication requires **custom logic**.

For example:

* custom tokens
* third-party identity systems
* legacy authentication
* unusual authorization rules

```text
Client
 ↓
API Gateway
 ↓
Lambda Authorizer
 ↓
Allow / Deny
```

### Example

> "The company already uses a custom token format that API Gateway must validate."

→ **Lambda Authorizer**

### Remember

> **Custom authentication logic → Lambda Authorizer**

---

# 4. Throttling

API Gateway can limit how many requests clients can send.

If clients send too many requests, API Gateway can return:

```text
HTTP 429
Too Many Requests
```

### Remember

> **429 → throttling**

The commonly tested account-level default is **10,000 requests/second per Region**, though AWS can change quotas and account limits can be adjusted.

---

# 5. API Keys and Usage Plans

These are mainly an **API customer management** feature.

Imagine a company sells an API:

```text
Basic
→ 100 requests/minute

Pro
→ 1,000 requests/minute

Enterprise
→ higher limit
```

You can use:

**API Keys + Usage Plans**

These are associated with **REST APIs**.

### Example

> "A SaaS company wants different API request limits for different customers."

→ **REST API + API Keys + Usage Plans**

### Remember

> **Customer API tiers → Usage Plans**

---

# 6. API Gateway Caching

API Gateway can cache responses.

Suppose many users request the same data:

```text
Client
  ↓
API Gateway
  ↓
Cache
```

If the response is already cached, API Gateway can return it without calling the backend again.

### Benefits

* fewer backend requests
* lower latency
* reduced backend load

The default API Gateway cache TTL commonly tested is **300 seconds**.

### Example

> "Repeated requests are hitting the backend unnecessarily."

→ **Enable API Gateway caching**

---

# 7. API Gateway Timeout

This is an important exam limit.

API Gateway has a **29-second integration timeout**.

That means:

```text
Client
  ↓
API Gateway
  ↓
Backend
```

The backend cannot simply take several minutes while the client waits through API Gateway.

### Example

Your backend job takes:

```text
2 minutes
```

You cannot solve this by simply increasing API Gateway's timeout beyond its limit.

Instead, make the operation **asynchronous**.

```text
Client
  ↓
API Gateway
  ↓
202 Accepted
  ↓
SQS / Step Functions
  ↓
Worker
  ↓
Long-running job
```

The API responds quickly, while the actual work happens in the background.

### Remember

> **Long job → asynchronous processing**

Common pattern:

**API Gateway → SQS → Lambda**

or

**API Gateway → Step Functions**

---

# 8. CORS

CORS matters mainly when a **browser** calls an API from a different domain.

Example:

```text
Frontend:
https://example.com

API:
https://api.example.com
```

The browser may block the request unless the API allows the cross-origin request.

### Example

> "Browser JavaScript is getting a cross-origin error when calling the API."

→ **Configure CORS**

### Remember

> **Browser + different origin + blocked request → CORS**

---

# 9. Common Exam Questions

### "Build a serverless REST API"

Typical architecture:

```text
API Gateway
    ↓
Lambda
    ↓
DynamoDB
```

---

### "Customers have different API request limits"

→ **REST API + API Keys + Usage Plans**

---

### "Users of a mobile/web application must log in"

→ **Cognito User Pool**

---

### "AWS resources need to call the API"

→ **IAM / SigV4**

---

### "Company uses a custom authentication system"

→ **Lambda Authorizer**

---

### "Expose a simple Lambda API at the lowest cost"

→ **HTTP API**

---

### "Real-time chat application"

→ **WebSocket API**

---

### "API must only be accessible from inside the VPC"

→ **Private API Gateway endpoint + Interface VPC Endpoint**

---

### "Clients are receiving HTTP 429"

→ **Throttling**

---

### "Repeated requests are unnecessarily hitting the backend"

→ **API Gateway caching**

---

### "Backend takes 2 minutes and API Gateway times out"

→ **Asynchronous processing**

For example:

```text
API Gateway
    ↓
SQS
    ↓
Lambda
```

Do not try to make the API Gateway request wait 2 minutes.

---

### "Browser gets a cross-origin error"

→ **CORS**

---

# Final Memory Card

## API types

```text
REST
= Full features

HTTP
= Simple + cheap

WebSocket
= Real-time two-way communication
```

## Endpoint types

```text
Global clients
→ Edge-Optimized

Regional clients / own CloudFront
→ Regional

VPC only
→ Private
```

## Authentication

```text
AWS callers
→ IAM

Application users
→ Cognito

Custom authentication
→ Lambda Authorizer
```

## Other important keywords

```text
API keys / Usage Plans
→ REST API

429
→ Throttling

Repeated requests / reduce backend calls
→ Caching

Browser cross-origin error
→ CORS

Long-running backend job
→ Async processing

Real-time communication
→ WebSocket
```

## The simplest way to think about API Gateway

```text
API Gateway
│
├── What type?
│   ├── REST
│   ├── HTTP
│   └── WebSocket
│
├── Where?
│   ├── Edge-Optimized
│   ├── Regional
│   └── Private
│
├── Who can call?
│   ├── IAM
│   ├── Cognito
│   └── Lambda Authorizer
│
└── What traffic/features?
    ├── Throttling
    ├── API Keys / Usage Plans
    ├── Caching
    └── CORS
```
