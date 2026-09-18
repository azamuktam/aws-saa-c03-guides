# Section 17: SQS, SNS, SWF, Step Functions & Kinesis — the messaging block

## The idea

Messaging and integration services help **decouple application components** so they do not need to communicate directly.

Without a queue:

```text
Web server
    ↓
Order processor
```

Problems:

* If the processor is slow, the web request waits.
* If the processor fails, the request can fail or the work can be lost.
* If traffic suddenly increases, the processor may become overloaded.

With SQS:

```text
Web server
    ↓
SQS queue
    ↓
Order processor
```

Now:

* The web tier can send the message and continue.
* Messages wait in the queue until a worker processes them.
* Traffic spikes are absorbed by the queue.
* Workers can scale based on the number of messages.

This is **decoupling**.

---

# SQS — Simple Queue Service

**SQS = managed message queue for decoupling applications.**

Producers send messages to the queue.

Consumers **pull** messages from the queue and process them.

For normal SQS usage, a message is processed by **one consumer at a time**.

```text
Producer
    ↓
SQS
    ↓
Consumer
```

---

## Visibility timeout

This is one of the most important SQS concepts.

When a consumer receives a message:

```text
Message
   ↓
Received by worker
   ↓
Temporarily hidden
```

The message is **not deleted immediately**.

The default visibility timeout is **30 seconds**.

If the worker successfully processes the message, it should delete it.

```text
Receive
   ↓
Process successfully
   ↓
Delete message
```

If the worker crashes before deleting it, the visibility timeout expires and the message becomes visible again.

```text
Receive
   ↓
Worker crashes
   ↓
Visibility timeout expires
   ↓
Message becomes visible again
   ↓
Another worker can process it
```

### Common exam question

> **"Messages are being processed more than once."**

A common cause is that processing takes longer than the visibility timeout.

Example:

```text
Visibility timeout = 30 seconds
Processing time    = 60 seconds
```

The message becomes visible again after 30 seconds even though the first worker is still processing it.

### Fix

→ **Increase the visibility timeout**

---

# Dead-Letter Queue (DLQ)

A **Dead-Letter Queue** stores messages that repeatedly fail processing.

You configure a **MaxReceiveCount**.

Example:

```text
Main SQS queue
      ↓
Message fails repeatedly
      ↓
MaxReceiveCount reached
      ↓
Dead-Letter Queue
```

This prevents a bad message from continuously returning to the main queue.

### Signal

> **Messages repeatedly fail processing → DLQ + MaxReceiveCount**

---

# Long polling

With short polling, a consumer repeatedly checks the queue and may receive empty responses.

**Long polling** allows the consumer to wait for messages for up to **20 seconds**.

This reduces:

* unnecessary empty responses
* API calls
* polling cost

### Signal

> **Reduce empty polling / reduce polling cost → Long polling**

---

# SQS message size

The maximum SQS message size is **256 KB**.

If the payload is larger:

```text
Large payload
    ↓
Store object in S3
    ↓
Send S3 location in SQS message
```

### Signal

> **Message larger than 256 KB → S3 + pointer in SQS**

---

# SQS message retention

SQS retains messages for:

* **4 days by default**
* **14 days maximum**

SQS is a queue, not permanent storage.

---

# Standard vs FIFO

|            | Standard                          | FIFO                                                                                                |
| ---------- | --------------------------------- | --------------------------------------------------------------------------------------------------- |
| Throughput | Very high / virtually unlimited   | **300 msg/s** or **3,000 msg/s with batching** per FIFO queue without high-throughput FIFO settings |
| Delivery   | At-least-once                     | Designed to avoid duplicates through FIFO deduplication                                             |
| Ordering   | Best-effort                       | **Strict order**                                                                                    |
| Use when   | Highest throughput / normal queue | Ordering or duplicate-sensitive workloads                                                           |

## Standard Queue

Use Standard when:

* very high throughput is required
* occasional duplicate delivery is acceptable
* strict ordering is not required

## FIFO Queue

Use FIFO when:

* message order matters
* duplicate processing must be minimized
* messages must be processed in strict order

### Signal

> **"Messages must be processed in strict order."**

→ **SQS FIFO**

### Important nuance

FIFO provides **exactly-once processing support / deduplication behavior**, but your application should still be designed carefully for retries and side effects.

---

# SQS + Auto Scaling

A common architecture is:

```text
Application
    ↓
SQS
    ↓
Auto Scaling Group
    ↓
EC2 workers
```

CloudWatch can monitor the queue, especially:

`ApproximateNumberOfMessagesVisible`

As the queue grows:

```text
More messages
    ↓
Scale out workers
```

As the queue becomes smaller:

```text
Fewer messages
    ↓
Scale in workers
```

### Signal

> **Workers should scale based on queue depth → SQS + CloudWatch + Auto Scaling**

---

# SNS — Simple Notification Service

**SNS = managed publish/subscribe messaging service.**

A producer publishes a message to an **SNS topic**.

SNS then sends the message to its subscribers.

```text
Publisher
    ↓
SNS Topic
    ↓
Subscribers
```

Possible subscribers include:

* SQS
* Lambda
* HTTP/S endpoints
* Email
* SMS
* mobile push notifications

The key idea is:

> **One message can be delivered to many subscribers.**

---

# SNS push model

SNS uses a **push** model.

```text
Publisher
    ↓
SNS Topic
    ↓
Subscriber 1
Subscriber 2
Subscriber 3
```

This is different from SQS:

```text
SQS
= consumers pull messages

SNS
= SNS pushes messages to subscribers
```

---

# SNS message storage

SNS is not a long-term message queue.

For SAA questions, remember:

> **SNS is mainly for notification and fan-out, not for storing messages for consumers to process later.**

If a downstream application needs durable buffering, use:

```text
SNS
 ↓
SQS
 ↓
Consumer
```

---

# SNS Fan-out

One of the most important SNS patterns is **fan-out**.

Suppose one event needs to be processed by three separate systems:

```text
                    → SQS → Analytics
                   /
Producer → SNS Topic
                   \
                    → SQS → Fulfillment
                   /
                    → SQS → Fraud
```

Each SQS queue receives its own copy of the message.

This gives each application an independent queue.

### Why use SQS after SNS?

Suppose the fraud service is unavailable.

Without SQS:

```text
SNS → Fraud service
```

There is no queue where the messages can wait for that consumer.

With SQS:

```text
SNS
 ↓
SQS
 ↓
Fraud service
```

The queue can hold the messages until the service recovers.

### Signal

> **One event → multiple independent consumers → SNS + multiple SQS queues**

---

# SNS message filtering

SNS supports **subscription filter policies**.

This allows different subscribers to receive only the messages they are interested in.

Example:

```text
SNS Topic
    ↓
Billing subscription → refund events
Fraud subscription   → suspicious events
Shipping subscription → shipment events
```

The publisher sends **one message** to the SNS topic with message attributes or body values used by the filter policy.

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

### Signal

> **Different subscribers should receive different message types → SNS message filtering**

---

# SWF — Simple Workflow Service

**Amazon SWF = managed workflow orchestration for distributed applications.**

SWF coordinates **long-running, asynchronous workflows** made up of multiple tasks.

The important exam concept is that the workers performing those tasks can run:

* on Amazon EC2
* on AWS infrastructure
* on **on-premises servers**
* in other environments that can reach SWF

A classic exam pattern is a company with workloads in both AWS and on-premises that needs a **distributed workflow**.

---

## SWF basic architecture

```text
                    Amazon SWF
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
       Workflow logic          Task coordination
             │
       ┌─────┴─────┐
       ↓           ↓
   EC2 worker   On-prem worker
```

SWF coordinates things such as:

* task scheduling
* task dependencies
* workflow state
* retries
* failures
* sequential tasks
* parallel tasks

The workers perform the actual business logic.

---

# SWF vs SQS

These services are related, but they solve different problems.

### SQS

**SQS = message queue / application decoupling**

```text
Producer
   ↓
  SQS
   ↓
Consumer
```

Main idea:

> **Put work into a queue so another application can process it later.**

### SWF

**SWF = workflow orchestration**

```text
Start workflow
      ↓
Task A
      ↓
Task B
      ↓
Task C
      ↓
Complete
```

Main idea:

> **Coordinate a multi-step workflow and keep track of workflow progress.**

---

# SWF and AWS + on-premises

This is a particularly useful exam clue.

### SQS example

```text
On-premises application
        ↓
       SQS
        ↓
     EC2 worker
```

SQS decouples the applications through messages.

### SWF example

```text
              SWF
               │
       ┌───────┴───────┐
       ↓               ↓
On-premises worker   EC2 worker
       │               │
       └───────┬───────┘
               ↓
        Workflow continues
```

### Exam signal

> **"Coordinate tasks performed by workers running both on-premises and in AWS."**

→ **SWF**

---

# SWF workflow components

A simplified SWF workflow contains:

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

The **decider** determines what should happen next in the workflow.

Example:

```text
Payment successful?
       │
   ┌───┴───┐
   ↓       ↓
  YES      NO
   ↓       ↓
Ship     Cancel
```

## Activity worker

An **activity worker** performs the actual business task.

Examples:

```text
Payment worker
Shipping worker
Document-processing worker
On-premises validation worker
```

Workers poll SWF for tasks and report the results back.

---

# Sequential and parallel workflows

SWF can coordinate both sequential and parallel activities.

### Sequential

```text
Task A
  ↓
Task B
  ↓
Task C
```

### Parallel

```text
        ┌→ Task A ─┐
Start ──┤          ├→ Continue
        └→ Task B ─┘
```

---

# When to choose SWF

Choose SWF when the question emphasizes:

* distributed workflow
* long-running workflow
* asynchronous workflow
* multiple workflow steps
* task coordination
* workflow state
* workers running on **AWS and on-premises**
* sequential or parallel activities

### Strong exam signal

> **"Coordinate tasks performed by workers running both on-premises and in AWS."**

→ **SWF**

---

# SWF vs Step Functions

For **modern AWS architecture**, **AWS Step Functions** is the more important modern workflow orchestration service.

SWF is a **legacy/older workflow service** that can still appear in SAA-style questions.

```text
SWF
= older distributed workflow service
= custom workers
= can coordinate AWS + on-premises workers

Step Functions
= modern AWS workflow orchestration
= state-machine based
= integrates directly with many AWS services
```

Step Functions can also interact with external workers through **Activities**, so the distinction is not simply "SWF = on-premises and Step Functions = AWS."

For exam questions, the strongest SWF clue is:

> **Distributed, long-running workflows with workers running across environments, including on-premises.**

---

# Step Functions — modern workflow orchestration

**AWS Step Functions = managed workflow orchestration using state machines.**

It lets you coordinate multiple AWS services and application steps into a workflow.

Example:

```text
Order
  ↓
Validate
  ↓
Charge payment
  ↓
Ship
  ↓
Send notification
```

It can coordinate services such as:

* Lambda
* ECS
* SNS
* SQS
* DynamoDB
* AWS Batch
* Glue
* other AWS services

---

## Step Functions state machine

A workflow is represented as states and transitions.

Example:

```text
Start
  ↓
Validate Order
  ↓
Payment
  ↓
Choice
 ┌──┴──┐
 ↓     ↓
Ship  Cancel
  \    /
   ↓  ↓
    End
```

### Common state types

* Task
* Choice
* Wait
* Parallel
* Map
* Pass
* Succeed
* Fail

### Signal

> **"Coordinate several AWS services through a visual/state-machine workflow."**

→ **Step Functions**

---

# Step Functions vs SQS

These are not replacements for each other.

```text
SQS
= queue work

Step Functions
= orchestrate workflow steps
```

Example:

```text
Order service
    ↓
   SQS
    ↓
Worker
```

SQS simply buffers/distributes work.

But:

```text
Step Functions
    ↓
Validate
    ↓
Charge
    ↓
Choice
    ↓
Ship / Cancel
```

Step Functions manages the **workflow logic and state**.

---

# Kinesis Data Streams

**Kinesis Data Streams = real-time streaming service for continuously arriving data.**

Typical use cases include:

* clickstream data
* IoT telemetry
* application events
* logs
* real-time analytics

```text
Producers
    ↓
Kinesis Data Streams
    ↓
Consumers
```

The important difference from SQS is that **multiple applications can read the same stream of records**.

For example:

```text
             → Analytics application
            /
Producers → Kinesis Data Streams
            \
             → Fraud detection application
```

Both applications can process the same data.

---

# Replay

Kinesis Data Streams retains records so consumers can process them again.

Default retention:

**24 hours**

Maximum retention:

**365 days**

This means applications can go back and reprocess older records within the retention period.

### Signal

> **Need to replay/reprocess streaming data → Kinesis Data Streams**

---

# Shards

Kinesis Data Streams uses **shards** as a basic capacity unit.

A traditional provisioned shard provides approximately:

* **1 MB/s write throughput**
* **2 MB/s read throughput**

Records are assigned to shards based on the **partition key**.

Ordering is guaranteed **within a shard**.

```text
Partition key
      ↓
   Shard
      ↓
Ordered records
```

### Signal

> **Need higher provisioned stream throughput → increase the number of shards**

---

# Kinesis Data Streams vs SQS

|              | SQS                                  | Kinesis Data Streams                             |
| ------------ | ------------------------------------ | ------------------------------------------------ |
| Main purpose | Decoupling / work queues             | Real-time streaming                              |
| Consumption  | One consumer processes a message     | Multiple applications can read the same stream   |
| Replay       | Not designed for stream-style replay | **Yes, within retention period**                 |
| Ordering     | FIFO only                            | Ordering within a shard                          |
| Typical use  | Background jobs, buffering           | Streaming analytics, telemetry, event processing |

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

Typical destinations include:

* Amazon S3
* Amazon Redshift
* Amazon OpenSearch Service
* Splunk

Firehose can also use **Lambda for data transformation** before delivery.

Typical architecture:

```text
Streaming data
      ↓
Kinesis Data Firehose
      ↓
S3 / Redshift / OpenSearch / Splunk
```

Firehose automatically buffers incoming data before delivery, so it is **near real time rather than instant delivery**.

### Main advantage

> **Very low operational effort.**

You do not need to write your own consumer application just to deliver the stream to the supported destination.

### Signal

> **"Deliver streaming data to S3 with minimal operational overhead."**

→ **Kinesis Data Firehose**

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

* applications need to read the stream themselves
* multiple consumers need the same records
* you need replay/reprocessing
* custom real-time processing is required

### Choose Firehose when:

* data mainly needs to be delivered to S3, Redshift, OpenSearch, or Splunk
* minimal operational effort is required
* you do not need custom stream consumers

---

# Kinesis Video Streams

**Amazon Kinesis Video Streams = capture, transport, store, and process video streams for analysis and playback.**

It is designed for **video and other time-encoded media**, especially from sources such as:

* security cameras
* smartphones
* connected devices
* IoT cameras

Typical architecture:

```text
Camera / device
       ↓
Kinesis Video Streams
       ↓
Applications / video processing / playback
```

It can be used to securely stream video data to AWS for:

* real-time or near-real-time processing
* machine learning analysis
* video playback
* storage and later processing

### Important distinction

```text
Kinesis Data Streams
= application/event data
= logs, clicks, telemetry, events

Kinesis Video Streams
= video / time-encoded media
= cameras, video feeds
```

### Signal

> **"Stream video from cameras/devices to AWS."**

→ **Kinesis Video Streams**

---

# Amazon MQ

**Amazon MQ = managed message broker for existing applications that use traditional messaging protocols.**

It supports:

* RabbitMQ
* ActiveMQ

Common protocol keywords:

* AMQP
* MQTT
* JMS
* STOMP

Use Amazon MQ when an existing application already depends on these traditional broker technologies and migrating to SQS/SNS would require significant changes.

```text
Existing application
        ↓
RabbitMQ / ActiveMQ
        ↓
Amazon MQ
```

### Signal

> **Existing RabbitMQ / ActiveMQ application → Amazon MQ**

> **AMQP / MQTT / JMS / STOMP → Amazon MQ**

---

# Decoupled AWS + On-Premises Architecture

This is an important SAA pattern.

Suppose a company has:

```text
AWS
  └── EC2 application

On-premises
  └── Internal application
```

The goal is to decouple them.

Several AWS integration services can be used depending on the requirement.

### SQS

Use SQS when the requirement is:

> **Asynchronous message-based decoupling**

```text
On-premises
    ↓
   SQS
    ↓
   EC2
```

### SWF

Use SWF when the requirement is:

> **Coordinate a distributed workflow across workers in AWS and on-premises**

```text
              SWF
             /   \
            /     \
      EC2 worker  On-prem worker
```

### Step Functions

Use Step Functions when the requirement is:

> **Modern workflow orchestration using AWS services/state machines**

---

# What is NOT a decoupling service?

Several AWS services may connect or store data but do not themselves provide application decoupling.

### DynamoDB

```text
Database
≠
Message queue
```

### RDS

```text
Relational database
≠
Message queue
```

### VPC Peering

```text
Network connectivity
≠
Application decoupling
```

For AWS-to-on-premises network connectivity, think:

```text
Site-to-Site VPN
or
AWS Direct Connect
```

But network connectivity itself does not create a decoupled message architecture.

---

# Amazon SQS vs SWF vs Step Functions

| Service            | Main purpose                       | Key clue                                    |
| ------------------ | ---------------------------------- | ------------------------------------------- |
| **SQS**            | Message queue / decoupling         | Buffer work between applications            |
| **SWF**            | Distributed workflow orchestration | Long-running workflow + distributed workers |
| **Step Functions** | Modern workflow orchestration      | State machine + AWS service orchestration   |

### Memory

```text
SQS
= QUEUE WORK

SWF
= COORDINATE DISTRIBUTED WORKFLOW

Step Functions
= MODERN AWS WORKFLOW
```

---

# Four-way / seven-way decision table

| Service                   | Model                       | Main purpose                       | Pick when                                                              |
| ------------------------- | --------------------------- | ---------------------------------- | ---------------------------------------------------------------------- |
| **SQS**                   | Queue, PULL                 | Application decoupling / buffering | Background work, traffic spikes, asynchronous processing               |
| **SNS**                   | Pub/Sub, PUSH               | Notification / fan-out             | One event should reach many subscribers                                |
| **SWF**                   | Workflow orchestration      | Distributed workflow coordination  | Long-running workflows with distributed workers, including on-premises |
| **Step Functions**        | State-machine orchestration | Modern workflow coordination       | Coordinate AWS services and workflow logic                             |
| **Kinesis Data Streams**  | Real-time stream            | Streaming event processing         | Multiple consumers, replay, real-time analytics                        |
| **Kinesis Data Firehose** | Managed delivery            | Stream → supported destination     | Minimal operational effort                                             |
| **Kinesis Video Streams** | Video streaming             | Video/media ingestion              | Cameras and video processing                                           |
| **Amazon MQ**             | Traditional message broker  | Compatibility                      | Existing RabbitMQ/ActiveMQ applications                                |

---

# Question patterns

> **"Web traffic suddenly increases and background workers cannot keep up."**

→ **SQS + Auto Scaling workers**

The queue absorbs the spike while additional workers process the messages.

---

> **"Messages must be processed in strict order."**

→ **SQS FIFO**

---

> **"A message must notify three independent applications, and each application should process its own copy."**

→ **SNS + multiple SQS queues**

---

> **"Different SQS queues should receive different types of SNS messages."**

→ **One SNS topic + SQS subscriptions + SNS filter policies**

---

> **"Some SQS messages are being processed twice."**

→ **Increase the visibility timeout**

The processing time may be longer than the visibility timeout.

---

> **"Messages repeatedly fail processing and should be isolated."**

→ **Dead-Letter Queue + MaxReceiveCount**

---

> **"Consumers are making too many empty SQS polling requests."**

→ **Long polling**

---

> **"A message is larger than 256 KB."**

→ **Store the payload in S3 and send an S3 pointer through SQS**

---

> **"Two applications need to consume the same real-time stream, and the data may need to be reprocessed."**

→ **Kinesis Data Streams**

---

> **"Streaming telemetry should be delivered to S3 with minimal operational effort."**

→ **Kinesis Data Firehose**

---

> **"A company needs to stream video from security cameras to AWS."**

→ **Kinesis Video Streams**

---

> **"An existing application uses RabbitMQ and should migrate to AWS without rewriting the messaging architecture."**

→ **Amazon MQ**

---

> **"A long-running workflow needs to coordinate workers running both in AWS and on-premises."**

→ **SWF**

---

> **"A modern AWS-native workflow needs to coordinate Lambda, ECS, DynamoDB, and other AWS services."**

→ **Step Functions**

---

> **"A company has AWS and on-premises systems and needs asynchronous decoupling through messages."**

→ **SQS**

---

> **"Coordinate several sequential and parallel workflow tasks and track their state."**

→ **SWF or Step Functions, depending on the architecture**

For modern AWS-native designs:

→ **Step Functions**

For legacy/distributed-worker exam scenarios, especially workers across environments:

→ **SWF**

---

# Pocket card

| Keyword                                            | Answer                                            |
| -------------------------------------------------- | ------------------------------------------------- |
| Decouple applications / buffer spikes              | **SQS**                                           |
| One → one work processing                          | **SQS**                                           |
| Consumer PULLs messages                            | **SQS**                                           |
| Processing takes too long / duplicate processing   | **Increase visibility timeout**                   |
| Poison messages                                    | **DLQ + MaxReceiveCount**                         |
| Reduce empty polling                               | **Long polling (up to 20s)**                      |
| Strict order                                       | **SQS FIFO**                                      |
| Duplicate-sensitive workload                       | **SQS FIFO**                                      |
| Message > 256 KB                                   | **S3 + pointer**                                  |
| Retention                                          | **4 days default / 14 days max**                  |
| One → many                                         | **SNS**                                           |
| SNS pushes to subscribers                          | **SNS**                                           |
| Different subscribers need different message types | **SNS filtering**                                 |
| One event → multiple durable consumers             | **SNS → multiple SQS queues**                     |
| Long-running distributed workflow                  | **SWF**                                           |
| AWS + on-premises workflow workers                 | **SWF**                                           |
| Modern AWS workflow orchestration                  | **Step Functions**                                |
| State-machine workflow                             | **Step Functions**                                |
| Multiple consumers read the same stream            | **Kinesis Data Streams**                          |
| Replay / reprocess streaming data                  | **Kinesis Data Streams**                          |
| Real-time event streaming                          | **Kinesis Data Streams**                          |
| Shard throughput                                   | **~1 MB/s in, ~2 MB/s out per traditional shard** |
| Ordering                                           | **Per shard**                                     |
| Stream → S3 / Redshift / OpenSearch / Splunk       | **Kinesis Data Firehose**                         |
| Minimal operational effort for stream delivery     | **Firehose**                                      |
| Camera / video stream                              | **Kinesis Video Streams**                         |
| Existing RabbitMQ / ActiveMQ                       | **Amazon MQ**                                     |
| AMQP / MQTT / JMS / STOMP                          | **Amazon MQ**                                     |
| Workers scale on queue depth                       | **SQS + CloudWatch + Auto Scaling**               |
| AWS + on-premises asynchronous messaging           | **SQS**                                           |
| AWS + on-premises distributed workflow             | **SWF**                                           |

---

# Final memory

```text
SQS
= QUEUE
= DECOUPLING
= WORK
= PULL

SNS
= PUB/SUB
= FAN-OUT
= NOTIFY
= PUSH

SWF
= DISTRIBUTED WORKFLOW
= COORDINATE TASKS
= LONG-RUNNING WORKFLOWS
= AWS + ON-PREMISES WORKERS

Step Functions
= MODERN WORKFLOW ORCHESTRATION
= STATE MACHINE
= AWS SERVICE INTEGRATION

Kinesis Data Streams
= REAL-TIME APPLICATION DATA
= MULTIPLE CONSUMERS
= REPLAY

Kinesis Data Firehose
= STREAM → AWS DESTINATION
= MINIMAL OPERATIONAL EFFORT

Kinesis Video Streams
= VIDEO / CAMERA STREAMING

Amazon MQ
= EXISTING RABBITMQ / ACTIVEMQ
= TRADITIONAL BROKER PROTOCOLS
```

## The key decision

```text
Need to decouple workers?
→ SQS

Need to notify many subscribers?
→ SNS

Need different subscribers to receive different message types?
→ SNS filtering

Need a long-running distributed workflow across AWS/on-premises workers?
→ SWF

Need modern AWS workflow orchestration?
→ Step Functions

Need multiple consumers to read the same real-time data?
→ Kinesis Data Streams

Need to replay/reprocess streaming data?
→ Kinesis Data Streams

Need to deliver streaming data to S3/Redshift/OpenSearch/Splunk with minimal effort?
→ Kinesis Data Firehose

Need to stream video?
→ Kinesis Video Streams

Existing RabbitMQ / ActiveMQ application?
→ Amazon MQ
```
