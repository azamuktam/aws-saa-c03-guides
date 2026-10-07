# Section 37F: Systems Manager & Operations

## The idea

These AWS services commonly appear in SAA questions involving **instance management, secure access, command execution, patching, operational issues, AWS operational events, and Prometheus-compatible monitoring**.

Best strategy:

> **Read the requirement → identify the unique keyword → choose the service.**

```text
Secure shell access to private EC2 without SSH
→ AWS Systems Manager Session Manager

Run the same command on hundreds of EC2 instances
→ AWS Systems Manager Run Command

Automatically patch hundreds of EC2 instances
→ AWS Systems Manager Patch Manager

Manage / inspect EC2 instances
→ AWS Systems Manager Fleet Manager

Centrally track and investigate operational issues
→ AWS Systems Manager OpsCenter

AWS event specifically associated with your account/resources
→ AWS Health Dashboard – Your account health
   (older SAA material: Personal Health Dashboard)

Automatically react to AWS Health events
→ Amazon EventBridge

Prometheus / PromQL + container metrics
→ Amazon Managed Service for Prometheus (AMP)
```

---

# AWS Systems Manager

**AWS Systems Manager (SSM) = a collection of tools for managing and operating AWS and supported hybrid infrastructure.**

Most important SAA services:

* **AWS Systems Manager Session Manager**
* **AWS Systems Manager Run Command**
* **AWS Systems Manager Patch Manager**
* **AWS Systems Manager Fleet Manager**
* **AWS Systems Manager OpsCenter**

```text
Session Manager = ACCESS
Run Command     = RUN
Patch Manager   = PATCH
Fleet Manager   = MANAGE / INSPECT
OpsCenter       = TRACK / INVESTIGATE ISSUES
```

---

# Session Manager

**AWS Systems Manager Session Manager = secure interactive shell access to managed EC2 instances without traditional SSH.**

You can avoid:

* a bastion host
* inbound port 22
* exposing SSH to the Internet

Typical pattern:

```text
Admin
  ↓
AWS Systems Manager Session Manager
  ↓
Private EC2
```

### Signal

> **Secure access to private EC2 without SSH → AWS Systems Manager Session Manager**

### Example

> "Administrators need secure shell access to private EC2 instances without opening port 22 or maintaining a bastion host."

→ **AWS Systems Manager Session Manager**

### Memory

> **Session Manager = ACCESS instances**

---

# Run Command

**AWS Systems Manager Run Command = execute commands or scripts on one or many managed instances.**

Example:

```text
500 EC2 instances
      ↓
AWS Systems Manager Run Command
      ↓
Run the same script
```

### Signal

> **Run a command/script across many EC2 instances → AWS Systems Manager Run Command**

### Examples

> "Execute the same shell script on hundreds of EC2 instances."

→ **AWS Systems Manager Run Command**

> "Execute a script across a fleet of EC2 instances."

→ **AWS Systems Manager Run Command**

### Memory

> **Run Command = RUN commands**

---

# Patch Manager

**AWS Systems Manager Patch Manager = automate OS patching for managed instances.**

### Signal

> **Automatically patch many EC2 instances → AWS Systems Manager Patch Manager**

### Examples

> "Apply security patches to 500 EC2 instances every month."

→ **AWS Systems Manager Patch Manager**

> "Apply security patches to EC2 instances on a regular schedule."

→ **AWS Systems Manager Patch Manager**

### Memory

> **Patch Manager = PATCH instances**

---

# Fleet Manager

**AWS Systems Manager Fleet Manager = remotely inspect and manage EC2 instances.**

It helps administrators view and manage things such as:

* Files
* Processes
* Services
* Windows Registry
* Instance information

### Signal

> **Manage / inspect EC2 instances remotely → Fleet Manager**

### Example

> "Administrators need to inspect files and Windows services on EC2 instances without manually connecting to each server."

→ **AWS Systems Manager Fleet Manager**

### Memory

> **Fleet Manager = MANAGE / INSPECT EC2**

---

# OpsCenter

**AWS Systems Manager OpsCenter = centrally track, investigate, and manage operational issues.**

It gives operations teams a central place to work on **operational problems and incidents**.

Typical pattern:

```text
Operational issue
      ↓
AWS Systems Manager OpsCenter
      ↓
Track / investigate / manage
```

### Signal

> **Centrally track and investigate operational issues → AWS Systems Manager OpsCenter**

### Example

> "The operations team needs one place to track and investigate operational issues across AWS resources."

→ **AWS Systems Manager OpsCenter**

### Memory

> **OpsCenter = TRACK / INVESTIGATE operational issues**

---

# Session Manager vs Run Command vs Patch Manager vs Fleet Manager vs OpsCenter

| Service                                 | Main purpose                           | Signal                               |
| --------------------------------------- | -------------------------------------- | ------------------------------------ |
| **AWS Systems Manager Session Manager** | Interactive secure access              | Access private EC2 without SSH       |
| **AWS Systems Manager Run Command**     | Execute commands/scripts               | Same command on many instances       |
| **AWS Systems Manager Patch Manager**   | Automate OS patching                   | Patch many instances                 |
| **AWS Systems Manager Fleet Manager**   | Manage / inspect EC2                   | Files, processes, services, Registry |
| **AWS Systems Manager OpsCenter**       | Track / investigate operational issues | Central operational issue management |

```text
ACCESS an instance
→ AWS Systems Manager Session Manager

RUN a command/script
→ AWS Systems Manager Run Command

PATCH the OS
→ AWS Systems Manager Patch Manager

MANAGE / INSPECT an EC2 fleet
→ AWS Systems Manager Fleet Manager

TRACK / INVESTIGATE operational issues
→ AWS Systems Manager OpsCenter
```

---

# Hybrid Systems Manager

**AWS Systems Manager (SSM)** can also manage supported **on-premises servers** when the Systems Manager Agent and required connectivity are configured.

```text
AWS instances
+
Supported on-premises servers
→ AWS Systems Manager
```

### Important

The on-premises server must be properly configured, including the required agent and connectivity.

---

# AWS Health / Personal Health Dashboard

**AWS Health Dashboard** provides AWS Health information for both **public service events** and **account-specific events**. Older SAA material may refer to the account-specific view as the **AWS Personal Health Dashboard (PHD)**.

## Public / Service Health

**AWS Health Dashboard – Service health = public/general AWS service events.**

Example:

```text
Is Amazon EC2 experiencing
a service issue in this Region?
```

Public events are **not specific to an AWS account** and can describe a Regional service issue even when you do not use that service there.

### Signal

> **General AWS service status → AWS Health Dashboard – Service health**

---

## Account-Specific / Your Account Health

The signed-in **AWS Health Dashboard – Your account health** shows events specific to your account, including **upcoming scheduled changes** and affected resources.

Older SAA material commonly calls this the **AWS Personal Health Dashboard (PHD)**.

```text
AWS event
specific to your account/resources
        ↓
AWS Health Dashboard
Your account health
```

### Signal

> **AWS event specifically associated with my account/resources → AWS Health Dashboard – Your account health**

### Important distinction

```text
PUBLIC
= general AWS service / Regional event
= not specific to my account

ACCOUNT-SPECIFIC
= specific to my account / organization
= affected resources may be identified
```

The distinction is **not** whether the event could affect resources generally. Public events can also affect users. The distinction is whether the event is **specific to your account**.

---

# EventBridge + AWS Health

**Amazon EventBridge** can detect and route AWS Health events so you can automate actions.

Typical pattern:

```text
AWS Health event
      ↓
Amazon EventBridge
      ↓
Automation / target action
```

### Roles

```text
AWS Health
= provides the health event

Amazon EventBridge
= detects / routes / triggers from the event
```

### Example

> "Automatically react when AWS reports an event that may affect resources in the account."

→ **AWS Health Dashboard – Your account health + Amazon EventBridge**

This matches the older SAA wording:

→ **Personal Health Dashboard + EventBridge**

---

# Common AWS Health Traps

## Service Health vs Account-Specific Health

```text
General AWS service / Regional status
→ AWS Health Dashboard – Service health

Event specific to my account/resources
→ AWS Health Dashboard – Your account health
  (older term: Personal Health Dashboard)
```

## EventBridge

```text
Amazon EventBridge
= detect / route / trigger
```

---

# AWS Health Question Patterns

> **"An EC2 instance may be affected by an upcoming AWS event specific to the company's account."**

→ **AWS Health Dashboard – Your account health**

---

> **"A company wants general information about the status of an AWS service."**

→ **AWS Health Dashboard – Service health**

---

> **"Automatically take action when an AWS Health event occurs."**

→ **Amazon EventBridge**

---

# Amazon Managed Service for Prometheus

**Amazon Managed Service for Prometheus (AMP) = managed, Prometheus-compatible monitoring and alerting.**

Especially relevant to:

* Amazon EKS
* Amazon ECS
* AWS Fargate
* Kubernetes
* container workloads

Uses the **Prometheus data model and PromQL**.

### Signal

> **Prometheus / PromQL + container/Kubernetes metrics → Amazon Managed Service for Prometheus (AMP)**

### Examples

> "A company uses Kubernetes and wants Prometheus and PromQL for monitoring without managing Prometheus infrastructure."

→ **Amazon Managed Service for Prometheus (AMP)**

> "The company needs managed Prometheus-compatible metrics for EKS workloads."

→ **Amazon Managed Service for Prometheus (AMP)**

---
Examples of **Prometheus metrics** are simple numerical measurements collected over time:

```text
cpu_usage_percent = 72
memory_usage_bytes = 4294967296
http_requests_total = 154320
http_request_duration_seconds = 0.42
http_errors_total = 37
active_connections = 128
```

For Kubernetes:

```text
pod_cpu_usage
pod_memory_usage
container_restarts_total
container_network_receive_bytes
```

Think:

> **Prometheus metrics = numbers about how your application/infrastructure is behaving over time.**

**Prometheus then stores and queries these metrics using PromQL.**
 
# Typical AMP Architecture

```text
Container / Kubernetes workloads
             ↓
       Prometheus metrics
             ↓
Amazon Managed Service for Prometheus (AMP)
             ↓
         PromQL queries
             ↓
     Amazon Managed Grafana
             ↓
         Dashboards
```

### Important

```text
Amazon Managed Service for Prometheus (AMP)
= metrics + Prometheus querying

Amazon Managed Grafana
= visualization
```

---

# Operations Service Comparison

| Service                                                                          | What it does                           | Signal keyword                         |
| -------------------------------------------------------------------------------- | -------------------------------------- | -------------------------------------- |
| **AWS Systems Manager Session Manager**                                          | Secure interactive EC2 access          | Private EC2 without SSH                |
| **AWS Systems Manager Run Command**                                              | Run commands/scripts                   | Same command on many instances         |
| **AWS Systems Manager Patch Manager**                                            | Automate OS patching                   | Patch many instances                   |
| **AWS Systems Manager Fleet Manager**                                            | Manage / inspect EC2                   | Files, processes, services, Registry   |
| **AWS Systems Manager OpsCenter**                                                | Track / investigate operational issues | Central operational issue management   |
| **AWS Health Dashboard – Your account health / Personal Health Dashboard (PHD)** | Account-specific Health events         | Event specific to my account/resources |
| **AWS Health Dashboard – Service health**                                        | Public AWS service events              | General service / Regional status      |
| **Amazon EventBridge**                                                           | Detect / route AWS Health events       | Automatically react to an event        |
| **Amazon Managed Service for Prometheus (AMP)**                                  | Managed Prometheus monitoring          | Prometheus / PromQL / Kubernetes       |

---

# Systems Manager Decision Tree

```text
What does the administrator need to do?
          │
          ├── Access an instance interactively?
          │       ↓
          │   AWS Systems Manager Session Manager
          │
          ├── Run a command/script?
          │       ↓
          │   AWS Systems Manager Run Command
          │
          ├── Patch the operating system?
          │       ↓
          │   AWS Systems Manager Patch Manager
          │
          ├── Inspect / manage an EC2 fleet?
          │       ↓
          │   AWS Systems Manager Fleet Manager
          │
          └── Track / investigate operational issues?
                  ↓
              AWS Systems Manager OpsCenter
```

---

# AWS Health Decision Tree

```text
What kind of Health information is needed?
          │
          ├── General AWS service / Regional status?
          │       ↓
          │   AWS Health Dashboard – Service health
          │
          └── Event specific to my account/resources?
                  ↓
          AWS Health Dashboard – Your account health
          (older term: Personal Health Dashboard)
                  ↓
              Amazon EventBridge
```

---

# Important SAA Traps

## Session Manager vs Run Command

```text
Interactive shell / access
→ AWS Systems Manager Session Manager
```

```text
Execute commands/scripts
→ AWS Systems Manager Run Command
```

Example:

> "Connect to a private EC2 instance and inspect files interactively."

→ **AWS Systems Manager Session Manager**

> "Run the same command on 500 EC2 instances."

→ **AWS Systems Manager Run Command**

---

## Run Command vs Patch Manager

```text
General command/script
→ AWS Systems Manager Run Command

OS patching
→ AWS Systems Manager Patch Manager
```

---

## Fleet Manager vs Session Manager

```text
Interactive shell access
→ Session Manager

Manage / inspect instance details
→ Fleet Manager
```

---

## OpsCenter vs the other Systems Manager tools

```text
Need to access EC2
→ Session Manager

Need to run commands
→ Run Command

Need to patch OS
→ Patch Manager

Need to inspect/manage instances
→ Fleet Manager

Need to track/investigate operational issues
→ OpsCenter
```

---

## Session Manager vs SSH

> **"Access private instances without opening port 22."**

→ **AWS Systems Manager Session Manager**

Typical pattern does not require:

* public IP
* inbound SSH port 22
* bastion host

---

## Account-Specific Health vs Public Health

```text
General AWS service status
→ AWS Health Dashboard – Service health

Event specific to my account/resources
→ AWS Health Dashboard – Your account health
   (older term: Personal Health Dashboard)
```

A public EC2 event can still potentially affect your resources; what distinguishes it is that it is **not specific to your account**. Account-specific events can identify affected resources.

---

## EventBridge

```text
Amazon EventBridge
= detect / route / trigger
```

Typical pattern:

```text
AWS Health event
→ Amazon EventBridge
→ Automated action
```

---

# Hybrid Environment Example

**AWS Systems Manager (SSM)** can manage supported on-premises servers as well as AWS instances when configured appropriately.

```text
AWS EC2 instances
        +
On-premises servers
        ↓
AWS Systems Manager
        ↓
Operations / management
```

### Important

The on-premises server requires the required agent and connectivity configuration.

---

# Pocket Card

| Keyword                                   | Answer                                                                           |
| ----------------------------------------- | -------------------------------------------------------------------------------- |
| Secure instance shell without SSH         | **AWS Systems Manager Session Manager**                                          |
| Private EC2 without bastion               | **AWS Systems Manager Session Manager**                                          |
| Run commands across instances             | **AWS Systems Manager Run Command**                                              |
| Run same script on many instances         | **AWS Systems Manager Run Command**                                              |
| Automated OS patching                     | **AWS Systems Manager Patch Manager**                                            |
| Patch many instances                      | **AWS Systems Manager Patch Manager**                                            |
| Manage / inspect EC2 instances            | **AWS Systems Manager Fleet Manager**                                            |
| Track / investigate operational issues    | **AWS Systems Manager OpsCenter**                                                |
| Event specific to my account/resources    | **AWS Health Dashboard – Your account health / Personal Health Dashboard (PHD)** |
| General AWS service status                | **AWS Health Dashboard – Service health**                                        |
| Automatically react to AWS Health event   | **Amazon EventBridge**                                                           |
| Prometheus / PromQL                       | **Amazon Managed Service for Prometheus (AMP)**                                  |
| Kubernetes / container Prometheus metrics | **Amazon Managed Service for Prometheus (AMP)**                                  |
**
**
