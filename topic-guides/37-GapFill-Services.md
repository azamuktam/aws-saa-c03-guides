# Section 37: The Gap-Fill Services

## The idea

These are smaller AWS services that usually appear in SAA questions as **specific use cases**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Existing RabbitMQ application
→ Amazon MQ

Lift-and-shift servers to AWS
→ AWS Transform MGN

Disaster recovery for servers
→ AWS Elastic Disaster Recovery

TLS certificates
→ ACM

IPv6 outbound-only Internet access
→ Egress-Only Internet Gateway

Query S3 using SQL
→ Athena
```

---

## Migration & messaging

### Amazon MQ

**Amazon MQ = managed message broker for applications that already use traditional messaging systems.**

It supports managed brokers such as:

* ActiveMQ
* RabbitMQ

It is useful when an existing application already uses traditional messaging protocols or APIs and you want to migrate it to AWS without rewriting the application.

Common keywords:

* RabbitMQ
* ActiveMQ
* JMS
* AMQP
* existing message broker
* minimal application changes

### Important distinction

```text
Existing broker-based application
→ Amazon MQ

New AWS-native application
→ SQS / SNS / EventBridge
```

### Example

> *"An existing application uses RabbitMQ and the company wants to migrate to AWS with minimal code changes."*

→ **Amazon MQ**

---

### AWS Transform MGN

**AWS Application Migration Service (MGN) = rehost / lift-and-shift servers to AWS.**

The service continuously replicates the source server's **block-level data** to AWS.

```text
On-premises server
        ↓
    MGN agent
        ↓
Continuous replication
        ↓
AWS staging area
        ↓
EC2
```

The goal is to move the server to AWS with **minimal changes**.

### Use it when

> "Move existing physical or virtual servers to AWS without redesigning the application."

→ **MGN**

MGN provides continuous data protection with recovery points near seconds and can achieve recovery in minutes in appropriate configurations.

### Remember

> **Rehost / lift-and-shift → MGN**

---

### AWS Elastic Disaster Recovery (DRS)

**AWS Elastic Disaster Recovery = disaster recovery for servers.**

It continuously replicates workloads into AWS, but the recovery environment is mainly used **when a disaster happens**.

```text
Production server
      ↓
Continuous replication
      ↓
AWS recovery environment

Disaster
   ↓
Launch recovery instances
```

### Use it when

> "We need a cost-effective disaster recovery solution for physical, virtual, or cloud servers."

→ **AWS DRS**

### MGN vs DRS

```text
MGN
= migrate to AWS

DRS
= recover in AWS when disaster happens
```

Both use continuous replication, but the goal is different.

---

### AWS DMS + SCT

**AWS Database Migration Service (DMS) = move or replicate database data.**

**AWS Schema Conversion Tool (SCT) = convert schema/code when changing database engines.**

```text
Same database engine
→ DMS

Different database engine
→ SCT + DMS
```

Example:

```text
Oracle
   ↓
SCT → convert schema
   ↓
DMS → move data
   ↓
Aurora PostgreSQL
```

### Remember

> **DMS = move data**

> **SCT = convert schema**

---

## The 7 Rs of migration

The 7 Rs describe different migration strategies.

| R                           | Meaning                    | Simple idea                            |
| --------------------------- | -------------------------- | -------------------------------------- |
| **Rehost**                  | Move without major changes | Lift and shift                         |
| **Replatform**              | Move with small changes    | Use a managed AWS service              |
| **Repurchase**              | Replace the application    | Buy a SaaS product                     |
| **Refactor / Re-architect** | Redesign the application   | Build it for cloud-native architecture |
| **Relocate**                | Move the whole environment | VMware Cloud, for example              |
| **Retain**                  | Keep it where it is        | Don't migrate yet                      |
| **Retire**                  | Stop using it              | Decommission it                        |

### Easy examples

```text
Rehost
On-prem VM → EC2

Replatform
MySQL on EC2 → Amazon RDS

Repurchase
Self-hosted CRM → SaaS CRM

Refactor
Monolith → Lambda / containers / serverless architecture

Relocate
VMware environment → VMware Cloud on AWS

Retain
Keep the workload on-premises for now

Retire
Delete an unused application
```

---

# Certificates & network services

## ACM — AWS Certificate Manager

**ACM = TLS/SSL certificates for AWS services.**

Use it when you need HTTPS.

Common integrations:

* ALB
* CloudFront
* API Gateway

### Important facts

* Public ACM certificates are provided at no additional charge.
* ACM can automatically renew certificates that meet the renewal requirements.
* A CloudFront certificate must be in **us-east-1**.
* A regional service such as an ALB normally uses a certificate in the same Region.

### Important exam trap

You generally cannot export an **ACM public certificate's private key** for installation on an EC2 server.

So:

```text
HTTPS on ALB
→ ACM

HTTPS on CloudFront
→ ACM in us-east-1
```

---

## Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

```text
IPv6 EC2
   ↓
Egress-Only Internet Gateway
   ↓
Internet
```

It allows:

```text
EC2 → Internet ✅
Internet → EC2 ❌
```

### Remember

> **IPv6 + outbound only → Egress-Only Internet Gateway**

For IPv4 private instances, the equivalent pattern is normally a **NAT Gateway**.

---

## Route 53 Resolver endpoints

Route 53 Resolver endpoints connect **AWS DNS resolution with on-premises DNS**.

There are two types.

### Inbound endpoint

Queries come **into AWS**:

```text
On-premises
    ↓
Inbound Resolver Endpoint
    ↓
AWS VPC DNS
```

Use it when:

> **"On-premises servers need to resolve private DNS names in AWS."**

### Outbound endpoint

Queries go **out of AWS**:

```text
AWS VPC
   ↓
Outbound Resolver Endpoint
   ↓
On-premises DNS
```

Use it when:

> **"AWS resources need to resolve internal/on-premises DNS names."**

### Memory

```text
Inbound
= query comes INTO AWS

Outbound
= query goes OUT OF AWS
```

---

# Edge & hybrid infrastructure

| Service         | What it does                                                       | Signal keyword                 |
| --------------- | ------------------------------------------------------------------ | ------------------------------ |
| **Outposts**    | AWS infrastructure installed in your own data center               | AWS services **on-premises**   |
| **Local Zones** | AWS infrastructure closer to users in a specific metropolitan area | Very low latency to a **city** |
| **Wavelength**  | AWS infrastructure inside telecom 5G networks                      | **5G / mobile edge**           |

### Outposts

**AWS Outposts = AWS infrastructure physically installed in your data center.**

Use it when:

* data must remain on-premises
* you need very low latency to local systems
* local workloads must use AWS APIs/services

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS-style infrastructure
```

### Local Zones

**Local Zones = AWS infrastructure placed closer to users in a metropolitan area.**

Use it when an application needs very low latency for users in a specific city.

> **City-level low latency → Local Zones**

### Wavelength

**AWS Wavelength = AWS infrastructure inside 5G telecom networks.**

Use it for:

* mobile applications
* 5G applications
* extremely low-latency mobile workloads

> **5G → Wavelength**

---

# Analytics family

| Service            | What it does                                              | Signal keyword                   |
| ------------------ | --------------------------------------------------------- | -------------------------------- |
| ****           | Serverless ETL + Data Catalog                             | ETL / data preparation / catalog |
| **EMR**            | Managed big-data frameworks                               | Spark / Hadoop                   |
| **MSK**            | Managed Apache Kafka                                      | Kafka                            |
| **Athena**         | SQL directly on S3                                        | SQL on S3                        |
| **QuickSight**     | BI dashboards                                             | Business dashboards              |
| **Lake Formation** | Build/manage a data lake with fine-grained access control | Data lake + permissions          |
| **AppFlow**        | Move data between SaaS and AWS services                   | Salesforce → S3                  |

### 

**AWS  = serverless ETL and data catalog.**

ETL means:

```text
Extract
Transform
Load
```

 can:

* discover data
* catalog schemas
* transform data
* prepare data for analytics

### Signal

> **Serverless ETL / Data Catalog → **

## Glue ETL + S3

Glue is a **serverless ETL service**, so it is a strong choice when data must be transformed without managing servers.

Example:

```text
S3
 ↓ Object Created
EventBridge
 ↓
Glue ETL job
 ↓
CSV → Parquet
 ↓
S3
```
Glue is preferred over:

EC2 + Spark → requires server/infrastructure management
Lambda → better for lightweight/event-driven processing, not large ETL workloads
Glue Crawler → discovers/catalogs data; it does not perform the ETL transformation
---

### EMR

**Amazon EMR = managed big-data processing using frameworks such as Apache Spark and Hadoop.**

Use it for:

* large-scale data processing
* Spark jobs
* Hadoop workloads

You can use Spot Instances for suitable EMR task nodes to reduce cost.

### Signal

> **Spark / Hadoop → EMR**

---

### MSK

**Amazon Managed Streaming for Apache Kafka = managed Kafka.**

You manage Kafka-compatible workloads without managing the Kafka infrastructure yourself.

### Signal

> **Kafka → MSK**

Don't overthink it.

---

### Athena

**Athena = SQL directly on S3.**

You already know this from Section 12.

### Signal

> **SQL + S3 → Athena**

---

### QuickSight

**Amazon QuickSight = business intelligence and dashboards.**

Use it to create:

* dashboards
* charts
* reports
* business visualizations

### Signal

> **Business dashboard → QuickSight**

---

### Lake Formation

**AWS Lake Formation = build and manage a data lake with centralized, fine-grained permissions.**

It can control access to specific:

* tables
* columns
* rows

### Signal

> **Data lake + fine-grained permissions → Lake Formation**

---

### AppFlow

**Amazon AppFlow = transfer data between SaaS applications and AWS services without writing the integration yourself.**

Example:

```text
Salesforce
    ↓
AppFlow
    ↓
S3
```

### Signal

> **Salesforce/SaaS → S3 or Redshift → AppFlow**

---

# ML one-liners

| Service | What it does | Signal |
|---|---|---|
| **Rekognition** | Looks at **images and videos** and detects things such as faces, objects, people, and unsafe content | Faces, objects, video |
| **Transcribe** | Takes **audio/speech** and turns it into **written text** | Call recording → transcript |
| **Polly** | Takes **written text** and turns it into **spoken audio** | App reads text aloud |
| **Translate** | Takes **text in one language** and translates it into another language | English → French |
| **Comprehend** | Takes **text** and analyzes its meaning, such as **sentiment, entities, and key phrases** | "Is this review positive or negative?" |
| **Textract** | Takes **scanned documents/images** and extracts **text, tables, and form fields** | Invoice/form → structured data |
| **Kendra** | Searches **company documents** and finds relevant answers using natural-language queries | "Find our vacation policy" |
| **Personalize** | Uses user/item behavior to generate **personalized recommendations** | "Customers also bought..." |
| **Forecast** | Uses historical **time-series data** to predict future values | Predict future sales/demand |
| **Lex** | Lets you build **conversational chatbots** that understand user messages and respond | "Build a customer-service chatbot" |
| **SageMaker** | Lets data scientists **build, train, tune, and deploy their own ML models** | Train your own ML model |

Picture / video
→ Rekognition

Audio
→ Transcribe
   ↓
  Text

Text → speech
→ Polly

Text → another language
→ Translate

Understand text
→ Comprehend

Scanned document → text/tables/forms
→ Textract

Search company documents
→ Kendra

Recommend products/content
→ Personalize

Predict future numbers
→ Forecast

Chat with users
→ Lex

Build your own ML model
→ SageMaker

### The important distinctions

```text
Image / video
→ Rekognition

Speech → text
→ Transcribe

Text → speech
→ Polly

Text meaning / sentiment
→ Comprehend

Scanned form / invoice
→ Textract

Search company documents
→ Kendra

Recommendations
→ Personalize

Forecast future values
→ Forecast

Chatbot
→ Lex

Build your own ML model
→ SageMaker
```

### Common trap

> **"Extract names, fields, tables, and values from scanned invoices."**

→ **Textract**

Not Rekognition.

Rekognition is for images/video analysis; Textract is designed for extracting text and structured data from documents.

---

# Systems Manager (SSM) suite

AWS Systems Manager contains several tools that solve different operational tasks.

### Session Manager

**Session Manager = secure shell access to EC2 without SSH.**

You don't need:

* a bastion host
* an open inbound port 22
* SSH keys for the session itself

```text
Admin
  ↓
Session Manager
  ↓
Private EC2
```

### Signal

> **Secure access to private EC2 without SSH → Session Manager**

---

### Run Command

**Run Command = execute commands/scripts on many instances.**

Example:

```text
500 EC2 instances
      ↓
   Run Command
      ↓
Run the same script
on all selected instances
```

### Signal

> **Run a command across many EC2 instances → Run Command**

---

### Patch Manager

**Patch Manager = automate OS patching.**

Example:

> "Apply security patches to 500 EC2 instances every month."

→ **Patch Manager**

---

### Hybrid Systems Manager

Systems Manager can also manage supported **on-premises servers** when the SSM Agent and required connectivity are configured.

### Memory

```text
Session Manager
= ACCESS instances

Run Command
= RUN commands

Patch Manager
= PATCH instances
```

---

# Developer & app services

## AWS Batch

**AWS Batch = run large batch jobs without managing the job scheduler yourself.**

It is designed for jobs that may run for a long time.

Example:

```text
Thousands of jobs
      ↓
AWS Batch
      ↓
EC2 / Spot capacity
```

### Signal

> **Long-running batch jobs / containers → AWS Batch**

A common comparison:

```text
Lambda
= short event-driven functions

AWS Batch
= long-running batch workloads
```

---

## AppSync

**AWS AppSync = managed GraphQL API service.**

Use it when the question says:

* GraphQL
* real-time subscriptions
* offline synchronization for applications

Example:

```text
Mobile/Web app
      ↓
   AppSync
      ↓
Data sources
```

### Signal

> **GraphQL → AppSync**

---

## Amplify

**AWS Amplify = tools for quickly building and deploying web/mobile applications.**

It helps developers connect frontend applications with AWS backend services.

### Signal

> **Quick full-stack web/mobile development → Amplify**

---

## SES

**Amazon SES (Simple Email Service) = send application emails.**

Examples:

* receipts
* verification emails
* notifications
* marketing emails

### Signal

> **Application needs to send email → SES**

Don't confuse this with SNS:

```text
SNS
= notifications/pub-sub

SES
= email sending
```

---

## S3 Batch Operations

**S3 Batch Operations = perform an operation on many existing S3 objects.**

For example:

* copy objects
* restore objects
* add tags
* invoke Lambda
* perform other supported object-level operations

Example:

```text
Millions of existing objects
        ↓
S3 Batch Operations
        ↓
Apply the operation
```

### Important idea

A bucket's default encryption setting does **not retroactively change old objects**.

So:

> **"Perform an operation on millions/billions of existing S3 objects."**

→ **S3 Batch Operations**

---

## Aurora Cloning

**Aurora Cloning = quickly create a copy of an Aurora database using copy-on-write.**

Use it when:

> "Create a production-like database for testing without immediately duplicating all the storage."

This is covered in your Aurora section as well.

### Signal

> **Quick Aurora copy for testing → Aurora Cloning**

---

## X-Ray

**AWS X-Ray = distributed tracing.**

It follows a request as it moves through different services.

Example:

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
Service A
  ↓
Service B
```

X-Ray helps identify:

* which service is slow
* where a request failed
* where latency is coming from

### Important distinction

```text
CloudWatch
= metrics + logs + monitoring

X-Ray
= trace one request across services
```

### Signal

> **Find which microservice is causing latency → X-Ray**

---

## AWS Artifact

**AWS Artifact = access AWS compliance documents and reports.**

Examples include AWS compliance documentation such as:

* SOC
* PCI
* ISO-related reports

### Signal

> **Auditor needs AWS compliance reports → Artifact**

Artifact is a document portal. It is not a monitoring service.

---

## Question patterns

> *"Existing on-premises application uses RabbitMQ and must migrate with minimal code changes."* → **Amazon MQ**

> *"Move existing servers to AWS with minimal application changes."* → **AWS Transform MGN**

> *"Need disaster recovery for physical/virtual/cloud servers."* → **AWS Elastic Disaster Recovery (DRS)**

> *"Migrate Oracle to Aurora PostgreSQL."* → **AWS SCT + DMS**

> *"Need HTTPS certificate for CloudFront."* → **ACM in us-east-1**

> *"IPv6 instances need outbound Internet access but must block unsolicited inbound connections."* → **Egress-Only Internet Gateway**

> *"On-premises servers need to resolve private AWS DNS names."* → **Route 53 Resolver inbound endpoint**

> *"AWS resources need to resolve internal corporate DNS names."* → **Route 53 Resolver outbound endpoint**

> *"Workloads must run in the company's own data center but use AWS infrastructure/services."* → **Outposts**

> *"Need very low latency for users in a specific metropolitan area."* → **Local Zones**

> *"Application needs extremely low-latency processing over a 5G network."* → **Wavelength**

> *"Serverless ETL and a central data catalog are required."* → **AWS Glue**

> *"Managed Apache Spark processing."* → **Amazon EMR**

> *"Existing Apache Kafka workload."* → **Amazon MSK**

> *"Query files in S3 using SQL."* → **Amazon Athena**

> *"Business users need dashboards and visual reports."* → **Amazon QuickSight**

> *"Build a data lake with fine-grained table/row/column permissions."* → **Lake Formation**

> *"Move Salesforce data to S3 without custom integration code."* → **AppFlow**

> *"Automatically extract fields and tables from scanned invoices."* → **Textract**

> *"Search internal company documents using natural-language queries."* → **Kendra**

> *"Speech recordings need to become text."* → **Transcribe**

> *"Application needs to read text aloud."* → **Polly**

> *"Customer reviews need sentiment analysis."* → **Comprehend**

> *"Product recommendations based on user behavior."* → **Personalize**

> *"Predict future demand based on historical time-series data."* → **Forecast**

> *"Build a conversational chatbot."* → **Lex**

> *"Data scientists need to build/train/deploy a custom ML model."* → **SageMaker**

> *"Secure shell access to private EC2 without SSH or a bastion."* → **SSM Session Manager**

> *"Run the same command on hundreds of EC2 instances."* → **SSM Run Command**

> *"Automatically patch hundreds of EC2 instances."* → **SSM Patch Manager**

> *"Run long-running batch workloads."* → **AWS Batch**

> *"Application needs GraphQL and real-time subscriptions."* → **AppSync**

> *"Quickly build and deploy a web/mobile application."* → **Amplify**

> *"Application needs to send emails."* → **SES**

> *"Apply an operation to millions of existing S3 objects."* → **S3 Batch Operations**

> *"Create a fast copy of an Aurora database for testing."* → **Aurora Cloning**

> *"Find which microservice is causing latency in a request."* → **X-Ray**

> *"Auditors need AWS compliance reports."* → **AWS Artifact**

---

## Pocket card

| Keyword                                  | Answer                                  |
| ---------------------------------------- | --------------------------------------- |
| Existing RabbitMQ / ActiveMQ application | **Amazon MQ**                           |
| Minimal-change server migration / rehost | **AWS Transform MGN**                   |
| Disaster recovery for servers            | **AWS Elastic Disaster Recovery (DRS)** |
| Database migration                       | **DMS**                                 |
| Different database engines               | **SCT + DMS**                           |
| TLS/SSL certificates                     | **ACM**                                 |
| CloudFront certificate                   | **ACM in us-east-1**                    |
| IPv6 outbound-only Internet              | **Egress-Only Internet Gateway**        |
| On-prem → AWS DNS queries                | **Route 53 Resolver inbound endpoint**  |
| AWS → on-prem DNS queries                | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your data center   | **Outposts**                            |
| Low latency to a specific city           | **Local Zones**                         |
| 5G edge                                  | **Wavelength**                          |
| Serverless ETL / data catalog            | **Glue**                                |
| Spark / Hadoop                           | **EMR**                                 |
| Kafka                                    | **MSK**                                 |
| SQL on S3                                | **Athena**                              |
| BI dashboards                            | **QuickSight**                          |
| Data lake + fine-grained permissions     | **Lake Formation**                      |
| SaaS → S3 / Redshift                     | **AppFlow**                             |
| Images/video                             | **Rekognition**                         |
| Speech → text                            | **Transcribe**                          |
| Text → speech                            | **Polly**                               |
| Text/sentiment analysis                  | **Comprehend**                          |
| Scanned forms/invoices                   | **Textract**                            |
| Enterprise document search               | **Kendra**                              |
| Recommendations                          | **Personalize**                         |
| Time-series forecasting                  | **Forecast**                            |
| Chatbot                                  | **Lex**                                 |
| Custom ML models                         | **SageMaker**                           |
| Secure instance shell without SSH        | **SSM Session Manager**                 |
| Run commands across instances            | **SSM Run Command**                     |
| Automated OS patching                    | **SSM Patch Manager**                   |
| Long-running batch jobs                  | **AWS Batch**                           |
| GraphQL / subscriptions                  | **AppSync**                             |
| Fast web/mobile application development  | **Amplify**                             |
| Application email                        | **SES**                                 |
| Bulk operations on existing S3 objects   | **S3 Batch Operations**                 |
| Quick Aurora copy                        | **Aurora Cloning**                      |
| Distributed request tracing              | **X-Ray**                               |
| AWS compliance reports                   | **AWS Artifact**                        |

## Final memory

```text id="5v7v5v"
Amazon MQ
= EXISTING MESSAGE BROKER

MGN
= MOVE SERVERS

DRS
= RECOVER SERVERS

DMS
= MOVE DATABASE DATA

SCT
= CONVERT DATABASE SCHEMA

ACM
= HTTPS CERTIFICATES

Egress-Only IGW
= IPv6 OUTBOUND ONLY

Outposts
= AWS IN YOUR DATA CENTER

Local Zones
= AWS CLOSER TO A CITY

Wavelength
= AWS ON 5G

Glue
= ETL / DATA CATALOG

EMR
= SPARK / HADOOP

MSK
= KAFKA

AppFlow
= SaaS → AWS

Textract
= SCANNED DOCUMENTS

Kendra
= DOCUMENT SEARCH

SSM Session Manager
= SECURE INSTANCE ACCESS

SSM Run Command
= RUN COMMANDS ON MANY INSTANCES

SSM Patch Manager
= PATCH INSTANCES

Batch
= LONG BATCH JOBS

AppSync
= GRAPHQL

SES
= EMAIL

X-Ray
= DISTRIBUTED TRACING

Artifact
= COMPLIANCE DOCUMENTS
```

The main rule for this section is:

```text id="z9ig3z"
Don't memorize the implementation.

Memorize the unique signal.
```

For example:

```text
RabbitMQ      → MQ
Rehost        → MGN
DR            → DRS
Kafka         → MSK
Spark         → EMR
5G            → Wavelength
Scanned form  → Textract
GraphQL       → AppSync
Compliance    → Artifact
```
