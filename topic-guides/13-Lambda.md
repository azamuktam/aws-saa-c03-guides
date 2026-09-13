# Section 13: Lambda

## The idea

**AWS Lambda = serverless compute.**

You upload a **function** containing your code, define what **event** should trigger it, and AWS runs the function for you.

Examples of triggers:

```text
S3 upload
API request
EventBridge schedule
SNS message
SQS message
Kinesis record
DynamoDB Stream record
```

AWS manages:

* provisioning servers
* scaling
* operating systems
* infrastructure maintenance

You manage:

* the function code
* configuration
* permissions
* triggers

Lambda is **event-driven**:

```text
Event
  ↓
Lambda function
  ↓
Code runs
```

Lambda automatically scales based on incoming events.

You pay based on usage/execution rather than keeping servers running continuously.

```text
No invocation
→ No Lambda execution charge

More events
→ Lambda can run more concurrent executions
```

---

# Hard limits — where many exam traps come from

| Limit                          | Value                              |
| ------------------------------ | ---------------------------------- |
| **Maximum execution time**     | **15 minutes**                     |
| Memory                         | **128 MB – 10 GB**                 |
| CPU                            | **Scales with memory**             |
| Ephemeral `/tmp` storage       | Up to **10 GB**                    |
| Deployment package             | **50 MB zipped / 250 MB unzipped** |
| Container image                | Up to **10 GB**                    |
| Default concurrency per Region | **1,000**                          |

### Important trap: maximum execution time

Lambda cannot run for more than **15 minutes per invocation**.

So:

> **"A 2-hour video processing job needs to run as one Lambda invocation."**

→ **Not Lambda**

Possible alternatives:

* AWS Batch
* ECS / Fargate

**Step Functions** can also be used when the overall workflow can be **divided into multiple steps**, with each Lambda invocation staying within the 15-minute limit.

Example:

```text
2-hour workflow

Step 1 → Lambda < 15 min
        ↓
Step 2 → Lambda < 15 min
        ↓
Step 3 → Lambda < 15 min
        ↓
...
```

### Exam signal

> **One Lambda invocation longer than 15 minutes → not Lambda**

---

## CPU scales with memory

Lambda does not give you a separate CPU setting.

Instead:

> **More memory → more CPU**

So:

> **"A CPU-bound Lambda function is too slow. How should you improve its performance?"**

→ **Increase Lambda memory**

For example:

```text
512 MB
 ↓
slow

1024 MB
 ↓
more CPU
 ↓
faster
```

This is a common exam trap because the question may ask about **CPU**, while the correct setting is **memory**.

---

# Three ways Lambda gets invoked

There are three important invocation patterns to understand:

1. **Synchronous**
2. **Asynchronous**
3. **Event Source Mapping**

---

## 1. Synchronous invocation

The caller **waits for Lambda to finish** and receive the response.

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
Response
  ↓
Client
```

Common examples:

* API Gateway → Lambda
* Application directly invoking Lambda

If Lambda fails, the error can be returned to the caller.

### Remember

> **Synchronous = caller waits for the result**

---

# 2. Asynchronous invocation

The event source sends the event to Lambda and **does not wait for the function to finish**.

Typical asynchronous sources:

* S3
* SNS
* EventBridge

Example:

```text
S3
 ↓
Lambda
```

The source does not wait for Lambda's final result.

Lambda handles the event asynchronously.

### What happens when Lambda fails?

Lambda automatically retries the event.

For asynchronous invocation:

```text
Initial attempt
      ↓
Failure
      ↓
Retry
      ↓
Failure
      ↓
Retry
      ↓
Failure
      ↓
Event discarded
```

That means failed asynchronous events can eventually be lost unless you configure additional handling.

### Protect against lost asynchronous events

Use:

* **Dead-letter queue (DLQ)**
* **On-failure destination**

So:

```text
Async Lambda failure
        ↓
DLQ / failure destination
        ↓
Handle the failed event
```

### Important exam trap

> **"Asynchronously invoked Lambda events can disappear after retries."**

→ Configure a **DLQ or on-failure destination**.

### Remember

```text
S3
SNS
EventBridge
   ↓
Asynchronous Lambda invocation
```

---

# 3. Event Source Mapping

For some sources, Lambda **pulls records from the source** rather than the source directly invoking Lambda.

Important examples:

* Amazon SQS
* Amazon Kinesis Data Streams
* DynamoDB Streams

This is called **Event Source Mapping**.

Example:

```text
SQS
 ↓
Lambda polls queue
 ↓
Gets a batch of messages
 ↓
Lambda processes them
```

For streams:

```text
Kinesis / DynamoDB Streams
       ↓
Event Source Mapping
       ↓
Lambda
```

Lambda reads records in **batches**.

### What happens when processing fails?

For SQS:

```text
SQS
 ↓
Lambda
 ↓
Processing fails
 ↓
Message becomes available again
 ↓
Retry
 ↓
Eventually → SQS DLQ
```

For stream-based sources, failed records/batches are retried according to the stream/event-source behavior and configuration.

### Important distinction

```text
S3 / SNS / EventBridge
→ Lambda is PUSHED the event

SQS / Kinesis / DynamoDB Streams
→ Lambda PULLS records through Event Source Mapping
```

### Easy memory

> **S3/SNS/EventBridge = push**

> **SQS/Kinesis/DynamoDB Streams = pull**

---

# Cold starts and concurrency

## Cold start

A **cold start** happens when Lambda needs to initialize a new execution environment before running the function.

Initialization can include:

* starting the runtime
* loading your code
* loading dependencies

This can cause additional latency.

Typical symptom:

> **"The API normally responds quickly, but occasionally has latency spikes after periods of inactivity."**

→ **Cold start**

---

# Provisioned vs Reserved Concurrency

These two are easy to confuse.

## Provisioned Concurrency

**Provisioned Concurrency = keep execution environments pre-initialized and ready to handle requests.**

Purpose:

> **Reduce/eliminate cold-start latency.**

Think:

```text
Provisioned Concurrency
= PRE-WARMED
```

Example:

```text
API
 ↓
Lambda
 ↓
Provisioned Concurrency
 ↓
Already-initialized environments
```

### Exam signal

> **Cold-start latency → Provisioned Concurrency**

---

## Reserved Concurrency

**Reserved Concurrency = reserve a maximum number of concurrent executions for a function.**

Purpose:

> **Prevent a Lambda function from consuming unlimited concurrency and overwhelming a downstream service.**

Example:

```text
Lambda
 ↓
Legacy database
 ↓
Database supports only 50 connections
```

Set:

```text
Reserved Concurrency = 50
```

Now the function cannot run more than 50 concurrent executions.

Think:

```text
Reserved Concurrency
= RESTRICT / CAP
```

### Exam signal

> **Protect a downstream system / cap Lambda concurrency → Reserved Concurrency**

---

## The easiest distinction

```text
Provisioned Concurrency
= PRE-WARM
= reduce cold starts

Reserved Concurrency
= RESTRICT / CAP
= limit concurrent executions
```

---

# Lambda in a VPC

By default, Lambda functions run outside your VPC networking environment.

They can access the public Internet.

They cannot directly access resources that are private inside your VPC.

For example:

```text
Lambda
   ❌
Private RDS
```

If you configure Lambda to run in your VPC, Lambda creates **Elastic Network Interfaces (ENIs)** in your selected subnets.

Then Lambda can access private VPC resources:

```text
Lambda
   ↓
VPC
   ↓
Private RDS
```

### Important trap: VPC Lambda and Internet access

A Lambda function attached to a VPC does **not automatically have Internet access**.

Even placing it in a public subnet does not solve this by itself.

If it needs outbound Internet access, a common solution is:

```text
Lambda in private subnet
        ↓
NAT Gateway
        ↓
Internet
```

For AWS services, **VPC endpoints** may be preferable when available.

So:

```text
Lambda in VPC
+
Private resources
→ ✅

Lambda in VPC
+
Internet access
→ NAT Gateway or appropriate VPC endpoint
```

---

# Lambda + RDS

Lambda can scale to many concurrent executions.

That can create a problem if every invocation opens its own database connection.

Example:

```text
100 Lambda executions
        ↓
100 database connections
        ↓
RDS overwhelmed
```

The common solution is:

> **RDS Proxy**

RDS Proxy maintains a **connection pool** and allows Lambda executions to share database connections more efficiently.

```text
Lambda
  ↓
RDS Proxy
  ↓
Connection pool
  ↓
RDS
```

### Exam signal

> **Lambda + RDS + too many database connections → RDS Proxy**

Lambda + RDS in the same question is a strong signal to consider **RDS Proxy**, especially when the problem involves connection management or database overload.

---

# Lambda permissions and configuration

## Execution role

The Lambda function uses an **IAM execution role**.

That role determines what AWS resources the function can access.

Example:

```text
Lambda
 ↓
Execution role
 ↓
s3:GetObject
dynamodb:PutItem
secretsmanager:GetSecretValue
```

So:

> **"Lambda cannot write to DynamoDB."**

Check the **Lambda execution role** and its IAM permissions.

### Memory

> **Lambda permissions → Execution role**

---

## Lambda Layers

**Lambda Layers = package shared dependencies/libraries separately from the function code.**

Useful when multiple Lambda functions use the same libraries.

Example:

```text
Layer
 ├── shared library
 └── dependency

Function A → uses Layer
Function B → uses Layer
Function C → uses Layer
```

### Signal

> **Shared libraries/dependencies → Lambda Layers**

---

## Container images

Lambda functions can also be deployed using container images.

Maximum container image size:

> **10 GB**

This is useful when dependencies or packaging requirements are too large for a normal ZIP deployment.

Normal Lambda deployment package limits:

```text
ZIP
= 50 MB compressed
= 250 MB uncompressed
```

Container image:

```text
= up to 10 GB
```

### Signal

> **Lambda dependencies/package too large for ZIP → container image**

---

## Environment variables

**Environment variables = configuration stored outside the code.**

Examples:

```text
DATABASE_HOST
API_URL
ENVIRONMENT
TABLE_NAME
```

This makes it easier to use the same function code in different environments.

For example:

```text
Development
→ API_URL=dev.example.com

Production
→ API_URL=api.example.com
```

For sensitive values, use appropriate encryption/secrets management. Lambda environment variables can be encrypted with **AWS KMS**.

### Signal

> **Configuration outside the code → Environment variables**

---

# EventBridge schedule + Lambda

EventBridge can trigger Lambda on a schedule.

Example:

```text
Every night at 2 AM
        ↓
EventBridge
        ↓
Lambda
        ↓
Run cleanup script
```

This is effectively a **serverless cron** pattern.

### Exam signal

> **"Run a task every night at 2 AM without managing servers."**

→ **EventBridge schedule → Lambda**

---

# Lambda@Edge vs CloudFront Functions

Both allow code to run at **CloudFront edge locations**, but they have different capabilities.

|                | **Lambda@Edge**                       | **CloudFront Functions**                    |
| -------------- | ------------------------------------- | ------------------------------------------- |
| Runtime        | Node.js / Python                      | JavaScript                                  |
| Execution time | Up to seconds                         | **Sub-millisecond**                         |
| Network calls  | **Yes**                               | **No**                                      |
| Hooks          | Viewer + origin request/response      | Viewer request/response                     |
| Cost           | Higher                                | Lower; approximately **1/6 the price**      |
| Best for       | More complex edge logic               | Very lightweight edge logic                 |
| Examples       | JWT/auth validation, origin selection | Header manipulation, URL rewrites/redirects |

### Lambda@Edge

Use it when you need more complex logic at the CloudFront edge.

Examples:

* JWT/authentication validation
* origin selection
* logic requiring network calls

Important:

> **Lambda@Edge supports network calls.**

### CloudFront Functions

Use it for very lightweight, high-volume edge operations.

Examples:

* modify headers
* rewrite URLs
* redirects

Important:

> **CloudFront Functions cannot make network calls.**

They are designed for extremely fast execution.

### Simple rule

```text
Needs network calls
or more complex edge logic
→ Lambda@Edge

Simple header / URL manipulation
→ CloudFront Functions
```

---

# Question patterns

> **"90-minute video transcoding job — can it run on Lambda?"**
> → **No — Lambda has a 15-minute maximum. Use AWS Batch / Fargate.**

> **"Can the 90-minute workflow be split into multiple Lambda steps?"**
> → **Yes — Step Functions can orchestrate multiple steps, provided each Lambda invocation stays within the 15-minute limit.**

> **"Generate a thumbnail whenever an image lands in S3."**
> → **S3 event notification → Lambda**

> **"API is normally fast but has latency spikes after idle periods."**
> → **Provisioned Concurrency**

> **"Lambda scaling overwhelms a legacy database that supports only 50 concurrent connections."**
> → **Reserved Concurrency** to cap Lambda concurrency; **RDS Proxy** can additionally manage database connections.

> **"Lambda must query an RDS database in a private subnet."**
> → **Attach Lambda to the VPC + use RDS Proxy when connection management is a concern.**

> **"Lambda is attached to a VPC and still needs Internet access."**
> → **NAT Gateway** for Internet access, or an appropriate **VPC endpoint** when accessing a supported AWS service.

> **"CPU-intensive Lambda function is too slow."**
> → **Increase memory** because CPU scales with memory.

> **"Run a cleanup script every night at 2 AM without servers."**
> → **EventBridge schedule → Lambda**

> **"Validate JWT tokens or call an external authentication service at the edge."**
> → **Lambda@Edge**

> **"Rewrite URLs or add security headers on millions of requests as cheaply as possible."**
> → **CloudFront Functions**

> **"Asynchronously invoked Lambda fails and events eventually disappear."**
> → **DLQ / on-failure destination**

> **"Lambda should process SQS messages."**
> → **Event Source Mapping** — Lambda polls SQS and processes messages in batches.

> **"Lambda needs permission to write to DynamoDB."**
> → **Lambda execution role**

> **"Several Lambda functions share the same libraries."**
> → **Lambda Layers**

> **"Lambda dependencies exceed the normal ZIP package limit."**
> → **Container image**

> **"Lambda needs to run code every night at a specific time."**
> → **EventBridge schedule**

---

# Pocket card

| Keyword                                    | Answer                                     |
| ------------------------------------------ | ------------------------------------------ |
| Job > 15 minutes                           | **Not Lambda → Batch / Fargate**           |
| Workflow divided into <15-min Lambda steps | **Step Functions**                         |
| CPU-bound slow function                    | **Increase memory**                        |
| Cold-start latency                         | **Provisioned Concurrency**                |
| Protect downstream / cap invocations       | **Reserved Concurrency**                   |
| Lambda + RDS connection pressure           | **RDS Proxy**                              |
| Lambda in VPC needs Internet               | **NAT Gateway / appropriate VPC endpoint** |
| Async failures / prevent event loss        | **DLQ / on-failure destination**           |
| SQS / Kinesis / DDB Streams                | **Event Source Mapping → pull / batches**  |
| S3 / SNS / EventBridge                     | **Async push → retries**                   |
| Serverless cron                            | **EventBridge schedule → Lambda**          |
| Shared dependencies                        | **Lambda Layers**                          |
| ZIP package > 250 MB unzipped              | **Container image → 10 GB**                |
| Function permissions                       | **Execution role**                         |
| Configuration outside code                 | **Environment variables**                  |
| Edge + network calls / JWT                 | **Lambda@Edge**                            |
| Edge header/URL manipulation               | **CloudFront Functions**                   |
| Maximum Lambda runtime                     | **15 minutes**                             |
| Memory                                     | **128 MB – 10 GB**                         |
| Default regional concurrency               | **1,000**                                  |
| `/tmp` storage                             | **Up to 10 GB**                            |
| ZIP deployment package                     | **50 MB zipped / 250 MB unzipped**         |
| Container image                            | **10 GB**                                  |

---

# Final memory

```text
Lambda
= SERVERLESS COMPUTE
= EVENT-DRIVEN CODE
```

```text
15 MINUTES
= MAX LAMBDA EXECUTION TIME

MORE MEMORY
= MORE CPU

CPU-BOUND SLOW
→ INCREASE MEMORY
```

```text
SYNCHRONOUS
= CALLER WAITS

ASYNCHRONOUS
= EVENT PUSHED TO LAMBDA

EVENT SOURCE MAPPING
= LAMBDA PULLS FROM SQS / KINESIS / DDB STREAMS
```

```text
S3 / SNS / EventBridge
→ ASYNC PUSH

SQS / Kinesis / DDB Streams
→ EVENT SOURCE MAPPING
```

```text
PROVISIONED CONCURRENCY
= PRE-WARM
= REDUCE COLD STARTS

RESERVED CONCURRENCY
= CAP
= PROTECT DOWNSTREAM SYSTEMS
```

```text
LAMBDA + RDS
→ RDS PROXY

LAMBDA IN VPC + INTERNET
→ NAT GATEWAY / VPC ENDPOINT
```

```text
EXECUTION ROLE
= LAMBDA PERMISSIONS

LAYERS
= SHARED DEPENDENCIES

ENVIRONMENT VARIABLES
= CONFIGURATION

CONTAINER IMAGE
= LARGE PACKAGE / UP TO 10 GB
```

```text
EVENTBRIDGE SCHEDULE
→ SERVERLESS CRON
```

```text
LAMBDA@EDGE
= MORE COMPLEX EDGE LOGIC
= NETWORK CALLS ALLOWED

CLOUDFRONT FUNCTIONS
= LIGHTWEIGHT EDGE LOGIC
= NO NETWORK CALLS
= SUB-MILLISECOND
```

## The key exam distinctions

```text
Lambda
= Serverless compute

Batch
= Long-running batch jobs

Fargate
= Serverless containers

Step Functions
= Orchestrate multiple steps

Provisioned Concurrency
= Avoid cold starts

Reserved Concurrency
= Limit concurrency

RDS Proxy
= Manage Lambda → RDS connections

Event Source Mapping
= Lambda pulls from queues/streams

DLQ / Failure Destination
= Handle failed async events

Lambda@Edge
= Complex edge logic

CloudFront Functions
= Lightweight edge logic
```
