# Section 37F: Systems Manager & Operations

## The idea

These are AWS services that commonly appear in SAA questions involving **instance management, secure access, command execution, patching, distributed tracing, and Prometheus-compatible monitoring**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Secure shell access to private EC2 without SSH
→ SSM Session Manager

Run the same command on hundreds of EC2 instances
→ SSM Run Command

Automatically patch hundreds of EC2 instances
→ SSM Patch Manager

Find which microservice is causing latency
→ AWS X-Ray

Prometheus / PromQL + container metrics
→ Amazon Managed Service for Prometheus
```

---

# AWS Systems Manager

**AWS Systems Manager = a collection of tools for managing and operating AWS and supported hybrid infrastructure.**

The most important services for SAA questions in this section are:

* Session Manager
* Run Command
* Patch Manager

The easiest way to remember them is:

```text
Session Manager
= ACCESS

Run Command
= RUN

Patch Manager
= PATCH
```

---

# Session Manager

**AWS Systems Manager Session Manager = secure shell access to EC2 instances without traditional SSH.**

It allows administrators to connect to managed instances without requiring:

* a bastion host
* an open inbound port 22
* SSH keys for the session itself

### Typical pattern

```text
Admin
  ↓
Session Manager
  ↓
Private EC2
```

The instance can remain private without exposing SSH to the Internet.

### Signal

> **Secure access to private EC2 without SSH → Session Manager**

---

## Example

> "Administrators need secure shell access to private EC2 instances, but the company does not want to open port 22 or maintain a bastion host."

→ **AWS Systems Manager Session Manager**

### Memory

> **Session Manager = ACCESS instances**

---

# Run Command

**AWS Systems Manager Run Command = execute commands or scripts on one or many managed instances.**

This is useful when the same operation needs to be performed across many servers.

### Example

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

## Example

> "An administrator needs to execute the same shell script on hundreds of EC2 instances."

→ **SSM Run Command**

### Memory

> **Run Command = RUN commands**

---

# Patch Manager

**AWS Systems Manager Patch Manager = automate OS patching for managed instances.**

It can be used to automate patching across fleets of servers.

### Example

> "Apply security patches to 500 EC2 instances every month."

→ **Patch Manager**

### Signal

> **Automatically patch many EC2 instances → Patch Manager**

### Memory

> **Patch Manager = PATCH instances**

---

# Session Manager vs Run Command vs Patch Manager

These three are very easy to mix up.

| Service             | Main purpose                             | Signal                                 |
| ------------------- | ---------------------------------------- | -------------------------------------- |
| **Session Manager** | Interactive secure access to an instance | Access private EC2 without SSH         |
| **Run Command**     | Execute commands/scripts                 | Run the same command on many instances |
| **Patch Manager**   | Automate OS patching                     | Patch many instances                   |

### Mental model

```text
Need to ACCESS an instance
→ Session Manager

Need to RUN a command
→ Run Command

Need to PATCH instances
→ Patch Manager
```

---

# Hybrid Systems Manager

Systems Manager can also manage supported **on-premises servers** when the SSM Agent and required connectivity are configured.

This means Systems Manager is not limited to EC2.

A hybrid environment can be managed through Systems Manager as well.

### Memory

```text
AWS instances
+
supported on-premises servers
→ Systems Manager
```

---

# AWS X-Ray

**AWS X-Ray = distributed tracing.**

It follows a request as it moves through different services in a distributed application.

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

### Signal

> **Find which microservice is causing latency → X-Ray**

---

# Why X-Ray is useful

In a distributed application, a request may travel through multiple services.

Without tracing, you may know that a request took five seconds, but not which component caused the delay.

For example:

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

Suppose Service B takes most of the time.

X-Ray helps trace the request through the architecture and identify where the latency is occurring.

### Memory

> **X-Ray = TRACE one request across services**

---

# CloudWatch vs X-Ray

This is an important SAA distinction.

```text
CloudWatch
= metrics + logs + monitoring

X-Ray
= trace one request across services
```

### CloudWatch

Think:

```text
CPU utilization
Memory
Application logs
Alarms
Metrics
```

### X-Ray

Think:

```text
Request
 ↓
Service A
 ↓
Service B
 ↓
Service C

Where did the request become slow?
Where did it fail?
```

### Example

> "A distributed application has high latency and the team needs to identify which microservice is responsible."

→ **AWS X-Ray**

---

# Amazon Managed Service for Prometheus

**Amazon Managed Service for Prometheus = managed, Prometheus-compatible monitoring and alerting for container workloads.**

It is especially useful for monitoring:

* Amazon EKS
* Amazon ECS
* AWS Fargate
* Kubernetes environments

It uses the **Prometheus data model and PromQL** for querying metrics.

### Signal

> **Prometheus / PromQL + container metrics → Amazon Managed Service for Prometheus**

---

# Typical architecture

```text
Container workloads
        ↓
Prometheus metrics
        ↓
Amazon Managed Service for Prometheus
        ↓
PromQL / Grafana
```

Amazon Managed Grafana can be used to visualize Prometheus metrics.

### Memory

> **Managed Service for Prometheus = MANAGED PROMETHEUS**

---

# Prometheus and containers

A question may mention:

* Kubernetes
* EKS
* containers
* Prometheus
* PromQL
* metrics
* monitoring

These clues strongly point toward:

→ **Amazon Managed Service for Prometheus**

### Example

> "A company uses Kubernetes and wants to use Prometheus and PromQL for monitoring without managing the Prometheus infrastructure."

→ **Amazon Managed Service for Prometheus**

---

# Amazon Managed Service for Prometheus vs CloudWatch

These services can both be used for monitoring, but the signal is different.

```text
CloudWatch
= AWS-native metrics, logs, and monitoring

Amazon Managed Service for Prometheus
= Prometheus-compatible metrics
= PromQL
= especially useful for container/Kubernetes monitoring
```

### Important distinction

If the question emphasizes:

```text
Prometheus
PromQL
Kubernetes
container metrics
```

→ **Amazon Managed Service for Prometheus**

If it emphasizes:

```text
AWS metrics
logs
alarms
AWS resource monitoring
```

→ **CloudWatch**

---

# Amazon Managed Service for Prometheus + Grafana

Prometheus stores and provides the metrics/querying model, while Grafana can be used to visualize the metrics.

Typical pattern:

```text
Container / Kubernetes workloads
             ↓
       Prometheus metrics
             ↓
Amazon Managed Service for Prometheus
             ↓
         PromQL queries
             ↓
   Amazon Managed Grafana
             ↓
       Dashboards
```

### Important

> **Prometheus service = metrics/querying**

> **Grafana = visualization**

---

# Operations Service Comparison

| Service                            | What it does                             | Signal keyword                            |
| ---------------------------------- | ---------------------------------------- | ----------------------------------------- |
| **SSM Session Manager**            | Secure interactive access to EC2         | Private EC2 without SSH                   |
| **SSM Run Command**                | Run commands/scripts on instances        | Same command on many instances            |
| **SSM Patch Manager**              | Automate OS patching                     | Patch many instances                      |
| **AWS X-Ray**                      | Distributed request tracing              | Find latency/failure across microservices |
| **Managed Service for Prometheus** | Managed Prometheus-compatible monitoring | Prometheus / PromQL / Kubernetes          |

---

# Systems Manager Decision Tree

When you see an SSM question, ask:

```text
What does the administrator need to do?
          │
          ├── Access an instance interactively?
          │       ↓
          │   Session Manager
          │
          ├── Run a command/script?
          │       ↓
          │    Run Command
          │
          └── Patch the operating system?
                  ↓
              Patch Manager
```

---

# Monitoring Decision Tree

```text
What is the monitoring requirement?
          │
          ├── Trace one request through
          │   multiple services?
          │       ↓
          │      X-Ray
          │
          ├── Prometheus / PromQL?
          │       ↓
          │   Managed Service
          │   for Prometheus
          │
          └── General AWS metrics/logs/alarms?
                  ↓
               CloudWatch
```

---

# Common Question Patterns

> **"Secure shell access to private EC2 without SSH or a bastion."**

→ **SSM Session Manager**

---

> **"Run the same command on hundreds of EC2 instances."**

→ **SSM Run Command**

---

> **"Execute a script across a fleet of EC2 instances."**

→ **SSM Run Command**

---

> **"Automatically patch hundreds of EC2 instances."**

→ **SSM Patch Manager**

---

> **"Apply security patches to EC2 instances on a regular schedule."**

→ **SSM Patch Manager**

---

> **"Find which microservice is causing latency in a request."**

→ **AWS X-Ray**

---

> **"Trace a request across API Gateway, Lambda, and multiple downstream services."**

→ **AWS X-Ray**

---

> **"A company uses Kubernetes and wants Prometheus and PromQL for monitoring without managing Prometheus infrastructure."**

→ **Amazon Managed Service for Prometheus**

---

> **"The company needs managed Prometheus-compatible metrics for EKS workloads."**

→ **Amazon Managed Service for Prometheus**

---

# Important SAA Traps

## Session Manager vs Run Command

Both are part of Systems Manager, but they serve different purposes.

```text
Interactive shell / access
→ Session Manager
```

```text
Execute commands or scripts
→ Run Command
```

Example:

> "Connect to a private EC2 instance and inspect files interactively."

→ **Session Manager**

But:

> "Run the same command on 500 EC2 instances."

→ **Run Command**

---

## Run Command vs Patch Manager

Patching is a specific operational task.

```text
General command/script
→ Run Command
```

```text
OS patching
→ Patch Manager
```

---

## Session Manager vs SSH

If the question says:

> "Access private instances without opening port 22."

Think:

```text
Session Manager
```

You do not need:

* a public IP
* inbound SSH port 22
* a bastion host

for the Session Manager access pattern.

---

## X-Ray vs CloudWatch

Look at the thing being investigated.

```text
Metrics / logs / alarms
→ CloudWatch
```

```text
Request path / service latency
→ X-Ray
```

---

## Prometheus vs CloudWatch

Look for the monitoring technology named in the question.

```text
Prometheus
PromQL
Kubernetes metrics
→ Managed Service for Prometheus
```

```text
AWS-native monitoring
Metrics
Logs
Alarms
→ CloudWatch
```

---

# Hybrid Environment Example

Systems Manager can manage supported on-premises servers in addition to AWS instances when configured appropriately.

For example:

```text
AWS EC2 instances
        +
On-premises servers
        ↓
Systems Manager
        ↓
Operations / management
```

This can allow the same operational tooling to be used across a hybrid environment.

### Important

The on-premises server must be properly configured for Systems Manager, including the required agent and connectivity.

---

# Pocket Card

| Keyword                                   | Answer                                    |
| ----------------------------------------- | ----------------------------------------- |
| Secure instance shell without SSH         | **SSM Session Manager**                   |
| Private EC2 without bastion               | **SSM Session Manager**                   |
| Run commands across instances             | **SSM Run Command**                       |
| Run same script on many instances         | **SSM Run Command**                       |
| Automated OS patching                     | **SSM Patch Manager**                     |
| Patch many instances                      | **SSM Patch Manager**                     |
| Distributed request tracing               | **AWS X-Ray**                             |
| Find microservice latency                 | **AWS X-Ray**                             |
| Prometheus / PromQL                       | **Amazon Managed Service for Prometheus** |
| Kubernetes / container Prometheus metrics | **Amazon Managed Service for Prometheus** |

---

# Final Memory

```text
SSM Session Manager
= SECURE INSTANCE ACCESS
= NO SSH / NO BASTION REQUIRED

SSM Run Command
= RUN COMMANDS ON MANY INSTANCES

SSM Patch Manager
= PATCH INSTANCES

AWS X-Ray
= DISTRIBUTED TRACING
= TRACE REQUESTS ACROSS SERVICES

Amazon Managed Service for Prometheus
= MANAGED PROMETHEUS
= PROMQL
= CONTAINER / KUBERNETES METRICS
```

# The Golden Rule

```text
Secure access to private EC2
→ Session Manager

Run commands across instances
→ Run Command

Patch instances
→ Patch Manager

Find which service causes request latency
→ X-Ray

Prometheus / PromQL / Kubernetes metrics
→ Managed Service for Prometheus
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Private EC2 + no SSH     → Session Manager
Many instances + command → Run Command
Many instances + patches → Patch Manager
Distributed latency      → X-Ray
Prometheus / PromQL      → Managed Prometheus
```
