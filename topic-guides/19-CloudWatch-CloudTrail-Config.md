# Section 19: CloudWatch, CloudTrail & AWS Config

## The idea

These services all monitor AWS, but they answer **different questions**.

| Service        | Main question                                                   |
| -------------- | --------------------------------------------------------------- |
| **CloudWatch** | **How is it performing?**                                       |
| **CloudTrail** | **Who did what?**                                               |
| **AWS Config** | **What was the resource configured like, and is it compliant?** |
| **X-Ray**      | **Which part of the application request is slow?**              |

The easiest way to recognize them:

```text
Performance / metrics / logs / alarms
→ CloudWatch

Who created / deleted / changed something?
→ CloudTrail

What did the configuration look like?
Is the resource compliant?
→ AWS Config

Which service in a request chain is slow?
→ X-Ray
```

---

## CloudWatch — "How is it performing?"

**Amazon CloudWatch = monitoring for metrics, logs, and alarms.**

It helps you monitor:

* CPU usage
* network traffic
* application logs
* error counts
* latency
* alarms
* dashboards

### CloudWatch Metrics

Metrics are **numbers measured over time**.

Examples:

```text
CPU = 75%
Request count = 10,000
Latency = 250 ms
```

### Important EC2 metric trap

EC2 provides many standard metrics, such as:

* CPU utilization
* network traffic
* some disk-related metrics

But metrics such as:

* **memory usage**
* **free disk space**

are **not standard EC2 metrics**.

To collect these, install the **CloudWatch Agent**.

### Exam pattern

> *"Alert when EC2 memory usage exceeds 80%."*

→ **CloudWatch Agent + metric + CloudWatch Alarm**

---

## CloudWatch Enhanced Monitoring vs normal CloudWatch

For **RDS**, there is an important distinction.

### Normal CloudWatch RDS monitoring

CloudWatch can show the overall database instance metrics.

Example:

```text
RDS CPUUtilization = 75%
```

This tells you about the **RDS instance as a whole**.

### RDS Enhanced Monitoring

**Enhanced Monitoring = OS-level monitoring of the RDS instance.**

It provides more detailed information such as:

* CPU usage
* memory usage
* processes
* process-level CPU usage
* process-level memory usage
* load
* filesystem information

Example:

```text
RDS instance
│
├── Process A → CPU 20%, memory 300 MB
├── Process B → CPU 35%, memory 500 MB
└── Process C → CPU 10%, memory 100 MB
```

### Exam pattern

> *"Monitor the CPU and memory used by individual processes on an RDS instance."*

→ **RDS Enhanced Monitoring**

### Easy distinction

```text
Overall RDS CPU
→ CloudWatch

CPU / memory / processes at OS level
→ Enhanced Monitoring
```

---

## CloudWatch Alarms

A CloudWatch Alarm watches a metric and takes action when a condition is met.

Example:

```text
CPU > 80%
    ↓
CloudWatch Alarm
    ↓
SNS notification
```

An alarm can also trigger things such as:

* Auto Scaling
* EC2 actions
* SNS notifications

### Composite alarms

A **Composite Alarm** combines multiple alarms.

Example:

```text
CPU > 80%
AND
Error rate > 5%
        ↓
Composite Alarm
```

Use it when you want to reduce unnecessary alerts.

### Exam pattern

> *"Several alarms are firing and the company wants fewer unnecessary notifications."*

→ **Composite Alarm**

---

## CloudWatch Logs

CloudWatch Logs stores application and system logs.

The structure is:

```text
Log Group
   ├── Log Stream
   ├── Log Stream
   └── Log Stream
```

A **log group** usually represents an application or service.

A **log stream** contains logs from one source, such as an instance or container.

### Metric Filters

A **metric filter** searches logs for a pattern and turns the result into a metric.

Example:

```text
Application log:
ERROR
ERROR
INFO
ERROR
```

Metric filter:

```text
Count ERROR
    ↓
CloudWatch metric
    ↓
Alarm if > 100
```

### Exam pattern

> *"Trigger an alarm when the application writes more than 100 ERROR messages in 5 minutes."*

→ **CloudWatch Logs Metric Filter + Alarm**

---

## CloudWatch Logs Insights

**Logs Insights = query and analyze logs interactively.**

Use it when you want to search and analyze large amounts of CloudWatch Logs.

Example:

> "Find all requests that took more than 2 seconds."

→ **CloudWatch Logs Insights**

### Remember

```text
Metric Filter
= turn log patterns into metrics

Logs Insights
= query/analyze logs
```

---

## CloudWatch Logs Subscription Filters

A **subscription filter** sends log events somewhere else in **near real time**.

Common destinations include:

* Kinesis Data Streams
* Kinesis Data Firehose
* Lambda

Example:

```text
CloudWatch Logs
      ↓
Subscription Filter
      ↓
Lambda / Kinesis
```

### Exam pattern

> *"Process log entries in real time as they arrive."*

→ **CloudWatch Logs Subscription Filter**

---

## CloudWatch Dashboards

CloudWatch dashboards display metrics and monitoring information in one place.

They can show resources and metrics from different Regions.

### Example

```text
US
CPU / Errors / Latency
+
Europe
CPU / Errors / Latency
+
Asia
CPU / Errors / Latency
```

→ One CloudWatch dashboard

---

# CloudTrail — "Who did what?"

**AWS CloudTrail = audit log of AWS API activity.**

It records API actions such as:

* who made the request
* what action they performed
* when it happened
* where the request came from
* what resource was affected

Example:

```text
Alice
   ↓
Terminate EC2 instance
   ↓
CloudTrail
```

### Example question

> "Who terminated the production EC2 instance yesterday?"

→ **CloudTrail**

---

## CloudTrail Event History

CloudTrail provides **90 days of management event history** without requiring you to create a trail.

For longer-term retention:

```text
CloudTrail Trail
      ↓
S3
```

Store the logs in S3 for long-term retention.

### Exam pattern

> *"Keep API activity records for several years."*

→ **CloudTrail Trail → S3**

---

## CloudTrail Management Events vs Data Events

This is very important.

### Management events

These are **control-plane operations**.

Examples:

* create EC2 instance
* terminate EC2 instance
* create S3 bucket
* change IAM policy

Management events are logged by default in CloudTrail event history.

### Data events

These are **resource-level operations**.

Examples:

* reading an S3 object
* deleting an S3 object
* invoking a Lambda function

Data events must generally be **enabled explicitly** and can incur additional charges.

### Exam trap

> *"Who deleted a specific object from S3?"*

→ **CloudTrail S3 data events**

Normal management events are not enough.

---

## CloudTrail Log File Integrity Validation

This helps prove that CloudTrail log files have **not been modified after delivery**.

Useful for:

* security investigations
* compliance
* forensics
* proving log integrity

### Exam pattern

> *"Logs must be tamper-evident for forensic purposes."*

→ **CloudTrail log file integrity validation**

---

## CloudTrail Insights

**CloudTrail Insights detects unusual API activity.**

Example:

Normally:

```text
TerminateInstances = 2 per day
```

Suddenly:

```text
TerminateInstances = 500 in 10 minutes
```

CloudTrail Insights can identify this unusual behavior.

### Remember

> **CloudTrail = API audit**

> **CloudTrail Insights = unusual API activity**

---

# AWS Config — "What was the configuration, and is it compliant?"

**AWS Config = resource configuration history + compliance checking.**

It answers questions such as:

> "What did this security group look like last Tuesday?"

or:

> "Does this security group comply with our security rules?"

---

## Configuration history

AWS Config records changes to resource configurations.

Example:

```text
Monday
SG allows port 22 from company IP

Tuesday
SG changed to 0.0.0.0/0

Wednesday
SG changed again
```

You can investigate the configuration history.

### Exam pattern

> *"Show what a security group's configuration was one week ago."*

→ **AWS Config**

---

## Config Rules

AWS Config Rules evaluate whether resources meet a requirement.

Example:

```text
Rule:
SSH must not be open to the Internet

        ↓

Security Group
0.0.0.0/0:22
        ↓
NONCOMPLIANT
```

Rules can be:

* AWS-managed
* custom rules

### Exam pattern

> *"Identify security groups that allow SSH from the Internet."*

→ **AWS Config Rule**

---

## Config Remediation

AWS Config can automatically trigger remediation when a resource becomes noncompliant.

A common pattern is:

```text
Config Rule
     ↓
Noncompliant
     ↓
SSM Automation
     ↓
Fix the resource
```

### Example

> "Automatically remove a world-open SSH rule."

→ **Config Rule + remediation action**

---

## Important Config limitation

**AWS Config does not prevent the action from happening.**

It detects configuration and compliance problems.

If the question says:

> **"Prevent users from creating this resource or performing this action."**

Think about:

* IAM
* SCPs
* other preventive controls

not Config.

### Easy distinction

```text
Config
= detect / record / evaluate

SCP / IAM
= prevent / control permissions
```

---

# X-Ray — "Which part of the request is slow?"

**AWS X-Ray = distributed tracing.**

It follows one request as it moves through multiple services.

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
  ↓
DynamoDB
```

Suppose the whole request takes:

```text
2 seconds
```

X-Ray can help identify:

```text
API Gateway   50 ms
Lambda        100 ms
Service A     150 ms
Service B     1,600 ms  ← problem
DynamoDB      100 ms
```

### Exam pattern

> *"A request passes through ten microservices. Find which service is causing the latency."*

→ **X-Ray**

### CloudWatch vs X-Ray

```text
CloudWatch
= this service is slow

X-Ray
= this specific hop in the request is slow
```

---

# The four-way comparison

| Service        | Main question                                | Example                     |
| -------------- | -------------------------------------------- | --------------------------- |
| **CloudWatch** | How is it performing?                        | CPU = 90%                   |
| **CloudTrail** | Who did what?                                | Alice terminated EC2        |
| **AWS Config** | What was the configuration? Is it compliant? | SG allowed SSH last Tuesday |
| **X-Ray**      | Which part of the request is slow?           | Service B adds 1.6 seconds  |

### Memory

```text
CloudWatch
= PERFORMANCE

CloudTrail
= API AUDIT

Config
= CONFIGURATION + COMPLIANCE

X-Ray
= REQUEST TRACE
```

---

## Question patterns

> *"Determine who terminated a production EC2 instance last week."* → **CloudTrail**

> *"Alert when EC2 memory usage exceeds 80%."* → **CloudWatch Agent + custom metric + CloudWatch Alarm**

> *"Monitor CPU and memory usage of individual processes on an RDS instance."* → **RDS Enhanced Monitoring**

> *"Trigger an alarm when the application logs more than 100 ERROR messages in 5 minutes."* → **CloudWatch Logs Metric Filter + Alarm**

> *"Query application logs to find requests taking more than 2 seconds."* → **CloudWatch Logs Insights**

> *"Process log entries in real time as they are written."* → **CloudWatch Logs Subscription Filter → Lambda/Kinesis**

> *"Show the security group configuration as it existed last Tuesday."* → **AWS Config**

> *"Identify security groups that allow SSH from the Internet."* → **AWS Config Rule**

> *"Automatically fix noncompliant security groups."* → **Config Rule + remediation action / SSM Automation**

> *"A request crosses multiple microservices; identify which service adds the latency."* → **X-Ray**

> *"Identify who downloaded a specific S3 object."* → **CloudTrail S3 Data Events**

> *"Keep API activity records for seven years."* → **CloudTrail Trail → S3**

> *"Prove CloudTrail logs were not tampered with."* → **CloudTrail Log File Integrity Validation**

> *"Detect unusual spikes in AWS API activity."* → **CloudTrail Insights**

> *"Prevent users from disabling a security control across the organization."* → **SCP**, not Config

---

## Pocket card

| Keyword                         | Answer                            |
| ------------------------------- | --------------------------------- |
| Performance / health / metrics  | **CloudWatch**                    |
| Logs                            | **CloudWatch Logs**               |
| Alarm on a metric               | **CloudWatch Alarm**              |
| Count log messages → metric     | **Metric Filter**                 |
| Query logs                      | **Logs Insights**                 |
| Real-time log processing        | **Subscription Filter**           |
| Reduce alert noise              | **Composite Alarm**               |
| EC2 memory / disk-space metrics | **CloudWatch Agent**              |
| RDS process-level CPU/memory    | **Enhanced Monitoring**           |
| Who did what / API audit        | **CloudTrail**                    |
| Long-term API logs              | **CloudTrail Trail → S3**         |
| S3 object-level "who"           | **CloudTrail Data Events**        |
| Unusual API activity            | **CloudTrail Insights**           |
| Prove logs weren't modified     | **Log File Integrity Validation** |
| Configuration history           | **AWS Config**                    |
| Compliance checking             | **AWS Config Rules**              |
| Automatically fix noncompliance | **Config + remediation**          |
| Prevent an action               | **IAM / SCP**                     |
| Trace request across services   | **X-Ray**                         |

## Final memory

```text
CloudWatch
= HOW IS IT PERFORMING?

CloudTrail
= WHO DID WHAT?

AWS Config
= WHAT WAS IT CONFIGURED LIKE?
  IS IT COMPLIANT?

X-Ray
= WHICH PART OF THE REQUEST IS SLOW?
```

### The most important RDS monitoring distinction

```text
Overall RDS CPU
→ CloudWatch

RDS process-level CPU / memory
→ Enhanced Monitoring
```

You now have the monitoring services separated by exactly what the exam is asking you to **measure, audit, inspect, or trace**.
