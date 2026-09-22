# Section 14: ECS, EKS & Fargate

## The idea

Containers package an application together with everything it needs to run:

* Application code
* Runtime
* Libraries and dependencies
* Configuration
* OS-level dependencies

This package is called a **container image**.

Instead of installing the application directly on a server, you run the image as a **container**.

Containers are useful when you want more control than Lambda but do not necessarily want to manage a full server for every application.

### The problem with containers

One application may need only a few containers. A production system may need hundreds.

You need something that can:

* Start containers
* Stop and replace failed containers
* Run multiple copies of an application
* Distribute containers across compute resources
* Connect containers to load balancers
* Scale the number of running containers

This is the job of a **container orchestrator**.

AWS provides two major orchestration options:

| Service | What it provides                   |
| ------- | ---------------------------------- |
| **ECS** | AWS-native container orchestration |
| **EKS** | Managed Kubernetes                 |

**Fargate is not an orchestrator.**
Fargate is a **serverless compute option** that can run containers managed by ECS or EKS.

```text
                 Container orchestration
                    ECS or EKS
                       |
            -------------------------
            |                       |
         EC2 compute             Fargate
       You manage hosts       AWS manages hosts
```

---

# ECS vocabulary

Learn these four terms clearly.

| ECS term            | Meaning                                            |
| ------------------- | -------------------------------------------------- |
| **Task Definition** | Configuration that describes how a task should run |
| **Task**            | A running instance of a task definition            |
| **Service**         | Maintains a desired number of running tasks        |
| **Cluster**         | Logical grouping used to organize ECS resources    |

### Task Definition

A **Task Definition** is the configuration for a containerized workload.

It can specify:

* Container image
* CPU and memory
* Ports
* Environment variables
* Logging
* Task role
* Task execution role
* Network settings
* Secrets

Think:

**Task Definition = "How should this container workload run?"**

### Task

A **Task** is an actual running instance of a task definition.

```text
Task Definition
      ↓
   run it
      ↓
    Task
```

For example:

```text
Task Definition: "web-app"

Desired count: 3

→ Task 1
→ Task 2
→ Task 3
```

### Service

An ECS **Service** maintains a desired number of tasks.

For example:

```text
Desired count = 3

Task 1   Running
Task 2   Running
Task 3   Running
```

If Task 2 crashes:

```text
Task 2   ❌
   ↓
ECS starts replacement
   ↓
Task 4   ✅
```

A service can also integrate with load balancers and Service Auto Scaling.

### Cluster

An ECS **Cluster** is a logical grouping for ECS resources and workloads.

For exam questions, remember:

```text
Cluster
   ↓
Service
   ↓
Tasks
   ↓
Containers
```

---

# ECS launch options

For ECS, the two important compute choices are:

## ECS on EC2

Containers run on EC2 instances that you manage.

You control:

* EC2 instance types
* OS and AMI
* Capacity
* Scaling of instances
* Installed software
* GPUs
* Spot usage

Advantages:

* More control
* GPU support
* Can use EC2 Spot Instances
* Can choose specialized instance types

Disadvantages:

* You manage the underlying instances
* You must think about EC2 capacity, patching, and scaling

---

# ECS on Fargate

Fargate provides **serverless container compute**.

You specify resources such as:

* CPU
* Memory
* Networking
* Container configuration

AWS manages the underlying infrastructure.

You do not manage the EC2 instances running your containers.

### Main exam trigger

> **"Run containers without managing servers/infrastructure"**

→ **Fargate**

### Important limitation

Fargate does **not** support GPUs.

Therefore:

> **Containerized workload requires GPU**

→ **ECS on EC2**

```text
Containers + no server management
        → Fargate

Containers + GPU
        → ECS on EC2
```

---

# Fargate vs EC2

| Requirement                                |         ECS on EC2 |                Fargate |
| ------------------------------------------ | -----------------: | ---------------------: |
| Manage EC2 instances                       |                Yes |                     No |
| Serverless containers                      |                 No |                    Yes |
| GPUs                                       |            **Yes** |                 **No** |
| EC2 Spot Instances                         |            **Yes** |                     No |
| Maximum infrastructure control             |            **Yes** |                   Less |
| Operational overhead                       |             Higher |                  Lower |
| Good for interruptible container workloads | Yes, with EC2 Spot | Yes, with Fargate Spot |

---

# Fargate Spot

**Fargate Spot** provides discounted Fargate capacity for workloads that can tolerate interruption.

Good examples:

* Batch processing
* Background jobs
* Fault-tolerant workers
* Non-critical asynchronous workloads

Exam trigger:

> **"Cost optimize Fargate workloads that can tolerate interruptions"**

→ **Fargate Spot**

---

# ECS vs EKS

## ECS

**ECS = AWS-native container orchestration.**

Choose ECS when the question simply asks you to run containers on AWS and does not require Kubernetes-specific functionality.

ECS is generally simpler when you want an AWS-native container platform.

## EKS

**EKS = managed Kubernetes.**

Strong signals for EKS:

1. The company already uses **Kubernetes**
2. The workload requires **Kubernetes compatibility/portability**

Examples:

> "The company already runs Kubernetes on-premises and wants to migrate to AWS."

→ **EKS**

> "The company wants Kubernetes-based workloads that can be moved between cloud providers."

→ **EKS**

### Important

Do not choose EKS simply because it is more powerful.

For SAA questions, look for an explicit Kubernetes requirement.

```text
AWS-native containers
→ ECS

Already using Kubernetes
→ EKS

Kubernetes portability / compatibility
→ EKS
```

---

# ECR — Container Image Registry

**Amazon ECR (Elastic Container Registry)** stores container images.

Typical workflow:

```text
Developer
   ↓
Build Docker image
   ↓
Push image to ECR
   ↓
ECS / EKS pulls image
   ↓
Container starts
```

ECR is similar to a private container registry such as Docker Hub, but integrates with AWS IAM and other AWS services.

## ECR vulnerability scanning

ECR can scan container images for vulnerabilities.

There are two important scanning concepts:

* **Basic scanning** — detects vulnerabilities in supported OS packages
* **Enhanced scanning** — integrates with Amazon Inspector and also scans programming-language packages

Exam trigger:

> **"Scan container images for vulnerabilities"**

→ **ECR image scanning**

---

# ECS IAM roles

This is one of the most important ECS exam topics.

There are two roles you must distinguish:

| Role                    | Used by                          | Purpose                                                |
| ----------------------- | -------------------------------- | ------------------------------------------------------ |
| **Task Execution Role** | ECS/Fargate agent                | Infrastructure operations needed to start/run the task |
| **Task Role**           | Application inside the container | Permissions used by your application                   |

## Task Execution Role

The **Task Execution Role** gives ECS/Fargate permission to perform actions needed to run the task.

Common examples:

* Pull image from private ECR
* Send logs to CloudWatch Logs
* Retrieve certain secrets referenced by the task

Think:

> **"Can ECS get the container running?"**

### Typical question

> "The ECS task cannot pull its image from ECR."

→ **Task Execution Role**

> "The ECS task cannot send logs to CloudWatch."

→ **Task Execution Role**

AWS documents these as responsibilities of the task execution role.

---

# Task Role

The **Task Role** gives the application running inside the container permission to call AWS services.

Examples:

```text
Application
   ↓
S3
DynamoDB
SQS
SNS
Secrets Manager
```

### Typical question

> "The application inside the container receives AccessDenied when calling DynamoDB."

→ **Task Role**

> "The application needs permission to upload files to S3."

→ **Task Role**

AWS explicitly separates these application permissions from the task execution role.

## Easy memory rule

```text
Execution Role
→ ECS infrastructure / task startup

Task Role
→ Your application
```

---

# ECS networking

## awsvpc network mode

With **`awsvpc`**:

* Each task gets its own ENI
* Each task gets its own private IP address
* You can assign security groups directly to the task

```text
VPC
 |
 +-- Task 1
 |    └── ENI + private IP + Security Group
 |
 +-- Task 2
      └── ENI + private IP + Security Group
```

This is especially important for Fargate because `awsvpc` is the required networking model for Fargate tasks.

Exam trigger:

> **"Give each ECS task its own security group"**

→ **`awsvpc`**

> **"Each task should have its own ENI/private IP"**

→ **`awsvpc`**

AWS documents `awsvpc` as the mode that provides a separate ENI and allows security groups to be assigned at the task level.

---

# ALB and ECS

An ECS service can integrate with an **Application Load Balancer (ALB)**.

Typical architecture:

```text
Users
  ↓
ALB
  ↓
ECS Service
  ↓
Task 1
Task 2
Task 3
```

The ALB distributes traffic across healthy tasks.

## Dynamic port mapping

With ECS on EC2, dynamic port mapping can allow multiple copies of a service to run on the same EC2 instance while using different host ports.

For example:

```text
EC2 instance
 ├── Task 1 → host port 32768
 ├── Task 2 → host port 32769
 └── Task 3 → host port 32770
```

The ALB keeps track of the appropriate target port.

With `awsvpc`, each task has its own IP address, so host-port management is much less important because traffic can be sent directly to the task's IP and port.

---

# ECS Service Auto Scaling

ECS Service Auto Scaling changes the **number of running tasks**.

You can scale based on metrics such as:

* CPU utilization
* Memory utilization
* Application-specific CloudWatch metrics
* Queue workload

For queue workers:

```text
SQS queue grows
      ↓
More work waiting
      ↓
ECS increases task count
      ↓
More workers process messages
```

Exam trigger:

> **"Increase the number of ECS tasks when an SQS queue contains more work."**

→ **ECS Service Auto Scaling**

For SQS-based workloads, AWS recommends scaling using backlog-per-task rather than blindly using raw queue depth.

---

# EKS and Kubernetes

Kubernetes has its own terminology and architecture.

The important SAA-level distinction is:

```text
ECS
→ AWS container orchestration

EKS
→ Managed Kubernetes
```

Choose EKS when Kubernetes itself is part of the requirement.

Typical examples:

* Existing Kubernetes cluster
* Existing Kubernetes manifests/tools
* Kubernetes-based application platform
* Kubernetes portability requirements

You do not need to memorize the entire Kubernetes architecture for basic SAA questions.

---

# EKS Secrets encryption with AWS KMS

EKS stores Kubernetes API data in the managed Kubernetes control plane, with etcd used as the datastore.

For **Kubernetes 1.28 and later**, Amazon EKS provides **default envelope encryption for all Kubernetes API data**.

This includes Kubernetes resources such as:

* Secrets
* ConfigMaps
* Other Kubernetes API objects

EKS uses AWS KMS as part of this encryption architecture.

```text
Kubernetes API data
        ↓
Envelope encryption
        ↓
Kubernetes control plane / etcd
```

For clusters using a **customer-managed KMS key**, that key can provide customer-controlled encryption of Kubernetes API data.

### Exam takeaway

Older questions may specifically mention:

> **"Encrypt Kubernetes Secrets in EKS using AWS KMS."**

→ **KMS envelope encryption**

For modern EKS, remember that encryption of Kubernetes API data is already enabled by default for Kubernetes 1.28+; a customer-managed KMS key is an additional control rather than something required simply to obtain encryption.

---

# Compute decision ladder

Use the workload requirements rather than memorizing product names.

| Requirement                                        | Answer                |
| -------------------------------------------------- | --------------------- |
| Event-driven code, maximum **15 minutes**          | **Lambda**            |
| Containers, no server management                   | **Fargate**           |
| Containers + GPU                                   | **ECS on EC2**        |
| Containers + EC2-level control                     | **ECS on EC2**        |
| Kubernetes                                         | **EKS**               |
| Kubernetes portability / existing Kubernetes       | **EKS**               |
| "Just deploy my application" with managed platform | **Elastic Beanstalk** |
| Full operating-system control                      | **EC2**               |

---

# High-value question patterns

### 1. No server management

> "Run containers without managing servers."

**Answer: Fargate**

---

### 2. Existing Kubernetes

> "The company already runs Kubernetes on-premises and wants to migrate to AWS."

**Answer: EKS**

---

### 3. Kubernetes portability

> "The company wants Kubernetes workloads that can run across different cloud providers."

**Answer: EKS**

---

### 4. Task cannot pull image

> "An ECS task fails because it cannot pull the image from ECR."

**Answer: Task Execution Role**

---

### 5. Application gets AccessDenied

> "The application inside the ECS container receives AccessDenied when accessing DynamoDB."

**Answer: Task Role**

---

### 6. GPU workload

> "Run GPU-based ML inference in containers."

**Answer: ECS on EC2**

**Not Fargate.**

---

### 7. Container vulnerability scanning

> "Automatically scan container images for vulnerabilities."

**Answer: ECR image scanning**

---

### 8. Per-task security groups

> "Each ECS task needs its own security group."

**Answer: `awsvpc`**

---

### 9. Cheap interruptible containers

> "Run fault-tolerant containerized batch processing at lower cost."

**Answer: Fargate Spot**

---

### 10. Scale workers from SQS

> "Increase the number of container workers as the SQS workload increases."

**Answer: ECS Service Auto Scaling**

---

# Pocket card

| Keyword                                                    | Answer                            |
| ---------------------------------------------------------- | --------------------------------- |
| **No server management + containers**                      | Fargate                           |
| **Already using Kubernetes**                               | EKS                               |
| **Kubernetes portability**                                 | EKS                               |
| **GPU containers**                                         | ECS on EC2                        |
| **Can't pull image**                                       | Task Execution Role               |
| **Can't write ECS logs**                                   | Task Execution Role               |
| **Application AccessDenied to AWS service**                | Task Role                         |
| **Recipe / configuration**                                 | Task Definition                   |
| **Running copy**                                           | Task                              |
| **Keeps desired number of tasks running**                  | Service                           |
| **Container image storage**                                | ECR                               |
| **Image vulnerability scanning**                           | ECR                               |
| **Per-task ENI / security group**                          | `awsvpc`                          |
| **Cheap interruptible Fargate**                            | Fargate Spot                      |
| **Scale tasks based on workload**                          | ECS Service Auto Scaling          |
| **Kubernetes API-data encryption**                         | EKS envelope encryption / AWS KMS |
| **Multiple tasks on one EC2 host with dynamic host ports** | ALB + dynamic port mapping        |

---

# The most important distinctions

```text
ECS vs EKS
──────────
ECS = AWS-native
EKS = Kubernetes


ECS EC2 vs Fargate
──────────────────
EC2 = manage instances
Fargate = no instance management


Execution Role vs Task Role
────────────────────────────
Execution Role = ECS/Fargate can run the task
Task Role      = application can access AWS services


Task Definition vs Task
───────────────────────
Task Definition = configuration
Task             = running instance


Service vs Task
───────────────
Task     = one running workload
Service  = maintains the desired number of tasks


ECR vs ECS
──────────
ECR = stores container images
ECS = runs and manages containers
```

The key idea is simple:

**ECS/EKS decide how containers are orchestrated.
EC2/Fargate provide the compute.
ECR stores the images.
IAM roles control what ECS and the application can access.**

Next: **Elastic Beanstalk** — a higher-level option where you deploy application code without directly managing containers or the underlying infrastructure.
