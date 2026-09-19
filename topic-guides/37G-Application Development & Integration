# Section 37G: Application Development & Integration

## The idea

These are smaller AWS application-development services that commonly appear in SAA questions as **specific application requirements**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Long-running batch jobs
→ AWS Batch

GraphQL
→ AWS AppSync

GraphQL + real-time subscriptions
→ AWS AppSync

Quick web/mobile application development
→ AWS Amplify

Application needs to send email
→ Amazon SES
```

---

# AWS Batch

**AWS Batch = run large batch jobs without managing the job scheduler yourself.**

It is designed for **batch computing workloads** where jobs can run for a long time and may need significant compute resources.

AWS Batch manages the scheduling and provisioning of compute resources needed to run batch jobs.

### Signal

> **Long-running batch jobs / containers → AWS Batch**

---

## Typical pattern

```text
Thousands of jobs
      ↓
   AWS Batch
      ↓
Compute capacity
      ↓
Run batch workloads
```

AWS Batch can provision and manage compute resources for the jobs rather than requiring you to build and maintain your own batch scheduling system.

---

## Example

> "A company needs to run thousands of long-running data-processing jobs and does not want to manage the job scheduler."

→ **AWS Batch**

---

# AWS Batch vs Lambda

A common SAA comparison is between AWS Batch and Lambda.

### Lambda

```text
Lambda
= short event-driven functions
```

Lambda is designed for individual function executions triggered by events.

### AWS Batch

```text
AWS Batch
= long-running batch workloads
```

It is appropriate for workloads where jobs may run for significantly longer periods and require dedicated compute capacity.

### Simple memory

```text
Lambda
= short event-driven function

AWS Batch
= long-running batch job
```

---

## Example

> "A company has thousands of computationally intensive jobs that may run for hours."

→ **AWS Batch**

The question is describing a batch-computing workload rather than a typical Lambda function.

---

# AWS Batch with Containers

AWS Batch is commonly used for containerized batch workloads.

A typical architecture can look like:

```text
Job submissions
      ↓
AWS Batch
      ↓
Containerized jobs
      ↓
EC2 / Spot capacity
```

The important SAA signal remains:

> **Large-scale / long-running batch processing → AWS Batch**

---

# AWS Batch and Spot Instances

AWS Batch can use suitable **Spot capacity** to reduce the cost of batch workloads.

This can be useful when jobs can tolerate interruptions.

### Memory

```text
Batch workload
+
Cost optimization
+
Interruptible jobs
→ Spot capacity can be appropriate
```

The exact compute environment can vary, but you generally do not need to memorize the infrastructure details for SAA.

---

# AWS AppSync

**AWS AppSync = managed GraphQL API service.**

Use it when the question says:

* GraphQL
* real-time subscriptions
* real-time application updates
* offline synchronization for applications

### Signal

> **GraphQL → AppSync**

---

## Typical pattern

```text
Mobile / Web application
          ↓
       AppSync
          ↓
      Data sources
```

AppSync provides a managed GraphQL API layer between the application and backend data sources.

---

# GraphQL

GraphQL allows clients to request the data they need through a GraphQL API.

So when the SAA question explicitly says:

> **GraphQL**

Think:

→ **AWS AppSync**

This is one of the strongest service keywords to memorize.

---

# AppSync real-time subscriptions

AppSync supports **real-time subscriptions**, which are useful when clients need to receive updates as backend data changes.

Example:

```text
User A updates data
       ↓
    AppSync
       ↓
Real-time update
       ↓
User B
```

### Signal

> **GraphQL + real-time subscriptions → AppSync**

---

# AppSync offline synchronization

AppSync can also support **offline synchronization** for applications.

This is particularly useful for mobile or other clients that may temporarily lose connectivity.

The application can continue working with local data and synchronize changes when connectivity is restored.

### Signal

> **GraphQL + offline synchronization → AppSync**

---

# Example

> "A mobile application needs a managed GraphQL API with real-time subscriptions."

→ **AWS AppSync**

---

> "A mobile application needs GraphQL and offline data synchronization."

→ **AWS AppSync**

---

# AppSync memory

```text
AppSync
= GRAPHQL
= REAL-TIME SUBSCRIPTIONS
= OFFLINE SYNCHRONIZATION
```

The strongest keyword remains:

> **GraphQL → AppSync**

---

# AWS Amplify

**AWS Amplify = tools for quickly building and deploying web/mobile applications.**

It helps developers connect frontend applications with AWS backend services.

Amplify is designed to make it easier to build and deploy application frontends and integrate them with AWS services.

### Signal

> **Quick full-stack web/mobile development → Amplify**

---

# Typical use case

A developer wants to quickly build a web or mobile application and connect it to AWS services without manually configuring every backend component.

```text
Web / Mobile frontend
        ↓
     Amplify
        ↓
AWS backend services
```

Amplify can help with application development and deployment workflows.

---

## Example

> "A development team wants to quickly build and deploy a web and mobile application using AWS backend services."

→ **AWS Amplify**

---

# Amplify vs AppSync

These services can appear together, but they solve different problems.

```text
AppSync
= managed GraphQL API

Amplify
= tools for building and deploying web/mobile applications
```

They can also be used together:

```text
Web / Mobile app
      ↓
   Amplify
      ↓
   AppSync
      ↓
  Data sources
```

### Important distinction

> **GraphQL → AppSync**

> **Quick web/mobile application development → Amplify**

---

# Amazon SES

**Amazon SES (Simple Email Service) = send application emails.**

It is designed for sending email from applications.

Common examples include:

* receipts
* verification emails
* notifications
* marketing emails

### Signal

> **Application needs to send email → SES**

---

# SES examples

### Verification email

```text
User signs up
     ↓
Application
     ↓
SES
     ↓
Verification email
```

### Receipt

```text
Purchase
   ↓
Application
   ↓
SES
   ↓
Email receipt
```

### Notification

```text
Application event
      ↓
     SES
      ↓
Email notification
```

---

# SES vs SNS

This is an important SAA distinction.

```text
SNS
= notifications / pub-sub

SES
= email sending
```

### SNS

SNS is primarily used for:

* pub/sub messaging
* notifications
* fan-out
* delivering messages to supported subscribers

Example:

```text
Application
    ↓
   SNS
   ├── SQS
   ├── Lambda
   └── other subscribers
```

### SES

SES is specifically for sending email.

```text
Application
    ↓
   SES
    ↓
  Email
```

### Memory

> **SNS = notifications / pub-sub**

> **SES = email**

---

# SES vs SNS exam trap

A question may say:

> "A web application needs to send an email to a customer after a purchase."

→ **SES**

Not SNS merely because the question says "notification."

The actual delivery mechanism is **email**.

---

> "A service needs to publish a notification to multiple subscribers."

→ **SNS**

The key is the required communication pattern.

---

# Application Service Comparison

| Service         | What it does                                    | Signal keyword         |
| --------------- | ----------------------------------------------- | ---------------------- |
| **AWS Batch**   | Runs large / long-running batch workloads       | Batch jobs             |
| **AWS AppSync** | Managed GraphQL APIs                            | GraphQL                |
| **AWS Amplify** | Rapid web/mobile app development and deployment | Web/mobile development |
| **Amazon SES**  | Sends application email                         | Email                  |

---

# Important SAA Distinctions

## Batch vs Lambda

```text
Lambda
= short event-driven functions

AWS Batch
= long-running batch workloads
```

Think about the **workload model**, not simply whether a container is involved.

---

## AppSync vs Amplify

```text
GraphQL API
→ AppSync

Build/deploy web or mobile application
→ Amplify
```

They can be used together.

---

## SES vs SNS

```text
Email
→ SES

Notifications / pub-sub
→ SNS
```

---

# Common Question Patterns

> **"Run long-running batch workloads."**

→ **AWS Batch**

---

> **"Thousands of computational jobs need to run without managing the batch scheduler."**

→ **AWS Batch**

---

> **"Application needs GraphQL."**

→ **AWS AppSync**

---

> **"Application needs GraphQL and real-time subscriptions."**

→ **AWS AppSync**

---

> **"Mobile application needs GraphQL and offline synchronization."**

→ **AWS AppSync**

---

> **"Quickly build and deploy a web/mobile application."**

→ **AWS Amplify**

---

> **"Development team wants an easier way to connect a frontend application with AWS backend services."**

→ **AWS Amplify**

---

> **"Application needs to send verification emails."**

→ **Amazon SES**

---

> **"Application needs to send receipts and notification emails."**

→ **Amazon SES**

---

# Application Development Decision Tree

```text
What is the requirement?
          │
          ├── Long-running batch jobs?
          │       ↓
          │    AWS Batch
          │
          ├── GraphQL?
          │       ↓
          │    AppSync
          │
          ├── GraphQL + real-time subscriptions?
          │       ↓
          │    AppSync
          │
          ├── GraphQL + offline synchronization?
          │       ↓
          │    AppSync
          │
          ├── Quickly build/deploy web or mobile app?
          │       ↓
          │    Amplify
          │
          └── Application email?
                  ↓
                 SES
```

---

# Pocket Card

| Keyword                                  | Answer        |
| ---------------------------------------- | ------------- |
| Long-running batch jobs                  | **AWS Batch** |
| Large-scale batch workloads              | **AWS Batch** |
| Batch containers                         | **AWS Batch** |
| GraphQL                                  | **AppSync**   |
| GraphQL + subscriptions                  | **AppSync**   |
| GraphQL + real-time updates              | **AppSync**   |
| GraphQL + offline synchronization        | **AppSync**   |
| Quick web/mobile application development | **Amplify**   |
| Web/mobile app deployment                | **Amplify**   |
| Application email                        | **SES**       |
| Verification emails                      | **SES**       |
| Email receipts                           | **SES**       |
| Email notifications                      | **SES**       |

---

# Final Memory

```text
AWS Batch
= BATCH COMPUTING
= LONG-RUNNING BATCH JOBS

AppSync
= GRAPHQL
= REAL-TIME SUBSCRIPTIONS
= OFFLINE SYNCHRONIZATION

Amplify
= WEB / MOBILE APP DEVELOPMENT
= RAPID APP BUILD + DEPLOYMENT

SES
= APPLICATION EMAIL
```

# The Golden Rule

```text
Long-running batch jobs
→ AWS Batch

GraphQL
→ AppSync

GraphQL + real-time subscriptions
→ AppSync

GraphQL + offline synchronization
→ AppSync

Quick web/mobile development
→ Amplify

Application needs to send email
→ SES
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Batch jobs           → AWS Batch
GraphQL              → AppSync
GraphQL subscriptions → AppSync
Offline sync         → AppSync
Web/mobile app       → Amplify
Application email    → SES
```
