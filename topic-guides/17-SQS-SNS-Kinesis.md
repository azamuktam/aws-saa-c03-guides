# Section 17: SQS, SNS & Kinesis — the messaging block

## The idea

Messaging services help **decouple application components** so they do not need to communicate directly.

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

> **Increase the visibility timeout.**

---

## Dead-Letter Queue (DLQ)

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

## Long polling

With short polling, a consumer repeatedly checks the queue and may receive empty responses.

**Long polling** allows the consumer to wait for messages for up to **20 seconds**.

This reduces:

* unnecessary empty responses
* API calls
* polling cost

### Signal

> **Reduce empty polling / reduce polling cost → Long polling**

---

## SQS message size

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

## SQS message retention

SQS retains messages for:

* **4 days by default**
* **14 days maximum**

SQS is a queue, not permanent storage.

---

# Standard vs FIFO

|            | Standard                          | FIFO                                           |
| ---------- | --------------------------------- | ---------------------------------------------- |
| Throughput | Very high / virtually unlimited   | **300 msg/s** or **3,000 msg/s with batching** |
| Delivery   | At-least-once                     | Exactly-once processing support                |
| Ordering   | Best-effort                       | **Strict order**                               |
| Use when   | Highest throughput / normal queue | Ordering or duplicate-sensitive workloads      |

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

---

## SQS + Auto Scaling

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

## SNS push model

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

## SNS message storage

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

The application does not have a queue where the messages can wait for that consumer.

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

## SNS message filtering

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

### Signal

> **Different subscribers should receive different message types → SNS message filtering**

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

## Replay

Kinesis Data Streams retains records so consumers can process them again.

Default retention:

**24 hours**

Maximum retention:

**365 days**

This means applications can go back and reprocess older records within the retention period.

### Signal

> **Need to replay/reprocess streaming data → Kinesis Data Streams**

---

## Shards

Kinesis Data Streams uses **shards** as the basic capacity unit.

A shard provides approximately:

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

> **Need higher stream throughput → increase the number of shards**

---

# Kinesis Data Streams vs SQS

|              | SQS                                    | Kinesis Data Streams                             |
| ------------ | -------------------------------------- | ------------------------------------------------ |
| Main purpose | Decoupling / work queues               | Real-time streaming                              |
| Consumption  | One consumer processes a message       | Multiple applications can read the same data     |
| Replay       | Not designed for replay after deletion | **Yes, within retention period**                 |
| Ordering     | FIFO only                              | Ordering within a shard                          |
| Typical use  | Background jobs, buffering             | Streaming analytics, telemetry, event processing |

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

# Amazon Kinesis Video Streams

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

# Four-way decision table

| Service                   | Model                      | Consumers                                | Storage / Replay                     | Pick when                                                            |
| ------------------------- | -------------------------- | ---------------------------------------- | ------------------------------------ | -------------------------------------------------------------------- |
| **SQS**                   | Queue, PULL                | One consumer per message                 | Message retention, not stream replay | Decouple applications, buffer spikes, distribute work                |
| **SNS**                   | Pub/Sub, PUSH              | Many subscribers                         | Not a durable work queue             | Notify many systems                                                  |
| **Kinesis Data Streams**  | Real-time stream           | Many consumers can read the same data    | **24h–365d, replayable**             | Streaming analytics, multiple consumers, reprocessing                |
| **Kinesis Data Firehose** | Managed delivery           | AWS manages delivery                     | Destination-based delivery           | Deliver streams to S3/Redshift/OpenSearch/Splunk with minimal effort |
| **Kinesis Video Streams** | Video streaming            | Video processing / playback applications | Video stream storage                 | Cameras, video feeds, media processing                               |
| **Amazon MQ**             | Traditional message broker | Broker clients                           | Broker semantics                     | Existing RabbitMQ/ActiveMQ applications                              |

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

> **"Some SQS messages are being processed twice."**

→ **Increase the visibility timeout**

The processing time is probably longer than the visibility timeout.

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

> **"An existing application uses RabbitMQ and should migrate to AWS without rewriting the application."**

→ **Amazon MQ**

---

> **"Different SNS subscribers should receive different types of messages."**

→ **SNS message filtering**

---

# Pocket card

| Keyword                                            | Answer                              |
| -------------------------------------------------- | ----------------------------------- |
| Decouple applications / buffer spikes              | **SQS**                             |
| One message → one worker                           | **SQS**                             |
| Consumer PULLs messages                            | **SQS**                             |
| Processing takes too long / duplicate processing   | **Increase visibility timeout**     |
| Poison messages                                    | **DLQ + MaxReceiveCount**           |
| Reduce empty polling                               | **Long polling (up to 20s)**        |
| Strict order                                       | **SQS FIFO**                        |
| Exactly-once processing requirement                | **SQS FIFO**                        |
| Message > 256 KB                                   | **S3 + pointer**                    |
| Retention                                          | **4 days default / 14 days max**    |
| One → many                                         | **SNS**                             |
| SNS pushes to subscribers                          | **SNS**                             |
| Different subscribers need different message types | **SNS filtering**                   |
| One event → multiple durable consumers             | **SNS → multiple SQS queues**       |
| Multiple consumers read the same stream            | **Kinesis Data Streams**            |
| Replay / reprocess streaming data                  | **Kinesis Data Streams**            |
| Real-time event streaming                          | **Kinesis Data Streams**            |
| Shard throughput                                   | **1 MB/s in, 2 MB/s out**           |
| Ordering                                           | **Per shard**                       |
| Stream → S3/Redshift/OpenSearch/Splunk             | **Kinesis Data Firehose**           |
| Minimal operational effort for stream delivery     | **Firehose**                        |
| Camera / video stream                              | **Kinesis Video Streams**           |
| Existing RabbitMQ / ActiveMQ                       | **Amazon MQ**                       |
| AMQP / MQTT / JMS / STOMP                          | **Amazon MQ**                       |
| Workers scale on queue depth                       | **SQS + CloudWatch + Auto Scaling** |

---

# Final memory

```text
SQS
= QUEUE
= DECOUPLING
= ONE WORKER PER MESSAGE
= PULL

SNS
= PUB/SUB
= ONE MESSAGE → MANY SUBSCRIBERS
= PUSH

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

Need multiple consumers to read the same real-time data?
→ Kinesis Data Streams

Need to replay streaming data?
→ Kinesis Data Streams

Need to deliver streaming data to S3/Redshift/OpenSearch/Splunk with minimal effort?
→ Kinesis Data Firehose

Need to stream video?
→ Kinesis Video Streams

Existing RabbitMQ / ActiveMQ application?
→ Amazon MQ
```
