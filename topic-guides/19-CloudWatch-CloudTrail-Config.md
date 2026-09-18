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

# CloudWatch — "How is it performing?"

**Amazon CloudWatch = monitoring for metrics, logs, alarms, and dashboards.**

It helps you monitor:

* CPU usage
* network traffic
* application logs
* error counts
* latency
* alarms
* dashboards
* application/system metrics

---

## CloudWatch Metrics

Metrics are **numbers measured over time**.

Examples:

```text
CPU = 75%
Request count = 10,000
Latency = 250 ms
```

---

## Important EC2 metric trap

EC2 provides many standard metrics, such as:

* CPU utilization
* network traffic
* status checks
* some EBS-related metrics

But **OS-level metrics** such as:

* memory usage
* swap usage
* filesystem disk usage

are not part of the normal EC2 metric set.

To collect these, install the **CloudWatch Agent**. The agent can collect memory, disk, process, network, and swap metrics.

### Exam pattern

> "Alert when EC2 memory usage exceeds 80%."

→ **CloudWatch Agent + memory metric + CloudWatch Alarm**

---

# EC2 Swap Space Monitoring

This is an important CloudWatch exam trap.

Suppose:

> "Several EC2 instances are failing because they have insufficient swap space. Monitor the available/used swap space."

The correct approach is:

→ **Install the CloudWatch Agent and monitor swap metrics.**

The CloudWatch Agent can collect:

```text
swap_free
swap_used
swap_used_percent
```

`swap_used_percent` represents the percentage of swap space currently being used.

Some exam/question banks may refer to this concept as **`SwapUtilization`**, but the current AWS CloudWatch Agent metric name is **`swap_used_percent`**.

### Example

```text
EC2 instance
      ↓
CloudWatch Agent
      ↓
swap_used_percent
      ↓
CloudWatch
      ↓
Alarm
```

Example:

```text
Swap space = 4 GB
Swap used  = 3 GB

swap_used_percent = 75%
```

### Exam pattern

> "EC2 instances are running out of swap space. Monitor swap utilization."

→ **CloudWatch Agent + swap metric**

---

## Why EC2 Detailed Monitoring is NOT enough

This is a common trap.

**EC2 Detailed Monitoring does not mean more OS-level metrics.**

It mainly changes the frequency of standard EC2 metrics:

```text
Basic monitoring
→ standard EC2 metrics
→ usually 5-minute periods

Detailed monitoring
→ same general EC2 metric categories
→ 1-minute periods
```

AWS documents EC2 detailed monitoring as providing metrics at one-minute intervals rather than the five-minute intervals of basic monitoring.

It does **not** suddenly give you:

* memory usage
* swap usage
* filesystem usage
* process-level metrics

For those, use the **CloudWatch Agent**.

### Remember

```text
Detailed Monitoring
= MORE FREQUENT EC2 METRICS

CloudWatch Agent
= MORE OS-LEVEL METRICS
```

---

## EC2 Monitoring — the important distinction

| Requirement                      | Solution                        |
| -------------------------------- | ------------------------------- |
| Standard EC2 CPU/network metrics | **CloudWatch**                  |
| Standard metrics every 1 minute  | **EC2 Detailed Monitoring**     |
| EC2 memory usage                 | **CloudWatch Agent**            |
| EC2 swap usage                   | **CloudWatch Agent**            |
| EC2 filesystem disk usage        | **CloudWatch Agent**            |
| EC2 process-level metrics        | **CloudWatch Agent / procstat** |
| Alarm on any collected metric    | **CloudWatch Alarm**            |

### Memory trick

```text
Detailed Monitoring
= frequency

CloudWatch Agent
= visibility inside the OS
```

---

## Auto Scaling + CloudWatch Agent

If the question says:

> "Instances are launched dynamically by an Auto Scaling Group. Monitor memory/swap/disk on every instance."

You still need the **CloudWatch Agent**.

For an Auto Scaling fleet, a common architecture is:

```text
Launch Template / AMI / User Data / Systems Manager
                    ↓
             CloudWatch Agent
                    ↓
              EC2 instances
                    ↓
             CloudWatch metrics
                    ↓
                 Alarms
```

The important exam point is:

> **Auto Scaling does not automatically install the CloudWatch Agent.**

You must configure the instances to have the agent.

---

# CloudWatch Enhanced Monitoring vs normal CloudWatch

For **RDS**, there is an important distinction.

## Normal CloudWatch RDS monitoring

CloudWatch can show overall database-instance metrics.

Example:

```text
RDS CPUUtilization = 75%
```

This tells you about the **RDS instance as a whole**.

---

## RDS Enhanced Monitoring

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

> "Monitor the CPU and memory used by individual processes on an RDS instance."

→ **RDS Enhanced Monitoring**

### Easy distinction

```text
Overall RDS CPU
→ CloudWatch

RDS OS-level CPU / memory / processes
→ Enhanced Monitoring
```

---

# RDS Performance Insights

Do not confuse **Enhanced Monitoring** with **Performance Insights**.

### Performance Insights

Performance Insights is focused on **database performance and query/load analysis**.

Think:

```text
Which SQL queries
are consuming database resources?
```

### Enhanced Monitoring

Enhanced Monitoring is focused on the **underlying OS**.

Think:

```text
How much CPU / memory
are the database processes using?
```

### Exam pattern

> "Identify which SQL queries are causing high database load."

→ **Performance Insights**

> "Monitor OS-level CPU, memory, and process information."

→ **Enhanced Monitoring**

### Memory trick

```text
Enhanced Monitoring
= OS-level RDS monitoring

Performance Insights
= database workload / query performance
```

---

# CloudWatch Alarms

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

---

## Composite alarms

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

> "Several alarms are firing and the company wants fewer unnecessary notifications."

→ **Composite Alarm**

---

# CloudWatch Logs

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

---

## Metric Filters

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

> "Trigger an alarm when the application writes more than 100 ERROR messages in 5 minutes."

→ **CloudWatch Logs Metric Filter + Alarm**

---

# CloudWatch Logs Insights

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
= query / analyze logs
```

---

# CloudWatch Logs Subscription Filters

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

> "Process log entries in real time as they arrive."

→ **CloudWatch Logs Subscription Filter**

---

# CloudWatch Logs → S3 / Firehose

Another common pattern is:

```text
CloudWatch Logs
      ↓
Subscription Filter
      ↓
Kinesis Data Firehose
      ↓
S3
```

Think of this when the requirement is to:

* continuously export logs
* centralize logs
* archive logs
* process logs before storing them

---

# CloudWatch Dashboards

CloudWatch dashboards display metrics and monitoring information in one place.

They can show resources and metrics from different Regions.

Example:

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

# CloudWatch — High-Value Exam Traps

### EC2 Memory

> "Monitor EC2 memory usage."

→ **CloudWatch Agent**

Not:

→ EC2 Detailed Monitoring

---

### EC2 Swap

> "Monitor EC2 swap utilization."

→ **CloudWatch Agent**

Current AWS Agent metrics include `swap_used_percent`, `swap_used`, and `swap_free`.

---

### EC2 Disk Space

> "Alert when the EC2 filesystem is 90% full."

→ **CloudWatch Agent**

Do not confuse:

```text
EBS volume metrics
```

with:

```text
filesystem space inside the operating system
```

The CloudWatch Agent can collect filesystem metrics such as `disk_used_percent`.

---

### EC2 Detailed Monitoring

> "The company wants EC2 metrics every minute instead of every five minutes."

→ **Detailed Monitoring**

It changes the **frequency**, not the type of OS metrics collected.

---

### Process-level EC2 metrics

> "Monitor CPU and memory usage for a specific process running on EC2."

→ **CloudWatch Agent / procstat**

The CloudWatch Agent can collect process-specific CPU and memory metrics.

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

# CloudTrail Event History

CloudTrail provides **90 days of management event history** without requiring you to create a trail.

For longer-term retention:

```text
CloudTrail Trail
      ↓
S3
```

Store the logs in S3 for long-term retention.

### Exam pattern

> "Keep API activity records for several years."

→ **CloudTrail Trail → S3**

---

# CloudTrail Management Events vs Data Events

This is very important.

## Management events

These are **control-plane operations**.

Examples:

* create EC2 instance
* terminate EC2 instance
* create S3 bucket
* change IAM policy

---

## Data events

These are **resource-level operations**.

Examples:

* reading an S3 object
* deleting an S3 object
* invoking a Lambda function

Data events generally must be **enabled explicitly** and can incur additional charges.

### Exam trap

> "Who deleted a specific object from S3?"

→ **CloudTrail S3 Data Events**

Normal management events are not enough.

---

# CloudTrail Log File Integrity Validation

This helps prove that CloudTrail log files have **not been modified after delivery**.

Useful for:

* security investigations
* compliance
* forensics
* proving log integrity

### Exam pattern

> "Logs must be tamper-evident for forensic purposes."

→ **CloudTrail Log File Integrity Validation**

---

# CloudTrail Insights

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

# Configuration history

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

> "Show what a security group's configuration was one week ago."

→ **AWS Config**

---

# Config Rules

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

> "Identify security groups that allow SSH from the Internet."

→ **AWS Config Rule**

---

# Config Remediation

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

# Important Config limitation

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

> "A request passes through ten microservices. Find which service is causing the latency."

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

# Similar Exam Questions

## 1. EC2 memory monitoring

> "An application running on EC2 occasionally runs out of memory. The architect needs to monitor memory utilization."

→ **CloudWatch Agent**

```text
EC2
 ↓
CloudWatch Agent
 ↓
Memory metric
 ↓
CloudWatch Alarm
```

---

## 2. EC2 swap monitoring

> "Several EC2 instances are failing because of insufficient swap space. Monitor swap utilization."

→ **CloudWatch Agent**

Current agent metric:

```text
swap_used_percent
```

(Some question banks may call this `SwapUtilization`.)

---

## 3. EC2 disk-space monitoring

> "Alert when `/var` reaches 90% disk utilization."

→ **CloudWatch Agent**

Because this is **filesystem-level information inside the OS**.

---

## 4. EC2 metric frequency

> "The company needs EC2 metrics every minute rather than every five minutes."

→ **EC2 Detailed Monitoring**

```text
Basic
→ 5 minutes

Detailed
→ 1 minute
```

---

## 5. EC2 process monitoring

> "Find how much CPU and memory a specific process is consuming."

→ **CloudWatch Agent with process/procstat monitoring**

---

## 6. RDS overall CPU

> "Monitor the CPU utilization of an RDS instance."

→ **CloudWatch**

---

## 7. RDS process-level CPU/memory

> "Monitor CPU and memory usage of individual processes on an RDS instance."

→ **RDS Enhanced Monitoring**

---

## 8. RDS query performance

> "Determine which SQL statements are responsible for database load."

→ **RDS Performance Insights**

---

## 9. Log message alarm

> "Send an alert if the application generates more than 100 ERROR entries in five minutes."

→ **CloudWatch Logs Metric Filter + CloudWatch Alarm**

---

## 10. Search logs

> "Find requests that took more than two seconds."

→ **CloudWatch Logs Insights**

---

## 11. Real-time log processing

> "Send application logs to Lambda for real-time processing."

→ **CloudWatch Logs Subscription Filter → Lambda**

---

## 12. Identify who performed an API action

> "Who terminated the EC2 instance?"

→ **CloudTrail**

---

## 13. Identify who deleted an S3 object

> "Who deleted this specific object from an S3 bucket?"

→ **CloudTrail S3 Data Events**

---

## 14. Long-term API auditing

> "Keep API activity records for seven years."

→ **CloudTrail Trail → S3**

---

## 15. Unusual API activity

> "Detect unusual spikes in API activity."

→ **CloudTrail Insights**

---

## 16. Prove logs were not modified

> "Provide evidence that CloudTrail logs have not been tampered with."

→ **CloudTrail Log File Integrity Validation**

---

## 17. Historical configuration

> "What did this security group look like last Tuesday?"

→ **AWS Config**

---

## 18. Compliance

> "Identify security groups that allow SSH from the Internet."

→ **AWS Config Rule**

---

## 19. Automatic compliance remediation

> "Automatically fix resources that violate the security rule."

→ **AWS Config Rule + remediation**

Often:

```text
Config
 ↓
Noncompliant
 ↓
SSM Automation
 ↓
Fix
```

---

## 20. Prevent the action

> "Prevent developers from disabling CloudTrail."

→ **IAM / SCP / preventive control**

Not:

→ AWS Config

---

## 21. Distributed application latency

> "A request passes through API Gateway, Lambda, several microservices, and DynamoDB. Find which component is causing the delay."

→ **X-Ray**

---

# Very Common Monitoring Traps

## Trap 1 — "Detailed Monitoring" sounds like detailed OS monitoring

It isn't.

```text
EC2 Detailed Monitoring
= more frequent standard EC2 metrics

CloudWatch Agent
= OS-level metrics
```

---

## Trap 2 — CloudWatch vs CloudWatch Agent

Both are CloudWatch-related, but remember:

```text
CloudWatch
= monitoring platform

CloudWatch Agent
= software running on the server
  that collects additional OS metrics
```

---

## Trap 3 — CloudWatch Agent vs RDS Enhanced Monitoring

```text
EC2 memory/swap/disk
→ CloudWatch Agent

RDS OS-level process/memory monitoring
→ Enhanced Monitoring
```

---

## Trap 4 — Enhanced Monitoring vs Performance Insights

```text
OS/process information
→ Enhanced Monitoring

Database/query/load information
→ Performance Insights
```

---

## Trap 5 — CloudWatch Logs vs CloudTrail

```text
Application logs
→ CloudWatch Logs

AWS API activity
→ CloudTrail
```

Example:

```text
"Application returned HTTP 500"
→ CloudWatch Logs

"Who deleted the EC2 instance?"
→ CloudTrail
```

---

## Trap 6 — CloudTrail vs Config

```text
Who changed the resource?
→ CloudTrail

What did the resource look like?
→ Config
```

Example:

```text
Who changed the security group?
→ CloudTrail

What was the security group's configuration yesterday?
→ Config
```

---

## Trap 7 — Config vs IAM/SCP

```text
Detect noncompliance
→ Config

Prevent unauthorized action
→ IAM / SCP
```

---

## Trap 8 — CloudWatch vs X-Ray

```text
Metric shows high latency
→ CloudWatch

Find which service/hop caused the latency
→ X-Ray
```

---

# Pocket Card

| Keyword                         | Answer                            |
| ------------------------------- | --------------------------------- |
| Performance / health / metrics  | **CloudWatch**                    |
| Logs                            | **CloudWatch Logs**               |
| Alarm on a metric               | **CloudWatch Alarm**              |
| Count log messages → metric     | **Metric Filter**                 |
| Query logs                      | **Logs Insights**                 |
| Real-time log processing        | **Subscription Filter**           |
| Reduce alert noise              | **Composite Alarm**               |
| EC2 memory                      | **CloudWatch Agent**              |
| EC2 swap                        | **CloudWatch Agent**              |
| EC2 filesystem disk usage       | **CloudWatch Agent**              |
| EC2 process metrics             | **CloudWatch Agent / procstat**   |
| EC2 metrics every 1 minute      | **Detailed Monitoring**           |
| RDS process-level CPU/memory    | **Enhanced Monitoring**           |
| RDS query/database load         | **Performance Insights**          |
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

---

# Final Memory

```text
CloudWatch
= HOW IS IT PERFORMING?

CloudWatch Agent
= WHAT IS HAPPENING INSIDE THE OS?

CloudWatch Detailed Monitoring
= GIVE ME STANDARD EC2 METRICS MORE FREQUENTLY

CloudTrail
= WHO DID WHAT?

AWS Config
= WHAT WAS IT CONFIGURED LIKE?
  IS IT COMPLIANT?

X-Ray
= WHICH PART OF THE REQUEST IS SLOW?
```

## The most important monitoring distinctions

```text
EC2 overall CPU
→ CloudWatch

EC2 memory / swap / filesystem
→ CloudWatch Agent

EC2 standard metrics every 1 minute
→ Detailed Monitoring

RDS overall CPU
→ CloudWatch

RDS OS / process CPU and memory
→ Enhanced Monitoring

RDS database/query workload
→ Performance Insights

Application logs
→ CloudWatch Logs

AWS API actions
→ CloudTrail

Resource configuration history / compliance
→ AWS Config

Distributed request tracing
→ X-Ray
```
