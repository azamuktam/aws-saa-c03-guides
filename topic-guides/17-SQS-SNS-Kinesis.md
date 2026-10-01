# Section 17: SQS, SNS, SWF, Step Functions & Kinesis — the messaging block

## The idea

Messaging/integration services **decouple application components** so they do not need direct communication.

| Without a queue | With SQS |
|---|---|
| Slow processor makes web request wait | Web tier sends message and continues |
| Processor failure can fail request or lose work | Messages wait until a worker processes them |
| Traffic spikes can overload processor | Queue absorbs spikes; workers can scale with message count |

**Decoupling = separating message production from message processing.**

---

# SQS — Simple Queue Service

**SQS = managed message queue for decoupling applications.**

- **Producer → SQS → Consumer**
- Consumers **pull** messages.
- In normal SQS usage, a message is processed by **one consumer at a time**.

---

## Visibility timeout

When a consumer receives a message, it becomes temporarily hidden; it is **not deleted immediately**.

- Default visibility timeout: **30 seconds**
- Successful processing → consumer should **delete** the message.
- Worker crashes before deletion → timeout expires → message becomes visible again → another worker can process it.

### Common exam question

> **"Messages are being processed more than once."**

A common cause: **processing takes longer than the visibility timeout**.

Example:
```text
Visibility timeout = 30 seconds
Processing time    = 60 seconds
```

The message becomes visible again after 30s while the first worker is still processing it.

**Fix → Increase the visibility timeout.**

---

# Dead-Letter Queue (DLQ)

A **Dead-Letter Queue** stores messages that repeatedly fail processing.

Configure **MaxReceiveCount**:

```text
Main SQS queue
  ↓
Repeated processing failures
  ↓
MaxReceiveCount reached
  ↓
Dead-Letter Queue
```

Prevents a bad message from continuously returning to the main queue.

**Signal:** repeated processing failures → **DLQ + MaxReceiveCount**

---

# Long polling

With short polling, a consumer repeatedly checks the queue and may receive empty responses.

**Long polling:**
- Waits for messages for up to **20 seconds**
- Reduces unnecessary empty responses
- Reduces API calls
- Reduces polling cost

**Signal:** reduce empty polling / polling cost → **Long polling**

---

# SQS message size

Maximum SQS message size: **256 KB**.

For larger payloads:

```text
Large payload
  ↓
Store object in S3
  ↓
Send S3 location/pointer in SQS message
```

**Signal:** message > **256 KB** → **S3 + pointer in SQS**

---

# SQS message retention

- **4 days by default**
- **14 days maximum**

SQS is a queue, **not permanent storage**.

---

# Standard vs FIFO

| | Standard | FIFO |
|---|---|---|
| Throughput | Very high / virtually unlimited | **300 msg/s** or **3,000 msg/s with batching** per FIFO queue without high-throughput FIFO settings |
| Delivery | At-least-once | Designed to avoid duplicates through FIFO deduplication |
| Ordering | Best-effort | **Strict order** |
| Use when | Highest throughput / normal queue | Ordering or duplicate-sensitive workloads |

## Standard Queue

Use Standard when:
- Very high throughput is required
- Occasional duplicate delivery is acceptable
- Strict ordering is not required

## FIFO Queue

Use FIFO when:
- Message order matters
- Duplicate processing must be minimized
- Messages must be processed in strict order

**Signal:** "Messages must be processed in strict order." → **SQS FIFO**

### Important nuance

FIFO provides **exactly-once processing support / deduplication behavior**, but the application should still be designed carefully for retries and side effects.

---

# SQS + Auto Scaling

Common architecture:

```text
Application
  ↓
SQS
  ↓
Auto Scaling Group
  ↓
EC2 workers
```

CloudWatch can monitor queue depth, especially:

`ApproximateNumberOfMessagesVisible`

- More messages → scale out workers
- Fewer messages → scale in workers

**Signal:** workers should scale based on queue depth → **SQS + CloudWatch + Auto Scaling**

---

# SNS — Simple Notification Service

**SNS = managed publish/subscribe messaging service.**

- Producer publishes to an **SNS topic**.
- SNS sends the message to its subscribers.
- Subscribers can include:
  - **SQS**
  - **Lambda**
  - **HTTP/S endpoints**
  - **Email**
  - **SMS**
  - **Mobile push notifications**

Core idea: **one message can be delivered to many subscribers.**

---

# SNS push model

SNS uses **push**:

```text
Publisher
  ↓
SNS Topic
  ↓
Subscriber 1
Subscriber 2
Subscriber 3
```

Contrast:

```text
SQS = consumers PULL messages
SNS = SNS PUSHES messages to subscribers
```

---

# SNS message storage

SNS is **not a long-term message queue**.

For SAA questions:

> **SNS = notification/fan-out, not storage for consumers to process later.**

For durable downstream buffering:

```text
SNS
 ↓
SQS
 ↓
Consumer
```

---

# SNS Fan-out

**Fan-out = one event → multiple independent consumers.**

Example:

```text
                     → SQS → Analytics
                   /
Producer → SNS Topic → SQS → Fulfillment
                   \
                     → SQS → Fraud
```

Each SQS queue receives its **own copy**, giving each application an independent queue.

### Why use SQS after SNS?

If a consumer is unavailable:

```text
SNS → Fraud service
```

There is no queue where messages can wait for that consumer.

With SQS:

```text
SNS → SQS → Fraud service
```

The queue holds messages until the service recovers.

**Signal:** one event → multiple independent consumers → **SNS + multiple SQS queues**

---

# SNS message filtering

SNS supports **subscription filter policies** so different subscribers receive only the messages they need.

Example:

```text
SNS Topic
  ├→ Billing     → refund events
  ├→ Fraud       → suspicious events
  └→ Shipping    → shipment events
```

The publisher sends **one message** with message attributes or body values used by the filter policy.

Example:
```text
quoteType = "home"
```

SNS evaluates each subscription:

```text
Auto SQS  → filter: auto  → ❌
Home SQS  → filter: home  → ✅
Life SQS  → filter: life  → ❌
```

**Signal:** different subscribers should receive different message types → **SNS message filtering**

---

# SWF — Simple Workflow Service

**Amazon SWF = managed workflow orchestration for distributed applications.**

Coordinates **long-running, asynchronous workflows** made of multiple tasks.

Workers performing tasks can run:
- On **Amazon EC2**
- On AWS infrastructure
- On **on-premises servers**
- In other environments that can reach SWF

**Classic exam pattern:** workloads in both AWS and on-premises need a **distributed workflow**.

## SWF basic architecture

```text
Amazon SWF
  ├→ workflow logic / task coordination
  ├→ EC2 worker
  └→ on-premises worker
```

SWF coordinates:
- Task scheduling
- Task dependencies
- Workflow state
- Retries
- Failures
- Sequential tasks
- Parallel tasks

Workers perform the actual business logic.

---

# SWF vs SQS

| Service | Core job | Think |
|---|---|---|
| **SQS** | Message queue / application decoupling | Put work into a queue so another app can process it later |
| **SWF** | Workflow orchestration | Coordinate a multi-step workflow and track progress |

SQS:
```text
Producer → SQS → Consumer
```

SWF:
```text
Start workflow → Task A → Task B → Task C → Complete
```

---

# SWF and AWS + on-premises

### SQS example
```text
On-premises application
  ↓
 SQS
  ↓
EC2 worker
```

SQS decouples applications **through messages**.

### SWF example
```text
              SWF
             /   \
     EC2 worker   On-prem worker
             \   /
          Workflow continues
```

**Exam signal:**
> **"Coordinate tasks performed by workers running both on-premises and in AWS."**

→ **SWF**

---

# SWF workflow components

```text
Workflow
  ↓
Decider
  ↓
Activity tasks
  ↓
Activity workers
```

## Decider

The **decider** determines what should happen next.

Example:
```text
Payment successful?
  ├→ YES → Ship
  └→ NO  → Cancel
```

## Activity worker

An **activity worker** performs the actual business task.

Examples:
- Payment worker
- Shipping worker
- Document-processing worker
- On-premises validation worker

Workers **poll SWF for tasks** and report results back.

---

# Sequential and parallel workflows

SWF can coordinate both.

### Sequential
```text
Task A → Task B → Task C
```

### Parallel
```text
          ┌→ Task A ─┐
Start ────┤          ├→ Continue
          └→ Task B ─┘
```

---

# When to choose SWF

Choose SWF when the question emphasizes:
- Distributed workflow
- Long-running workflow
- Asynchronous workflow
- Multiple workflow steps
- Task coordination
- Workflow state
- Workers running on **AWS and on-premises**
- Sequential or parallel activities

**Strong exam signal:**
> **"Coordinate tasks performed by workers running both on-premises and in AWS."**

→ **SWF**

---

# SWF vs Step Functions

For **modern AWS architecture**, **AWS Step Functions** is the more important modern workflow orchestration service.

**SWF** is a **legacy/older workflow service** that can still appear in SAA-style questions.

| SWF | Step Functions |
|---|---|
| Older distributed workflow service | Modern AWS workflow orchestration |
| Custom workers | State-machine based |
| Can coordinate AWS + on-premises workers | Integrates directly with many AWS services |

Step Functions can also interact with external workers through **Activities**. Therefore the distinction is **not simply** "SWF = on-premises and Step Functions = AWS."

**Strong SWF clue:**
> **Distributed, long-running workflows with workers across environments, including on-premises.**

---

# Step Functions — modern workflow orchestration

**AWS Step Functions = managed workflow orchestration using state machines.**

Coordinates multiple AWS services/application steps into a workflow.

Example:

```text
Order → Validate → Charge payment → Ship → Send notification
```

Can coordinate:
- Lambda
- ECS
- SNS
- SQS
- DynamoDB
- AWS Batch
- Glue
- Other AWS services

---

## Step Functions state machine

A workflow is represented as **states and transitions**.

Example:
```text
Start
  ↓
Validate Order
  ↓
Payment
  ↓
Choice
 ├→ Ship
 └→ Cancel
      ↓
     End
```

### Common state types

- **Task**
- **Choice**
- **Wait**
- **Parallel**
- **Map**
- **Pass**
- **Succeed**
- **Fail**

**Signal:** "Coordinate several AWS services through a visual/state-machine workflow." → **Step Functions**

---

# Step Functions vs SQS

Not replacements:

```text
SQS            = queue work
Step Functions = orchestrate workflow steps
```

SQS example:
```text
Order service → SQS → Worker
```
SQS buffers/distributes work.

Step Functions:
```text
Step Functions
  ↓
Validate
  ↓
Charge
  ↓
Choice
  ├→ Ship
  └→ Cancel
```

Step Functions manages **workflow logic and state**.

---

# Kinesis Data Streams

**Kinesis Data Streams = real-time streaming service for continuously arriving data.**

Typical use cases:
- Clickstream data
- IoT telemetry
- Application events
- Logs
- Real-time analytics

```text
Producers
  ↓
Kinesis Data Streams
  ↓
Consumers
```

Key difference from SQS: **multiple applications can read the same stream of records.**

Example:
```text
              → Analytics application
            /
Producers → Kinesis Data Streams
            \
              → Fraud detection application
```

Both applications can process the **same data**.

---

# Replay

Kinesis Data Streams retains records so consumers can process them again.

- Default retention: **24 hours**
- Maximum retention: **365 days**

Applications can go back and reprocess older records **within the retention period**.

**Signal:** need to replay/reprocess streaming data → **Kinesis Data Streams**

---

# Shards

Kinesis Data Streams uses **shards** as a basic capacity unit.

A traditional provisioned shard provides approximately:
- **1 MB/s write throughput**
- **2 MB/s read throughput**

Records are assigned to shards based on the **partition key**.

Ordering is guaranteed **within a shard**.

```text
Partition key
  ↓
Shard
  ↓
Ordered records
```

**Signal:** need higher provisioned stream throughput → **increase number of shards**

---

# Kinesis Data Streams vs SQS

| | SQS | Kinesis Data Streams |
|---|---|---|
| Main purpose | Decoupling / work queues | Real-time streaming |
| Consumption | One consumer processes a message | Multiple applications can read the same stream |
| Replay | Not designed for stream-style replay | **Yes, within retention period** |
| Ordering | FIFO only | Ordering within a shard |
| Typical use | Background jobs, buffering | Streaming analytics, telemetry, event processing |

### Two important Kinesis signals

```text
Multiple applications need the same data
→ Kinesis Data Streams

Need to replay/reprocess old streaming data
→ Kinesis Data Streams
```

---

# Kinesis Data Firehose

**Kinesis Data Firehose = fully managed service for delivering streaming data to supported destinations.**

Typical destinations:
- **Amazon S3**
- **Amazon Redshift**
- **Amazon OpenSearch Service**
- **Splunk**

Can use **Lambda for data transformation** before delivery.

Typical architecture:
```text
Streaming data
  ↓
Kinesis Data Firehose
  ↓
S3 / Redshift / OpenSearch / Splunk
```

Firehose automatically **buffers** incoming data before delivery, so it is **near real time, not instant delivery**.

### Main advantage

**Very low operational effort.**

You do not need to write your own consumer application just to deliver the stream to a supported destination.

**Signal:** "Deliver streaming data to S3 with minimal operational overhead." → **Kinesis Data Firehose**

---

# Kinesis Data Streams vs Firehose

```text
Kinesis Data Streams
= custom stream consumers
= multiple consumers
= replay
= real-time processing

Kinesis Data Firehose
= managed delivery
= supported destinations
= minimal operational effort
= no custom consumer required
```

### Choose Data Streams when:
- Applications need to read the stream themselves
- Multiple consumers need the same records
- Replay/reprocessing is required
- Custom real-time processing is required

### Choose Firehose when:
- Data mainly needs delivery to **S3, Redshift, OpenSearch, or Splunk**
- Minimal operational effort is required
- Custom stream consumers are not needed

---

# Kinesis Video Streams

**Amazon Kinesis Video Streams = capture, transport, store, and process video streams for analysis and playback.**

Designed for **video and other time-encoded media**, especially:
- Security cameras
- Smartphones
- Connected devices
- IoT cameras

Typical architecture:
```text
Camera / device
  ↓
Kinesis Video Streams
  ↓
Applications / video processing / playback
```

Uses:
- Real-time or near-real-time processing
- Machine learning analysis
- Video playback
- Storage and later processing

### Important distinction

```text
Kinesis Data Streams
= application/event data
= logs, clicks, telemetry, events

Kinesis Video Streams
= video / time-encoded media
= cameras, video feeds
```

**Signal:** "Stream video from cameras/devices to AWS." → **Kinesis Video Streams**

---

# Amazon MQ

**Amazon MQ = managed message broker for existing applications using traditional messaging protocols.**

Supports:
- **RabbitMQ**
- **ActiveMQ**

Common protocol keywords:
- **AMQP**
- **MQTT**
- **JMS**
- **STOMP**

Use Amazon MQ when an existing application already depends on traditional broker technologies and migrating to SQS/SNS would require significant changes.

```text
Existing application
  ↓
RabbitMQ / ActiveMQ
  ↓
Amazon MQ
```

**Signals:**
- Existing RabbitMQ / ActiveMQ application → **Amazon MQ**
- AMQP / MQTT / JMS / STOMP → **Amazon MQ**

---

# Decoupled AWS + On-Premises Architecture

Important SAA pattern.

Example:
```text
AWS
  └─ EC2 application

On-premises
  └─ Internal application
```

Choose the integration service from the requirement.

## SQS

Requirement:
> **Asynchronous message-based decoupling**

```text
On-premises → SQS → EC2
```

## SWF

Requirement:
> **Coordinate a distributed workflow across workers in AWS and on-premises**

```text
              SWF
             /   \
       EC2 worker  On-prem worker
```

## Step Functions

Requirement:
> **Modern workflow orchestration using AWS services/state machines**

---

# What is NOT a decoupling service?

Some AWS services connect or store data but do not themselves provide application decoupling.

### DynamoDB
```text
Database ≠ Message queue
```

### RDS
```text
Relational database ≠ Message queue
```

### VPC Peering
```text
Network connectivity ≠ Application decoupling
```

For AWS-to-on-premises **network connectivity**, think:
- **Site-to-Site VPN**
- **AWS Direct Connect**

But network connectivity itself does **not** create a decoupled message architecture.

---

# Amazon SQS vs SWF vs Step Functions

| Service | Main purpose | Key clue |
|---|---|---|
| **SQS** | Message queue / decoupling | Buffer work between applications |
| **SWF** | Distributed workflow orchestration | Long-running workflow + distributed workers |
| **Step Functions** | Modern workflow orchestration | State machine + AWS service orchestration |

### Memory
```text
SQS          = QUEUE WORK
SWF          = COORDINATE DISTRIBUTED WORKFLOW
Step Functions = MODERN AWS WORKFLOW
```

---

# Four-way / seven-way decision table

| Service | Model | Main purpose | Pick when |
|---|---|---|---|
| **SQS** | Queue, **PULL** | Application decoupling / buffering | Background work, traffic spikes, asynchronous processing |
| **SNS** | Pub/Sub, **PUSH** | Notification / fan-out | One event should reach many subscribers |
| **SWF** | Workflow orchestration | Distributed workflow coordination | Long-running workflows with distributed workers, including on-premises |
| **Step Functions** | State-machine orchestration | Modern workflow coordination | Coordinate AWS services and workflow logic |
| **Kinesis Data Streams** | Real-time stream | Streaming event processing | Multiple consumers, replay, real-time analytics |
| **Kinesis Data Firehose** | Managed delivery | Stream → supported destination | Minimal operational effort |
| **Kinesis Video Streams** | Video streaming | Video/media ingestion | Cameras and video processing |
| **Amazon MQ** | Traditional message broker | Compatibility | Existing RabbitMQ/ActiveMQ applications |

---

# Question patterns

> **"Web traffic suddenly increases and background workers cannot keep up."**

→ **SQS + Auto Scaling workers**

The queue absorbs the spike while additional workers process the messages.

> **"Messages must be processed in strict order."**

→ **SQS FIFO**

> **"A message must notify three independent applications, and each application should process its own copy."**

→ **SNS + multiple SQS queues**

> **"Different SQS queues should receive different types of SNS messages."**

→ **One SNS topic + SQS subscriptions + SNS filter policies**

> **"Some SQS messages are being processed twice."**

→ **Increase the visibility timeout**

Processing time may be longer than the visibility timeout.

> **"Messages repeatedly fail processing and should be isolated."**

→ **Dead-Letter Queue + MaxReceiveCount**

> **"Consumers are making too many empty SQS polling requests."**

→ **Long polling**

> **"A message is larger than 256 KB."**

→ **Store payload in S3 and send an S3 pointer through SQS**

> **"Two applications need to consume the same real-time stream, and the data may need to be reprocessed."**

→ **Kinesis Data Streams**

> **"Streaming telemetry should be delivered to S3 with minimal operational effort."**

→ **Kinesis Data Firehose**

> **"A company needs to stream video from security cameras to AWS."**

→ **Kinesis Video Streams**

> **"An existing application uses RabbitMQ and should migrate to AWS without rewriting the messaging architecture."**

→ **Amazon MQ**

> **"A long-running workflow needs to coordinate workers running both in AWS and on-premises."**

→ **SWF**

> **"A modern AWS-native workflow needs to coordinate Lambda, ECS, DynamoDB, and other AWS services."**

→ **Step Functions**

> **"A company has AWS and on-premises systems and needs asynchronous decoupling through messages."**

→ **SQS**

> **"Coordinate several sequential and parallel workflow tasks and track their state."**

→ **SWF or Step Functions, depending on the architecture**
- Modern AWS-native designs → **Step Functions**
- Legacy/distributed-worker exam scenarios, especially workers across environments → **SWF**

---

# Pocket card

| Keyword | Answer |
|---|---|
| Decouple applications / buffer spikes | **SQS** |
| One → one work processing | **SQS** |
| Consumer PULLs messages | **SQS** |
| Processing takes too long / duplicate processing | **Increase visibility timeout** |
| Poison messages | **DLQ + MaxReceiveCount** |
| Reduce empty polling | **Long polling (up to 20s)** |
| Strict order | **SQS FIFO** |
| Duplicate-sensitive workload | **SQS FIFO** |
| Message > 256 KB | **S3 + pointer** |
| Retention | **4 days default / 14 days max** |
| One → many | **SNS** |
| SNS pushes to subscribers | **SNS** |
| Different subscribers need different message types | **SNS filtering** |
| One event → multiple durable consumers | **SNS → multiple SQS queues** |
| Long-running distributed workflow | **SWF** |
| AWS + on-premises workflow workers | **SWF** |
| Modern AWS workflow orchestration | **Step Functions** |
| State-machine workflow | **Step Functions** |
| Multiple consumers read the same stream | **Kinesis Data Streams** |
| Replay / reprocess streaming data | **Kinesis Data Streams** |
| Real-time event streaming | **Kinesis Data Streams** |
| Shard throughput | **~1 MB/s in, ~2 MB/s out per traditional shard** |
| Ordering | **Per shard** |
| Stream → S3 / Redshift / OpenSearch / Splunk | **Kinesis Data Firehose** |
| Minimal operational effort for stream delivery | **Firehose** |
| Camera / video stream | **Kinesis Video Streams** |
| Existing RabbitMQ / ActiveMQ | **Amazon MQ** |
| AMQP / MQTT / JMS / STOMP | **Amazon MQ** |
| Workers scale on queue depth | **SQS + CloudWatch + Auto Scaling** |
| AWS + on-premises asynchronous messaging | **SQS** |
| AWS + on-premises distributed workflow | **SWF** |


