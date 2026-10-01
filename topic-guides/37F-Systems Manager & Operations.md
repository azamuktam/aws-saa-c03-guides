# Section 37F: Systems Manager & Operations

## The idea

These AWS services commonly appear in SAA questions involving **instance management, secure access, command execution, patching, AWS operational events, and Prometheus-compatible monitoring**.

Best strategy:

> **Read the requirement → identify the unique keyword → choose the service.**

```text
Secure shell access to private EC2 without SSH
→ AWS Systems Manager Session Manager

Run the same command on hundreds of EC2 instances
→ AWS Systems Manager Run Command

Automatically patch hundreds of EC2 instances
→ AWS Systems Manager Patch Manager

AWS event specifically associated with your account/resources
→ AWS Health Dashboard – Your account health
   (older SAA material: Personal Health Dashboard)

Automatically react to AWS Health events
→ Amazon EventBridge

Send notifications
→ Amazon Simple Notification Service (Amazon SNS)

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

```text
Session Manager = ACCESS
Run Command     = RUN
Patch Manager   = PATCH
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

# Session Manager vs Run Command vs Patch Manager

| Service                                 | Main purpose              | Signal                         |
| --------------------------------------- | ------------------------- | ------------------------------ |
| **AWS Systems Manager Session Manager** | Interactive secure access | Access private EC2 without SSH |
| **AWS Systems Manager Run Command**     | Execute commands/scripts  | Same command on many instances |
| **AWS Systems Manager Patch Manager**   | Automate OS patching      | Patch many instances           |

```text
ACCESS an instance
→ AWS Systems Manager Session Manager

RUN a command/script
→ AWS Systems Manager Run Command

PATCH the OS
→ AWS Systems Manager Patch Manager
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
= general AWS service/Regional event
= not specific to my account

ACCOUNT-SPECIFIC
= specific to my account/organization
= affected resources may be identified
```

The distinction is **not** whether the event could affect resources generally; public events can also affect users. The distinction is whether the event is **specific to your account**.

---

# EventBridge + AWS Health

**Amazon EventBridge** can detect and route AWS Health events so you can automate actions or notifications.

Typical notification pattern:

```text
AWS Health event
      ↓
Amazon EventBridge
      ↓
Amazon Simple Notification Service (Amazon SNS)
      ↓
Notification
```

### Roles

```text
AWS Health
= provides the health event

Amazon EventBridge
= detects/routes/triggers from the event

Amazon SNS
= sends the notification
```

### Example

> "Notify administrators about upcoming AWS events that may affect specific EC2 instances."

→ **AWS Health Dashboard – Your account health + Amazon EventBridge + Amazon SNS**

This matches the older SAA wording:

→ **Personal Health Dashboard + EventBridge + SNS**

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

## EventBridge vs SNS

```text
Amazon EventBridge
= detect / route / trigger

Amazon Simple Notification Service (Amazon SNS)
= send notifications
```

---

# AWS Health Question Patterns

> **"An EC2 instance was unexpectedly powered down. Management wants notifications about upcoming AWS events that may affect their EC2 instances."**

→ **AWS Health Dashboard – Your account health + Amazon EventBridge + Amazon SNS**

---

> **"A company wants general information about the status of an AWS service."**

→ **AWS Health Dashboard – Service health**

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

| Service                                                                              | What it does                   | Signal keyword                         |
| ------------------------------------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| **AWS Systems Manager Session Manager**                                              | Secure interactive EC2 access  | Private EC2 without SSH                |
| **AWS Systems Manager Run Command**                                                  | Run commands/scripts           | Same command on many instances         |
| **AWS Systems Manager Patch Manager**                                                | Automate OS patching           | Patch many instances                   |
| **AWS Health Dashboard – Your account health** / **Personal Health Dashboard (PHD)** | Account-specific Health events | Event specific to my account/resources |
| **AWS Health Dashboard – Service health**                                            | Public AWS service events      | General service/Regional status        |
| **Amazon EventBridge**                                                               | Detect/route AWS Health events | Automatically react to an event        |
| **Amazon Simple Notification Service (Amazon SNS)**                                  | Send notifications             | Notify administrators                  |
| **Amazon Managed Service for Prometheus (AMP)**                                      | Managed Prometheus monitoring  | Prometheus / PromQL / Kubernetes       |

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
          └── Patch the operating system?
                  ↓
          AWS Systems Manager Patch Manager
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
                  ↓
          Amazon Simple Notification Service (Amazon SNS)
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

## EventBridge vs SNS

```text
Amazon EventBridge
= detect / route / trigger

Amazon Simple Notification Service (Amazon SNS)
= send notifications
```

Typical pattern:

```text
AWS Health event
→ Amazon EventBridge
→ Amazon SNS
→ Administrators
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
| Event specific to my account/resources    | **AWS Health Dashboard – Your account health / Personal Health Dashboard (PHD)** |
| General AWS service status                | **AWS Health Dashboard – Service health**                                        |
| Automatically react to AWS Health event   | **Amazon EventBridge**                                                           |
| Send AWS Health notifications             | **Amazon Simple Notification Service (Amazon SNS)**                              |
| Prometheus / PromQL                       | **Amazon Managed Service for Prometheus (AMP)**                                  |
| Kubernetes / container Prometheus metrics | **Amazon Managed Service for Prometheus (AMP)**                                  |
