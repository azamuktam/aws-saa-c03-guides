# Section 37: The Gap-Fill Services (small but tested)

## The idea

You've built the skyscrapers — EC2, S3, VPC, RDS. This section is the **spice rack**: dozens of small jars you'll each use once, but when the recipe calls for saffron, only saffron will do. Each of these services shows up on the exam **once or twice**, almost always as a pure **keyword-match** question. You don't need depth; you need to hear the signal word and slap the right label on it. That makes this the highest points-per-minute section you'll study — easy marks hiding in plain sight.

Read each entry as: *signal keyword → service*. That's genuinely how the questions work.

## Migration & messaging

**Amazon MQ** — managed **ActiveMQ / RabbitMQ** message broker. Signal: *"existing application uses **MQTT / AMQP / JMS** (industry-standard messaging protocols) and must migrate with **minimal rewrite**."* THE trap: for **new** applications, AWS wants **SQS/SNS** — MQ is only for lifting existing broker-based apps.

**Application Migration Service (MGN)** — the **lift-and-shift (rehost)** tool. Installs an agent on your on-prem servers, does **continuous block-level replication** into AWS, then you **cut over** to EC2 with minutes of downtime. Signal: *"migrate servers to AWS with minimal changes."*

**Elastic Disaster Recovery (DRS)** — MGN's twin, but for **DR instead of migration**: same continuous replication, but you only launch the recovery instances when disaster strikes. **RPO in seconds, RTO in minutes**, and it's cheap while idle. Signal: *"cost-effective disaster recovery for physical/virtual/cloud servers."*

**DMS + SCT** — **Database Migration Service** moves databases (and can do **ongoing replication**, source stays live during migration). If the engines differ — Oracle → Aurora — that's a **heterogeneous** migration and you add the **Schema Conversion Tool (SCT)** first. Same engine = DMS alone; **different engine = DMS + SCT**.

**The 7 Rs of migration** — one line each:

| R | Meaning |
|---|---|
| **Rehost** | Lift-and-shift, no changes (MGN) |
| **Replatform** | Lift-and-tinker (e.g., self-managed MySQL → RDS) |
| **Repurchase** | Drop it, buy SaaS instead |
| **Refactor** | Re-architect for cloud (e.g., monolith → serverless) |
| **Relocate** | Move VMware wholesale (VMware Cloud on AWS) |
| **Retain** | Leave it on-prem for now |
| **Retire** | Turn it off, nobody used it anyway |

## Certificates & network odds-and-ends

**ACM (AWS Certificate Manager)** — **free public TLS/SSL certificates** with **automatic renewal**, attachable to ALB, CloudFront, API Gateway. Two exam facts: a certificate for **CloudFront MUST live in us-east-1** (any region works for a regional ALB), and **you cannot export ACM public certs to install on an EC2 instance** — ACM only attaches to integrated services.

**Egress-only Internet Gateway** — the **IPv6 "NAT gateway."** IPv6 addresses are all public, so NAT doesn't apply; this gateway lets IPv6 instances make **outbound-only** connections while blocking inbound. Signal: *"IPv6 instances need outbound internet, no inbound."*

**Route 53 Resolver endpoints** — hybrid DNS plumbing. **Inbound endpoint**: **on-prem → VPC** DNS queries (your datacenter resolves AWS private names). **Outbound endpoint**: **VPC → on-prem** DNS queries (AWS resolves your corporate names). Remember it from the VPC's point of view: which way are the *queries coming*?

## Edge & hybrid infrastructure

| Service | One-liner | Signal keyword |
|---|---|---|
| **Outposts** | AWS-managed racks physically installed **in YOUR datacenter** | "data residency," "must stay on-premises but use AWS APIs" |
| **Local Zones** | AWS extension in a **metro city** near your users | "**single-digit millisecond** latency to users in Los Angeles" |
| **Wavelength** | AWS compute embedded in **5G telecom networks** | "5G," "mobile edge" |

## Analytics family

| Service | One-liner | Signal keyword |
|---|---|---|
| **Glue** | **Serverless ETL** (extract-transform-load) + **Data Catalog** + crawlers that auto-discover schemas | "serverless data preparation/catalog" |
| **EMR** | Managed **Hadoop/Spark** cluster; use **Spot for task nodes** to cut cost | "**Apache Spark**," "big data frameworks" |
| **MSK** | Managed **Apache Kafka** | the word "**Kafka**" — that's it |
| **Athena** | SQL queries directly on S3, pay per query (you know this one) | "query S3 with SQL" |
| **QuickSight** | **BI dashboards** and visualizations | "business dashboards for management" |
| **Lake Formation** | Build a **data lake** fast + **fine-grained (row/column) access control**. A data lake is a central place where an organization stores large amounts of raw data, usually in its original format, so it can be analyzed later.Data lake = huge storage pool for all kinds of data. | "central data lake with granular permissions" |
| **AppFlow** | Move data from **SaaS apps (Salesforce, Slack) → S3/Redshift**, no code | "Salesforce data into S3" |

THE trap: "streaming" alone → Kinesis; but "**Kafka**" named explicitly → **MSK**. AWS respects the proper noun.

## ML one-liners (pure keyword matching)

| Service | Does | Signal |
|---|---|---|
| **Rekognition** | Analyze **images/video** | face detection, content moderation |
| **Transcribe** | **Speech → text** | call transcripts, subtitles |
| **Polly** | **Text → speech** | "app reads articles aloud" |
| **Translate** | Language translation | "localize to 12 languages" |
| **Comprehend** | **NLP / sentiment** analysis | "customer review sentiment" |
| **Textract** | **Extract text + data from scanned forms/invoices** — understands tables and fields, **beyond simple OCR** | "process scanned documents/forms" |
| **Kendra** | **Intelligent document search** (natural-language answers from your docs) | "enterprise search across wikis/manuals" |
| **Personalize** | **Recommendations** engine | "customers also bought" |
| **Forecast** | Time-series **predictions** | "predict inventory demand" |
| **Lex** | **Chatbots** (the tech behind Alexa) | "conversational bot" |
| **SageMaker** | Build/train/deploy **custom ML models** | "data scientists need to train models" |

THE trap: scanned invoice/form extraction → **Textract**, not Rekognition (images ≠ documents) and not Comprehend (that's meaning, not extraction).

## Systems Manager (SSM) suite

One service, several exam-favorite features:

- **Session Manager** — shell access to instances with **no SSH keys, no bastion host, no open port 22**, all logged. Signal: *"secure instance access without opening inbound ports."*
- **Run Command** — execute scripts **across a fleet** at once, no SSH.
- **Patch Manager** — **automated OS patching** on a schedule (maintenance windows).
- **Hybrid**: install the SSM agent on **on-premises servers** and manage them alongside EC2.

## Developer & app services

**AWS Batch** — run **long batch jobs** as Docker containers; schedules onto EC2/Spot for you. Signal: *"job runs for hours"* — the escape hatch from **Lambda's 15-minute limit**.

**AppSync** — managed **GraphQL** API. Signal words: "**GraphQL**," "**real-time subscriptions**," "**offline sync**" for mobile. Any of the three → AppSync.

**Amplify** — full-stack web/mobile app **quickstart**: hosting, auth, backend wired together. Signal: *"frontend team wants to build and deploy quickly without managing infrastructure."*

**SES (Simple Email Service)** — send **application emails** at scale (receipts, notifications, marketing). Signal: *"application must send emails."* (SNS emails are plain-text alerts for ops; SES is for real email.)

**S3 Batch Operations** — run one operation across **billions of EXISTING objects**: copy, tag, restore, or — the exam favorite — **"encrypt all existing objects in the bucket."** Default bucket encryption only covers *new* uploads; Batch Operations fixes history.

**Aurora cloning** (recap) — **copy-on-write clone** of a production Aurora DB in minutes, no full copy, cheap. Signal: *"test against production data quickly without impacting production."*

**X-Ray** — **distributed tracing**: follow one request across microservices to find the slow/broken hop. Signal: *"identify performance bottlenecks across microservices."* (CloudWatch = metrics/logs; X-Ray = the request's journey.)

**AWS Artifact** — self-service portal to download **AWS compliance reports** (SOC, PCI, ISO). Signal: *"auditors need proof of AWS's compliance."* No agents, no config — just the download counter for paperwork.

## Question patterns

> *"Existing on-prem app uses RabbitMQ with AMQP; migrate with minimal code changes"* → **Amazon MQ** (protocol keywords = MQ; SQS would mean rewriting)

> *"HTTPS on CloudFront with a custom domain — where's the certificate?"* → **ACM in us-east-1** (CloudFront only reads certs from N. Virginia)

> *"IPv6-only instances need outbound internet access but must block inbound"* → **Egress-only Internet Gateway** (the IPv6 answer; NAT gateway is IPv4)

> *"Automatically extract fields and tables from scanned invoices"* → **Textract** (forms/documents, beyond OCR — not Rekognition)

> *"Migrate a self-managed Apache Kafka cluster with no application changes"* → **MSK** (the word Kafka does all the work)

> *"Mobile app needs a GraphQL API with real-time data sync"* → **AppSync** (GraphQL/subscriptions/offline = AppSync every time)

> *"Automatically apply OS security patches to 500 EC2 instances monthly"* → **SSM Patch Manager** (fleet patching on a schedule)

> *"Encrypt all existing objects already stored in an S3 bucket"* → **S3 Batch Operations** (default encryption only covers new objects)

> *"Data team needs managed Apache Spark for nightly processing at low cost"* → **EMR with Spot task nodes** (Spark = EMR; Spot = cheap)

> *"Regulations require workloads to run in the company's own datacenter using AWS services"* → **Outposts** (AWS racks on your floor — data residency)

> *"Shell access to private instances without SSH keys or bastion hosts"* → **SSM Session Manager** (no port 22, fully audited)

> *"Migrate Oracle to Aurora PostgreSQL"* → **DMS + SCT** (different engines = heterogeneous = add the Schema Conversion Tool)

## Pocket card

| Keyword | Answer |
|---|---|
| MQTT / AMQP / JMS, minimal rewrite | Amazon MQ |
| Lift-and-shift servers (rehost) | MGN |
| DR replication, RPO seconds | Elastic Disaster Recovery (DRS) |
| DB migration, different engines | DMS + SCT |
| Free auto-renewing TLS certs | ACM |
| CloudFront certificate region | us-east-1 (always) |
| IPv6 outbound-only | Egress-only Internet Gateway |
| On-prem → VPC DNS | Resolver inbound endpoint |
| VPC → on-prem DNS | Resolver outbound endpoint |
| AWS racks in your datacenter | Outposts |
| Single-digit ms to a metro city | Local Zones |
| 5G edge | Wavelength |
| Serverless ETL + catalog | Glue |
| Spark / Hadoop | EMR |
| Kafka | MSK |
| BI dashboards | QuickSight |
| Data lake + fine-grained access | Lake Formation |
| SaaS → S3 | AppFlow |
| Images/video analysis | Rekognition |
| Speech → text / text → speech | Transcribe / Polly |
| Sentiment | Comprehend |
| Scanned forms/invoices | Textract |
| Intelligent doc search | Kendra |
| Recommendations / forecasts / chatbots | Personalize / Forecast / Lex |
| Custom ML | SageMaker |
| No-SSH access / fleet scripts / auto-patching | Session Manager / Run Command / Patch Manager |
| Batch jobs > 15 min, Docker | AWS Batch |
| GraphQL, offline sync | AppSync |
| Full-stack quickstart | Amplify |
| Application emails | SES |
| Bulk ops on existing S3 objects | S3 Batch Operations |
| Instant prod DB copy for testing | Aurora cloning |
| Distributed tracing | X-Ray |
| Compliance reports for auditors | AWS Artifact |

That's the last jar on the spice rack — you've now covered the whole SAA-C03 kitchen. Sleep well, read the last sentence first, and go collect your certificate.
