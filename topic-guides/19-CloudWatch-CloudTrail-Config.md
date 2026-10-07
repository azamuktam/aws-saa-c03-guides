# Section 19: CloudWatch, CloudTrail & AWS Config

## The idea

These services monitor AWS but answer different questions:

| Service        | Main question                                                   |
| -------------- | --------------------------------------------------------------- |
| **CloudWatch** | **How is it performing?**                                       |
| **CloudTrail** | **Who did what?**                                               |
| **AWS Config** | **What was the resource configured like, and is it compliant?** |
| **X-Ray**      | **Which part of the application request is slow?**              |

```text
Performance / metrics / logs / alarms → CloudWatch
Who created / deleted / changed something? → CloudTrail
What did the configuration look like? Is it compliant? → AWS Config
Which service/hop in a request chain is slow? → X-Ray
```

---

# CloudWatch — "How is it performing?"

**Amazon CloudWatch = monitoring for metrics, logs, alarms, and dashboards.**

Monitors include CPU, network traffic, application logs, error counts, latency, alarms, dashboards, and application/system metrics.

## CloudWatch Metrics

Metrics are **numbers measured over time**.

```text
CPU = 75%
Request count = 10,000
Latency = 250 ms
```

## Important EC2 metric trap

Standard EC2 metrics include CPU utilization, network traffic, status checks, and some EBS-related metrics.

Normal EC2 metrics do **not** include OS-level:

* memory usage
* swap usage
* filesystem disk usage

Use the **CloudWatch Agent** for memory, disk, process, network, and swap metrics.

### Exam pattern

> "Alert when EC2 memory usage exceeds 80%."

→ **CloudWatch Agent + memory metric + CloudWatch Alarm**

---

# EC2 Swap Space Monitoring

For swap monitoring:

→ **CloudWatch Agent**

The agent can collect:

```text
swap_free
swap_used
swap_used_percent
```

`swap_used_percent` = percentage of swap currently used.

Some question banks may call this **`SwapUtilization`**, but the current AWS CloudWatch Agent metric name is **`swap_used_percent`**.

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

**EC2 Detailed Monitoring does not mean more OS-level metrics.** It mainly changes the frequency of standard EC2 metrics:

```text
Basic monitoring
→ standard EC2 metrics
→ usually 5-minute periods

Detailed monitoring
→ same general EC2 metric categories
→ 1-minute periods
```

It does **not** add memory, swap, filesystem, or process-level metrics.

For those → **CloudWatch Agent**.

```text
Detailed Monitoring = MORE FREQUENT EC2 METRICS
CloudWatch Agent   = MORE OS-LEVEL METRICS
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

```text
Detailed Monitoring = frequency
CloudWatch Agent    = visibility inside the OS
```

## Auto Scaling + CloudWatch Agent

If instances are launched dynamically by an Auto Scaling Group and you need memory/swap/disk on every instance, you still need the **CloudWatch Agent**.

Common architecture:

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

**Exam point:** Auto Scaling does **not** automatically install the CloudWatch Agent. Configure instances to have the agent.

---

# CloudWatch Container Insights

**CloudWatch Container Insights = container-focused monitoring for EKS, ECS, and other container workloads.**

It automatically collects useful **container, pod, node, and application metrics/logs** and sends them to CloudWatch for monitoring and dashboards.

```text
EKS / ECS
   ↓
CloudWatch Container Insights
   ↓
Metrics + Logs
   ↓
CloudWatch dashboards / alarms
```

### Exam pattern

> **"EKS/ECS + application/container logs and metrics + centralized monitoring + least operational overhead."**

→ **CloudWatch Container Insights**

### Container Insights vs CloudWatch Agent

```text
CloudWatch Agent
→ general-purpose agent
→ EC2 / on-prem / host-level metrics and logs

Container Insights
→ container-focused monitoring
→ EKS / ECS
→ container / pod / node metrics + logs
```

### Exam shortcut

> **Containerized workload → think Container Insights**

> **EC2/on-prem OS metrics such as memory, swap, filesystem → think CloudWatch Agent**

---

# CloudWatch Enhanced Monitoring vs normal CloudWatch

For **RDS**, distinguish normal CloudWatch monitoring from Enhanced Monitoring.

## Normal CloudWatch RDS monitoring

CloudWatch provides overall database-instance metrics, e.g.:

```text
RDS CPUUtilization = 75%
```

This is about the **RDS instance as a whole**.

## RDS Enhanced Monitoring

**Enhanced Monitoring = OS-level monitoring of the RDS instance.**

Includes:

* CPU usage
* memory usage
* processes
* process-level CPU
* process-level memory
* load
* filesystem information

Example:

```text
RDS instance
├── Process A → CPU 20%, memory 300 MB
├── Process B → CPU 35%, memory 500 MB
└── Process C → CPU 10%, memory 100 MB
```

### Exam pattern

> "Monitor the CPU and memory used by individual processes on an RDS instance."

→ **RDS Enhanced Monitoring**

```text
Overall RDS CPU
→ CloudWatch

RDS OS-level CPU / memory / processes
→ Enhanced Monitoring
```

---

# RDS Performance Insights

Do not confuse **Enhanced Monitoring** with **Performance Insights**.

**Performance Insights** focuses on **database performance and query/load analysis**:

```text
Which SQL queries are consuming database resources?
```

**Enhanced Monitoring** focuses on the **underlying OS**:

```text
How much CPU / memory are the database processes using?
```

### Exam patterns

> "Identify which SQL queries are causing high database load."

→ **Performance Insights**

> "Monitor OS-level CPU, memory, and process information."

→ **Enhanced Monitoring**

```text
Enhanced Monitoring = OS-level RDS monitoring
Performance Insights = database workload / query performance
```

---

# CloudWatch Alarms

A **CloudWatch Alarm** watches a metric and acts when a condition is met.

```text
CPU > 80%
    ↓
CloudWatch Alarm
    ↓
SNS notification
```

Can also trigger:

* Auto Scaling
* EC2 actions
* SNS notifications

## Composite alarms

A **Composite Alarm** combines multiple alarms.

```text
CPU > 80%
AND
Error rate > 5%
        ↓
Composite Alarm
```

Use it to reduce unnecessary alerts.

### Exam pattern

> "Several alarms are firing and the company wants fewer unnecessary notifications."

→ **Composite Alarm**

---

# CloudWatch Logs

CloudWatch Logs stores application and system logs.

```text
Log Group
├── Log Stream
├── Log Stream
└── Log Stream
```

* **Log group**: usually an application or service.
* **Log stream**: logs from one source, such as an instance or container.

## Metric Filters

A **metric filter** searches logs for a pattern and turns the result into a metric.

```text
Application log:
ERROR
ERROR
INFO
ERROR

Metric filter:
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

**Logs Insights = interactive query and analysis of CloudWatch Logs.**

Use it to search/analyze large amounts of logs.

> "Find all requests that took more than 2 seconds."

→ **CloudWatch Logs Insights**

```text
Metric Filter = turn log patterns into metrics
Logs Insights = query / analyze logs
```

---

# CloudWatch Logs Subscription Filters

A **subscription filter** sends log events somewhere else in **near real time**.

Common destinations:

* Kinesis Data Streams
* Kinesis Data Firehose
* Lambda

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

Common pattern:

```text
CloudWatch Logs
      ↓
Subscription Filter
      ↓
Kinesis Data Firehose
      ↓
S3
```

Think of this for:

* continuously exporting logs
* centralizing logs
* archiving logs
* processing logs before storage

---

# ALB Access Logs — detailed HTTP request logging

**Application Load Balancer Access Logs = detailed records of HTTP/HTTPS requests processed by the ALB.**

Different from CloudWatch metrics: access logs contain **individual request records**, not only aggregated numbers.

Can contain:

* client IP
* request path
* request method
* HTTP status codes
* bytes sent/received
* request processing time
* target processing time
* response processing time
* user agent and other request metadata

Published to **Amazon S3 periodically, typically every 5 minutes**.

### Exam pattern

> "Capture detailed information about every HTTP request through an ALB, including the client IP address and latency."

→ **ALB Access Logs**

## ALB Access Logs vs CloudWatch Metrics

```text
CloudWatch Metrics = aggregated numbers
RequestCount
TargetResponseTime
HTTPCode_ELB_5XX_Count
```

vs.

```text
ALB Access Logs = individual request records
Client IP
Request path
Status code
Request / target / response latency
```

Examples:

> "How many requests reached the ALB?"

→ **CloudWatch metric: RequestCount**

> "Show me the client IP and details of each request."

→ **ALB Access Logs**

## ALB Access Logs vs CloudTrail

```text
CloudTrail
= control-plane / API activity
"Who modified the ALB?"

ALB Access Logs
= data-plane HTTP traffic
"What HTTP request passed through the ALB?"
```

### Exam pattern

> "The company wants detailed information about all HTTP requests that passed through the public-facing ALB."

→ **ALB Access Logs**, not CloudTrail.

## ALB Access Logs vs X-Ray

```text
ALB Access Logs
= detailed request records at the load balancer

X-Ray
= distributed tracing across application components
```

Use access logs for client IP, path, HTTP status, and latency fields.

Use X-Ray when the goal is:

```text
Which application component in the request chain caused the delay?
```

## ALB Access Logs vs CloudWatch Logs

```text
ALB Access Logs
→ records created by the load balancer
→ commonly stored in S3
→ request-level ALB traffic information

CloudWatch Logs
→ application/system/container logs
→ stored in CloudWatch Logs
→ queried with Logs Insights
```

### Exam clues

> "Capture detailed information about requests handled by the ALB."

→ **ALB Access Logs**

> "Search application log messages."

→ **CloudWatch Logs / Logs Insights**

---

# CloudWatch Application Insights

**CloudWatch Application Insights** helps automatically discover and monitor application components and supporting AWS resources and helps troubleshoot application problems.

Useful for:

* application/resource monitoring
* relevant metrics
* logs
* dashboards
* detecting problems
* troubleshooting application issues

For containerized applications, it can be used for application monitoring and troubleshooting.

**Application Insights does not replace ALB access logs.**

```text
ALB Access Logs
= "What requests passed through the ALB?"

Application Insights
= "What is happening with the application and its supporting resources?"
```

A question can require both.

### Example architecture

```text
Client
   ↓
ALB
   ├── ALB Access Logs → S3
   ↓
Application / ECS workload
   ↓
CloudWatch Application Insights
   ├── Monitoring
   ├── Logs
   ├── Metrics
   └── Troubleshooting
```

### Exam pattern

> "Capture detailed ALB HTTP requests for traffic analysis and also simplify troubleshooting of the containerized application."

→ **ALB Access Logs + CloudWatch Application Insights**

---

# CloudWatch Dashboards

CloudWatch dashboards display metrics and monitoring information in one place.

They can show resources and metrics from **different Regions**.

```text
US      → CPU / Errors / Latency
Europe  → CPU / Errors / Latency
Asia    → CPU / Errors / Latency
                ↓
       One CloudWatch dashboard
```

---

# CloudWatch — High-Value Exam Traps

### EC2 Memory

> "Monitor EC2 memory usage."

→ **CloudWatch Agent**

Not → EC2 Detailed Monitoring.

### EC2 Swap

> "Monitor EC2 swap utilization."

→ **CloudWatch Agent**

Current AWS Agent metrics: `swap_used_percent`, `swap_used`, `swap_free`.

(Some question banks may call this `SwapUtilization`.)

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

CloudWatch Agent can collect filesystem metrics such as `disk_used_percent`.

### EC2 Detailed Monitoring

> "The company wants EC2 metrics every minute instead of every five minutes."

→ **Detailed Monitoring**

Changes **frequency**, not OS metric type.

### Process-level EC2 metrics

> "Monitor CPU and memory usage for a specific process running on EC2."

→ **CloudWatch Agent / procstat**

### ALB request details

> "Capture every HTTP request, including client IP and latency."

→ **ALB Access Logs**

Not → CloudWatch `GetMetricData`
Not → CloudTrail
Not necessarily → X-Ray

### ALB health

> "Monitor whether the load balancer/targets are healthy."

→ **ELB health checks / CloudWatch health-related metrics**

Do not confuse:

```text
Health checks = "Is the target healthy?"
Access logs  = "What requests went through the ALB?"
```

---

# CloudTrail — "Who did what?"

**AWS CloudTrail = audit log of AWS API activity.**

Records:

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

CloudTrail provides **90 days of management event history** without requiring a trail.

For longer-term retention:

```text
CloudTrail Trail
      ↓
     S3
```

Store logs in S3 for long-term retention.

### Exam pattern

> "Keep API activity records for several years."

→ **CloudTrail Trail → S3**

---

# CloudTrail Logs + S3 Encryption

CloudTrail log files delivered to S3 are **encrypted at rest by default with S3 server-side encryption (SSE-S3)**. You can optionally use **SSE-KMS** when you need control over the KMS key and its permissions.

Separately, **Amazon S3 automatically encrypts all new object uploads with SSE-S3 (AES-256) by default**. This has been the default for new S3 objects since **January 5, 2023**.

```text
New S3 object
→ automatically encrypted
→ SSE-S3
→ AES-256
```

Enabling bucket default encryption does **not** automatically re-encrypt old unencrypted objects.

### Exam trap

Older practice questions may say:

> "Configure the S3 bucket to use Server-Side Encryption."

Current behavior:

```text
S3 new objects
→ SSE-S3 by default

CloudTrail → S3
→ encrypted at rest by default

Need customer-controlled encryption key
→ SSE-KMS
```

### SSE-S3 vs SSE-KMS

```text
SSE-S3
= AWS manages the S3 encryption keys
= default S3 encryption

SSE-KMS
= AWS KMS manages the key
= more control over key policies / permissions
```

### Encryption exam clues

> "CloudTrail logs must be encrypted."

→ **CloudTrail → S3 with server-side encryption**

> "Need customer-controlled KMS key for CloudTrail logs."

→ **CloudTrail + SSE-KMS**

> "Does S3 automatically encrypt new objects?"

→ **Yes, SSE-S3**

> "Are existing unencrypted objects automatically changed?"

→ **No**

---

# CloudTrail Management Events vs Data Events

## Management events

**Control-plane operations**, e.g.:

* create EC2 instance
* terminate EC2 instance
* create S3 bucket
* change IAM policy

## Data events

**Resource-level operations**, e.g.:

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

Helps prove CloudTrail log files were **not modified after delivery**.

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

```text
Normally:
TerminateInstances = 2 per day

Suddenly:
TerminateInstances = 500 in 10 minutes
```

CloudTrail Insights can identify the unusual behavior.

```text
CloudTrail = API audit
CloudTrail Insights = unusual API activity
```

---

# AWS Config — "What was the configuration, and is it compliant?"

**AWS Config = resource configuration history + compliance checking.**

Answers:

> "What did this security group look like last Tuesday?"

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

You can investigate configuration history.

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
* custom

### Exam pattern

> "Identify security groups that allow SSH from the Internet."

→ **AWS Config Rule**

---

# IAM Access Key Rotation

AWS Config has a managed rule:

**`access-keys-rotated`**

It checks whether IAM user access keys have been rotated within the configured maximum age.

```text
maxAccessKeyAge = 90 days
        ↓
Access key > 90 days
        ↓
NON_COMPLIANT
```

### Exam pattern

> "Identify IAM user access keys that have not been rotated for more than 90 days."

→ **AWS Config managed rule `access-keys-rotated`**

### Automatic remediation

> "Automatically deactivate and delete IAM user access keys that are more than 90 days old."

→ **AWS Config `access-keys-rotated` → EventBridge → Lambda**

```text
Config
  ↓
NON_COMPLIANT
  ↓
EventBridge
  ↓
Lambda
  ↓
Deactivate + delete key
```

**Exam trap:** EventBridge does not itself determine that an IAM access key is older than 90 days. Config performs the compliance evaluation first.

---

# Config Remediation

AWS Config can automatically trigger remediation when a resource becomes noncompliant.

Common pattern:

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

**AWS Config does not prevent the action from happening.** It detects configuration and compliance problems.

> **"Prevent users from creating this resource or performing this action."**

Think:

* IAM
* SCPs
* other preventive controls

Not Config.

```text
Config
= detect / record / evaluate

SCP / IAM
= prevent / control permissions
```

---

# X-Ray — "Which part of the request is slow?"

**AWS X-Ray = distributed tracing.**

Follows one request across multiple services.

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

If the request takes 2 seconds:

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

```text
CloudWatch = this service is slow
X-Ray       = this specific hop in the request is slow
```

---

# The four-way comparison

| Service        | Main question                                | Example                     |
| -------------- | -------------------------------------------- | --------------------------- |
| **CloudWatch** | How is it performing?                        | CPU = 90%                   |
| **CloudTrail** | Who did what?                                | Alice terminated EC2        |
| **AWS Config** | What was the configuration? Is it compliant? | SG allowed SSH last Tuesday |
| **X-Ray**      | Which part of the request is slow?           | Service B adds 1.6 seconds  |

```text
CloudWatch = PERFORMANCE
CloudTrail = API AUDIT
Config     = CONFIGURATION + COMPLIANCE
X-Ray      = REQUEST TRACE
```

---

# ALB Monitoring — the five-way distinction

| Requirement                                  | Service                             |
| -------------------------------------------- | ----------------------------------- |
| Number of requests / aggregated ALB behavior | **CloudWatch Metrics**              |
| Detailed individual HTTP requests            | **ALB Access Logs**                 |
| Who changed the ALB configuration/API        | **CloudTrail**                      |
| Which application hop is slow                | **X-Ray**                           |
| Application/resource troubleshooting         | **CloudWatch Application Insights** |
| Target/load balancer health                  | **ELB Health Checks**               |

```text
ALB
├── CloudWatch Metrics
│   └── RequestCount / latency / errors
├── Access Logs
│   └── client IP / request / status / latency details
├── CloudTrail
│   └── API/control-plane changes
├── Health Checks
│   └── target health
└── X-Ray
    └── distributed request tracing
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

## 2. EC2 swap monitoring

> "Several EC2 instances are failing because of insufficient swap space. Monitor swap utilization."

→ **CloudWatch Agent**

Current agent metric:

```text
swap_used_percent
```

(Some question banks may call this `SwapUtilization`.)

## 3. EC2 disk-space monitoring

> "Alert when `/var` reaches 90% disk utilization."

→ **CloudWatch Agent**

Because this is **filesystem-level information inside the OS**.

## 4. EC2 metric frequency

> "The company needs EC2 metrics every minute rather than every five minutes."

→ **EC2 Detailed Monitoring**

```text
Basic    → 5 minutes
Detailed → 1 minute
```

## 5. EC2 process monitoring

> "Find how much CPU and memory a specific process is consuming."

→ **CloudWatch Agent with process/procstat monitoring**

## 6. RDS overall CPU

> "Monitor the CPU utilization of an RDS instance."

→ **CloudWatch**

## 7. RDS process-level CPU/memory

> "Monitor CPU and memory usage of individual processes on an RDS instance."

→ **RDS Enhanced Monitoring**

## 8. RDS query performance

> "Determine which SQL statements are responsible for database load."

→ **RDS Performance Insights**

## 9. Log message alarm

> "Send an alert if the application generates more than 100 ERROR entries in five minutes."

→ **CloudWatch Logs Metric Filter + CloudWatch Alarm**

## 10. Search logs

> "Find requests that took more than two seconds."

→ **CloudWatch Logs Insights**

## 11. Real-time log processing

> "Send application logs to Lambda for real-time processing."

→ **CloudWatch Logs Subscription Filter → Lambda**

## 12. Identify who performed an API action

> "Who terminated the EC2 instance?"

→ **CloudTrail**

## 13. Identify who deleted an S3 object

> "Who deleted this specific object from an S3 bucket?"

→ **CloudTrail S3 Data Events**

## 14. Long-term API auditing

> "Keep API activity records for seven years."

→ **CloudTrail Trail → S3**

## 15. CloudTrail log encryption

> "CloudTrail logs stored in S3 must be encrypted."

→ **CloudTrail → S3 with server-side encryption**

Current behavior:

```text
CloudTrail → S3
→ encrypted at rest by default

S3 new objects
→ SSE-S3 by default
```

Need a customer-controlled KMS key:

```text
CloudTrail
→ SSE-KMS
```

## 16. S3 default encryption

> "Are newly uploaded S3 objects encrypted by default?"

→ **Yes, SSE-S3 (AES-256).**

> "Will enabling default encryption encrypt old unencrypted objects?"

→ **No.**

## 17. Unusual API activity

> "Detect unusual spikes in API activity."

→ **CloudTrail Insights**

## 18. Prove logs were not modified

> "Provide evidence that CloudTrail logs have not been tampered with."

→ **CloudTrail Log File Integrity Validation**

## 19. Historical configuration

> "What did this security group look like last Tuesday?"

→ **AWS Config**

## 20. Compliance

> "Identify security groups that allow SSH from the Internet."

→ **AWS Config Rule**

## 21. Automatic compliance remediation

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

## 22. IAM access key rotation

> "A company wants to identify IAM user access keys that are more than 90 days old."

→ **AWS Config managed rule `access-keys-rotated`**

```text
maxAccessKeyAge = 90 days
```

## 23. IAM access key automatic cleanup

> "A company wants to automatically deactivate and delete any IAM user access key that is more than 90 days old with the least operational effort."

→ **AWS Config `access-keys-rotated` → EventBridge → Lambda**

```text
IAM access key
      ↓
AWS Config
access-keys-rotated
      ↓
> 90 days
      ↓
NON_COMPLIANT
      ↓
EventBridge
      ↓
Lambda
      ↓
Deactivate + delete
```

### Why not EventBridge directly?

> "Create an EventBridge rule to filter IAM access keys older than 90 days."

→ **Not the intended solution**

Config performs the access-key age/compliance evaluation first.

## 24. Prevent the action

> "Prevent developers from disabling CloudTrail."

→ **IAM / SCP / preventive control**

Not → **AWS Config**

## 25. Distributed application latency

> "A request passes through API Gateway, Lambda, several microservices, and DynamoDB. Find which component is causing the delay."

→ **X-Ray**

## 26. ALB client IP and request details

> "Capture detailed information about every HTTP request through an ALB, including client IP addresses and latency."

→ **ALB Access Logs**

## 27. ALB traffic patterns

> "Analyze detailed traffic patterns from requests passing through the Application Load Balancer."

→ **ALB Access Logs**

If the question also asks for application troubleshooting:

→ **ALB Access Logs + CloudWatch Application Insights**

## 28. ALB API changes

> "Find out who modified the ALB listener or configuration."

→ **CloudTrail**

Not → **ALB Access Logs**

## 29. ALB aggregate request count

> "Monitor the number of requests received by the ALB over time."

→ **CloudWatch Metrics**

Not → **ALB Access Logs** if only an aggregate metric is required.

## 30. ALB request path and client IP

> "The company needs the source IP address and detailed information about individual requests."

→ **ALB Access Logs**

## 31. Find the slow application component

> "The ALB shows high latency and the company needs to determine which downstream service is responsible."

→ **X-Ray**

The ALB may show overall latency, but **X-Ray traces the request across application components**.

## 32. Monitor application troubleshooting

> "The company wants automated application monitoring and troubleshooting for its application and supporting AWS resources."

→ **CloudWatch Application Insights**

---

# Very Common Monitoring Traps

## Trap 1 — "Detailed Monitoring" sounds like detailed OS monitoring

It isn't.

```text
Detailed Monitoring = frequency
CloudWatch Agent    = visibility inside the OS
```

## Trap 2 — CloudWatch vs CloudWatch Agent

```text
CloudWatch
= monitoring platform

CloudWatch Agent
= software running on the server
  that collects additional OS metrics
```

## Trap 3 — CloudWatch Agent vs RDS Enhanced Monitoring

```text
EC2 memory/swap/disk
→ CloudWatch Agent

RDS OS-level process/memory monitoring
→ Enhanced Monitoring
```

## Trap 4 — Enhanced Monitoring vs Performance Insights

```text
OS/process information
→ Enhanced Monitoring

Database/query/load information
→ Performance Insights
```

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

## Trap 6 — CloudTrail vs Config

```text
Who changed the resource?
→ CloudTrail

What was the resource configuration?
→ Config
```

Example:

```text
Who changed the security group?
→ CloudTrail

What was the security group's configuration yesterday?
→ Config
```

## Trap 7 — Config vs IAM/SCP

```text
Detect noncompliance
→ Config

Prevent unauthorized action
→ IAM / SCP
```

## Trap 8 — Config vs EventBridge

```text
Evaluate IAM key age / compliance
→ Config

React to the compliance change
→ EventBridge

Deactivate/delete the IAM key
→ Lambda
```

## Trap 9 — CloudWatch vs X-Ray

```text
Metric shows high latency
→ CloudWatch

Find which service/hop caused the latency
→ X-Ray
```

## Trap 10 — S3 encryption

```text
New S3 object
→ SSE-S3 automatically

Need customer-controlled key
→ SSE-KMS

Old unencrypted S3 objects
→ not automatically re-encrypted
```

## Trap 11 — ALB Access Logs vs CloudTrail

```text
HTTP requests through ALB
→ ALB Access Logs

AWS API calls involving the ALB
→ CloudTrail
```

## Trap 12 — ALB Access Logs vs CloudWatch Metrics

```text
Individual request details
→ ALB Access Logs

Aggregated request/latency metrics
→ CloudWatch Metrics
```

## Trap 13 — ALB Access Logs vs X-Ray

```text
Request details at the load balancer
→ ALB Access Logs

Trace the request through multiple application services
→ X-Ray
```

## Trap 14 — Health checks vs Access Logs

```text
Is the target healthy?
→ ELB Health Check

What requests are going through the ALB?
→ ALB Access Logs
```

## Trap 15 — Application Insights vs Access Logs

```text
Detailed ALB HTTP traffic
→ ALB Access Logs

Application monitoring / troubleshooting
→ CloudWatch Application Insights
```

They can be used together when the question requires both.

---

# Pocket Card

| Keyword                             | Answer                             |
| ----------------------------------- | ---------------------------------- |
| Performance / health / metrics      | **CloudWatch**                     |
| Logs                                | **CloudWatch Logs**                |
| Alarm on a metric                   | **CloudWatch Alarm**               |
| Count log messages → metric         | **Metric Filter**                  |
| Query logs                          | **Logs Insights**                  |
| Real-time log processing            | **Subscription Filter**            |
| Reduce alert noise                  | **Composite Alarm**                |
| EC2 memory                          | **CloudWatch Agent**               |
| EC2 swap                            | **CloudWatch Agent**               |
| EC2 filesystem disk usage           | **CloudWatch Agent**               |
| EC2 process metrics                 | **CloudWatch Agent / procstat**    |
| EC2 metrics every 1 minute          | **Detailed Monitoring**            |
| EKS/ECS container monitoring        | **CloudWatch Container Insights**  |
| RDS process-level CPU/memory        | **Enhanced Monitoring**            |
| RDS query/database load             | **Performance Insights**           |
| Detailed HTTP requests through ALB  | **ALB Access Logs**                |
| ALB client IP                       | **ALB Access Logs**                |
| ALB request/target/response latency | **ALB Access Logs**                |
| ALB aggregate request count         | **CloudWatch Metrics**             |
| ALB health                          | **ELB Health Checks**              |
| ALB/application troubleshooting     | **Application Insights**           |
| Who did what / API audit            | **CloudTrail**                     |
| Long-term API logs                  | **CloudTrail Trail → S3**          |
| S3 object-level "who"               | **CloudTrail Data Events**         |
| Unusual API activity                | **CloudTrail Insights**            |
| Prove logs weren't modified         | **Log File Integrity Validation**  |
| CloudTrail logs encrypted           | **S3 SSE-S3 by default**           |
| New S3 objects encrypted            | **SSE-S3 by default**              |
| Customer-controlled S3 key          | **SSE-KMS**                        |
| Existing old unencrypted data       | **Not automatically re-encrypted** |
| Configuration history               | **AWS Config**                     |
| Compliance checking                 | **AWS Config Rules**               |
| IAM access key >90 days             | **Config `access-keys-rotated`**   |
| Configure IAM access-key age        | **`maxAccessKeyAge`**              |
| Auto deactivate/delete old IAM key  | **Config → EventBridge → Lambda**  |
| Automatically fix Config violations | **Config + remediation**           |
| Prevent an action                   | **IAM / SCP**                      |
| Trace request across services       | **X-Ray**                          |
