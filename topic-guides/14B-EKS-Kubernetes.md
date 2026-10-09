# Section 14: Amazon EKS

## The idea

**Amazon EKS (Elastic Kubernetes Service)** is AWS's managed Kubernetes service.

Kubernetes is a container orchestration platform that manages containerized applications.

EKS lets you run Kubernetes workloads on AWS while AWS manages the Kubernetes control plane.

```text
                    Amazon EKS
                       |
             Managed Kubernetes
                       |
          -------------------------
          |                       |
       EC2 nodes              Fargate
     You manage hosts       AWS manages hosts
```

---

# EKS → Nodes → Pods → Containers → Application

This is the basic Kubernetes structure you should understand.

```text
EKS Cluster
     ↓
   Nodes
     ↓
    Pods
     ↓
 Containers
     ↓
Application
```

## Node

A **Node** is compute capacity where Kubernetes runs Pods.

With **EKS on EC2**, a node is usually an **EC2 instance**.

```text
EKS
 ↓
EC2 Node
 ↓
Pods
```

The node provides CPU, memory, networking, and other resources needed by the Pods.

Think:

> **Node = the machine that runs Pods**

---

## Pod

A **Pod** is the **smallest deployable unit in Kubernetes**.

A Pod contains one or more containers that are meant to run together.

```text
Node
 |
 +-- Pod 1
 |    └── Container
 |
 +-- Pod 2
      └── Container
```

Most simple applications use **one main container per Pod**.

Think:

> **Pod = the Kubernetes wrapper around your container(s)**

A Pod is **not** the same thing as a container.

```text
Pod
 ↓
Container
 ↓
Application
```

---

## Container

A **container** runs the actual application process.

For example:

```text
Pod
 ↓
Container
 ↓
Nginx
```

Or:

```text
Pod
 ↓
Container
 ↓
PHP application
```

The container is created from a **container image**, typically stored in a registry such as Amazon ECR.

---

## Application

Your actual application code runs inside the container.

For example:

```text
EKS Cluster
    ↓
EC2 Node
    ↓
Pod
    ↓
Container
    ↓
Laravel application
```

### Easy mental model

```text
Node
= machine

Pod
= Kubernetes unit running on the machine

Container
= process environment running inside the Pod

Application
= your actual software
```

---

## Important Fargate difference

With **EKS on Fargate**, you don't manage the underlying EC2 nodes.

You still think in terms of:

```text
EKS
 ↓
Pod
 ↓
Container
 ↓
Application
```

But AWS manages the underlying compute.

So:

```text
EKS on EC2
→ You manage Nodes

EKS on Fargate
→ AWS manages the underlying compute
```

---

### When should you choose EKS?

Strong signals for EKS:

1. The company already uses **Kubernetes**
2. The workload requires **Kubernetes compatibility**
3. The company wants Kubernetes tooling, APIs, or manifests
4. The company needs Kubernetes portability

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

# EKS compute options

EKS provides the Kubernetes control plane, but Kubernetes workloads still need compute capacity.

The main options relevant to SAA are:

| Option             | Who manages the underlying compute?       |
| ------------------ | ----------------------------------------- |
| **EKS on EC2**     | You manage the EC2 worker nodes           |
| **EKS on Fargate** | AWS manages the underlying infrastructure |

### EKS on EC2

Use EC2 worker nodes when you need more control over the compute environment.

You can choose:

* EC2 instance types
* GPUs
* Capacity configuration
* EC2 Spot Instances
* Operating-system configuration

### EKS on Fargate

EKS can run supported Kubernetes Pods using Fargate.

AWS manages the underlying compute infrastructure.

The key distinction is:

```text
EKS = Kubernetes orchestration

Fargate = serverless container compute
```

Fargate can therefore be used with EKS as well as ECS.

---

# AWS Load Balancer Controller (EKS ingress)

This is how you expose applications running in EKS to the internet or to other services.

The **AWS Load Balancer Controller** is the modern, AWS-recommended way to route traffic into an EKS cluster.

It is a **controller (pod) that runs inside the EKS cluster** and watches Kubernetes resources. When it sees them, it **calls AWS APIs to automatically provision load balancers** for you.

```text
Kubernetes / EKS world            AWS world
─────────────────────            ─────────
Ingress resource        ──►      ALB
Service (type LoadBalancer) ─►   NLB
        ▲
        │
AWS Load Balancer Controller
(runs as pods INSIDE EKS)
```

### Key idea

> The ALB/NLB is **not inside the cluster** — it lives in your VPC. The controller **provisions and manages it automatically** based on Kubernetes resources.

---

## What it creates

| Kubernetes resource              | AWS resource created |
| -------------------------------- | -------------------- |
| **Ingress**                      | **ALB**              |
| **Service (type LoadBalancer)**  | **NLB**              |

So the controller is the **bridge**:

```text
Ingress       → ALB
Service       → NLB
```

---

## Why it matters: path-based routing

An **Application Load Balancer (ALB)** supports **HTTP/HTTPS path-based routing** natively.

```text
Internet
   │
   ▼
┌─────────────────┐
│      ALB        │  ← provisioned in your VPC by the controller
└─────────────────┘
   │        │
 /orders   /users
   │        │
   ▼        ▼
[order pods] [user pods]
```

Example Ingress:

```yaml
rules:
- http:
    paths:
    - path: /orders
      backend: order-service
    - path: /users
      backend: user-service
```

The controller reads this and **creates the ALB, listeners, target groups, and routing rules** for you.

---

## Why not the alternatives (least setup)

| Approach                          | Setup effort |
| --------------------------------- | ------------ |
| **ALB + AWS Load Balancer Controller** | **Least** — install controller once, write Ingress YAML, ALB auto-created |
| NLB + AWS Load Balancer Controller | NLB is Layer 4, no native path routing |
| NGINX Ingress controller          | You deploy/manage NGINX pods + a LB |
| Lambda proxy                      | Custom proxy code to maintain |

### Key takeaway

> For **HTTP path-based routing into EKS with least setup**, use an **ALB provisioned by the AWS Load Balancer Controller**.

---

## Exam triggers

> "Route requests to EKS services based on URL paths with least setup."

→ **ALB via the AWS Load Balancer Controller**

> "Automatically create an ALB from a Kubernetes Ingress."

→ **AWS Load Balancer Controller**

> "Expose a Kubernetes Service with an NLB."

→ **AWS Load Balancer Controller**

### Easy memory

```text
Ingress  → ALB  → path-based HTTP routing
Service  → NLB  → TCP/UDP load balancing
Controller = pod in EKS that creates them
```

---

## Topic note

The **AWS Load Balancer Controller is an EKS topic** (EKS ingress), even though the ALB it creates belongs to the Elastic Load Balancing world.

```text
Pure ALB question (no Kubernetes)
→ ELB topic

Ingress / EKS / controller creating ALB
→ EKS ingress topic
```

---

# EKS and Kubernetes terminology

You do not need to memorize the entire Kubernetes architecture for basic SAA questions.

The important SAA-level distinction is:

```text
ECS
→ AWS container orchestration

EKS
→ Managed Kubernetes
```

Typical EKS signals:

* Existing Kubernetes cluster
* Existing Kubernetes manifests
* Existing Kubernetes tools
* Kubernetes-based application platform
* Kubernetes portability requirements

---

# EKS Autoscaling

EKS can scale at different levels.

The most important distinction is:

```text
HPA
→ scales Pods

VPA
→ changes Pod resource requests

Karpenter / Cluster Autoscaler
→ scales Nodes
```

## Scaling differences

| Component                     | What it does                                      | What changes?                      | Typical signal                                         |
| ----------------------------- | ------------------------------------------------- | ---------------------------------- | ------------------------------------------------------ |
| **Kubernetes Metrics Server** | Provides CPU/memory metrics                       | Nothing                            | "Measure Pod/node CPU or memory"                       |
| **HPA**                       | Adds/removes Pods                                 | **Number of Pods**                 | "Scale Pods based on CPU/request load"                 |
| **VPA**                       | Adjusts Pod resource requests                     | **CPU/memory per Pod**             | "Give Pods more/less CPU or memory"                    |
| **Cluster Autoscaler**        | Adjusts existing node groups                      | **Number of worker nodes**         | "Pods cannot be scheduled because nodes lack capacity" |
| **Karpenter**                 | Dynamically provisions/consolidates node capacity | **Number + size/type of nodes** | "Automatically provision flexible/right-sized nodes"   |

### Easy memory

```text
Metrics Server
→ measures

HPA
→ more/fewer Pods

VPA
→ bigger/smaller Pods

Cluster Autoscaler
→ more/fewer Nodes

Karpenter
→ node count + node type/size
```

---

## Horizontal Pod Autoscaler (HPA)

**HPA increases or decreases the number of Pods** based on workload demand.

For example:

```text
Traffic increases
      ↓
More CPU usage
      ↓
HPA
      ↓
More Pods
```

HPA commonly uses CPU or memory metrics provided by the **Kubernetes Metrics Server**.

Example:

```text
Before:
Node
 ├── Pod
 └── Pod

After traffic increases:
Node
 ├── Pod
 ├── Pod
 ├── Pod
 └── Pod
```

### Exam trigger

> **"Automatically increase the number of Pods based on CPU utilization."**

→ **HPA**

---

## Vertical Pod Autoscaler (VPA)

**VPA changes the resource requests of Pods**, such as CPU and memory.

Think:

```text
HPA
→ more Pods

VPA
→ bigger/smaller Pods
```

### Exam trigger

> **"Automatically adjust CPU and memory resources assigned to Pods."**

→ **VPA**

---

## Karpenter

**Karpenter scales the underlying EC2 nodes** when Pods need more compute capacity.

For example:

```text
Traffic increases
      ↓
HPA creates more Pods
      ↓
Not enough node capacity
      ↓
Karpenter
      ↓
New EC2 nodes
      ↓
Pods are scheduled
```

When demand decreases, Karpenter can remove or consolidate unnecessary capacity.

### Exam trigger

> **"Automatically provision EC2 capacity when Pods cannot be scheduled."**

→ **Karpenter**

---

## Cluster Autoscaler

**Cluster Autoscaler also scales the number of worker nodes**, typically by changing the size of existing node groups.

```text
Pods need more capacity
      ↓
Cluster Autoscaler
      ↓
Increase node group
      ↓
More EC2 nodes
```

It can also scale down nodes when they are no longer needed.

### Karpenter vs Cluster Autoscaler

For SAA, remember:

```text
Karpenter
→ dynamically provisions node capacity

Cluster Autoscaler
→ adjusts existing node groups
```

When a question emphasizes **least operational overhead and flexible node provisioning**, Karpenter is often the intended choice.

---

## The complete autoscaling picture

```text
              Application demand
                     ↓
                    HPA
                     ↓
                More Pods
                     ↓
          Not enough node capacity
                     ↓
                 Karpenter
                     ↓
                 More Nodes
```

This is the key relationship:

> **HPA scales the application layer. Karpenter scales the infrastructure layer.**

---

# EKS IAM authentication and Kubernetes RBAC

There are **two different access problems** to recognize.

## 1. IAM user/role → EKS cluster

This is about a person or AWS IAM principal accessing the Kubernetes API.

Modern EKS uses **Access Entries** to connect IAM users or roles to Kubernetes access.

```text
IAM user / role
       ↓
EKS Access Entry
       ↓
Kubernetes access
       ↓
RBAC permissions
```

For custom Kubernetes RBAC permissions, an Access Entry can associate the IAM principal with a Kubernetes group.

Then Kubernetes RBAC can use:

* **Role** — permissions within a namespace
* **ClusterRole** — cluster-wide permissions
* **RoleBinding** — attaches permissions to a user/group in a namespace
* **ClusterRoleBinding** — attaches permissions cluster-wide

### Important exam distinction

> **"Give an IAM role/user access to an EKS cluster."**

→ **EKS Access Entry**

Older questions may mention the **`aws-auth` ConfigMap** instead.

```text
IAM user / role
       ↓
aws-auth ConfigMap
       ↓
Kubernetes RBAC
```

Know `aws-auth` because it appears in older exam material, but it is **deprecated** in modern EKS.

---

## 2. Pod → AWS services

This is a different problem.

Suppose a Kubernetes Pod needs to access:

* S3
* DynamoDB
* SQS
* Secrets Manager

Do not confuse this with human access to the EKS cluster.

The Pod needs an IAM role:

```text
Pod
 ↓
Kubernetes Service Account
 ↓
EKS Pod Identity
 ↓
IAM Role
 ↓
AWS service
```

### EKS Pod Identity

**EKS Pod Identity** is the modern way to give Kubernetes workloads IAM permissions.

It allows a Pod to use an IAM role without giving that role to the entire worker node.

### IRSA

**IAM Roles for Service Accounts (IRSA)** is the older/alternative mechanism.

Conceptually:

```text
Pod
 ↓
Service Account
 ↓
OIDC
 ↓
IAM Role
 ↓
AWS service
```

### Easy rule

```text
IAM user/role → EKS cluster
→ Access Entry
→ Kubernetes RBAC

Pod → AWS service
→ EKS Pod Identity / IRSA
→ IAM Role
```

This distinction is important for SAA questions.

---

# EKS Secrets encryption with AWS KMS

EKS stores Kubernetes API data in the managed Kubernetes control plane, with **etcd** used as the datastore.

For **Kubernetes 1.28 and later**, Amazon EKS provides **default envelope encryption for all Kubernetes API data**.

This includes Kubernetes resources such as:

* Secrets
* ConfigMaps
* Other Kubernetes API objects

```text
Kubernetes API data
        ↓
Envelope encryption
        ↓
Kubernetes control plane / etcd
```

EKS uses AWS KMS as part of this encryption architecture.

By default, EKS uses an **AWS-owned KMS key**. You can optionally use your own **customer-managed KMS key** for customer-controlled encryption.

### Exam takeaway

Older questions may specifically mention:

> **"Encrypt Kubernetes Secrets in EKS using AWS KMS."**

→ **KMS envelope encryption**

For modern EKS, remember:

```text
Kubernetes 1.28+
→ All Kubernetes API data is encrypted by default
```

You do not need to configure KMS just to obtain the default envelope encryption. A customer-managed KMS key is an additional option when you need control over the key.

---

# EKS vs ECS

| Requirement                          | Answer            |
| ------------------------------------ | ----------------- |
| AWS-native container orchestration   | **ECS**           |
| Existing Kubernetes environment      | **EKS**           |
| Kubernetes manifests/tools           | **EKS**           |
| Kubernetes portability               | **EKS**           |
| Containers without server management | **Fargate**       |
| Kubernetes + serverless compute      | **EKS + Fargate** |
| Kubernetes + EC2-level control       | **EKS on EC2**    |

The key question is:

> **Does the scenario actually require Kubernetes?**

If yes, think **EKS**.

If not, ECS is the AWS-native container orchestration option.

---

# High-value question patterns

### 1. Existing Kubernetes

> "The company already runs Kubernetes on-premises and wants to migrate to AWS."

**Answer: EKS**

---

### 2. Kubernetes portability

> "The company wants Kubernetes workloads that can run across different cloud providers."

**Answer: EKS**

---

### 3. Existing Kubernetes manifests

> "The company has existing Kubernetes deployment manifests and wants to move the workloads to AWS."

**Answer: EKS**

---

### 4. Kubernetes + no server management

> "The company wants to run Kubernetes Pods without managing the underlying servers."

**Answer: EKS with Fargate**

---

### 5. Route traffic into EKS by URL path

> "Route requests to EKS services based on URL paths with the least setup."

**Answer: ALB via the AWS Load Balancer Controller**

---

### 6. Automatically create a load balancer from Kubernetes

> "Automatically create an ALB from a Kubernetes Ingress resource."

**Answer: AWS Load Balancer Controller**

---

### 7. Scale Pods based on demand

> "The application needs more Pods when traffic or CPU utilization increases."

**Answer: HPA**

---

### 8. Scale EKS infrastructure

> "Pods cannot be scheduled because there is not enough node capacity."

**Answer: Karpenter / Cluster Autoscaler**

For least operational overhead and flexible node provisioning:

**Karpenter**

---

### 9. IAM user/role needs cluster access

> "A developer needs to use `kubectl` against an EKS cluster."

**Answer: EKS Access Entry**

Older questions may use:

**`aws-auth` ConfigMap**

---

### 10. Pod needs AWS service access

> "A Kubernetes Pod needs permission to read from S3."

**Answer: EKS Pod Identity**

Know **IRSA** as the older/alternative approach.

---

### 11. Kubernetes RBAC

> "An IAM role should only be allowed to read Pods in a specific namespace."

Think:

**EKS access + Kubernetes RBAC**

Typically:

```text
IAM Role
   ↓
EKS Access Entry
   ↓
Kubernetes group
   ↓
Role
   ↓
RoleBinding
```

---

### 12. Kubernetes API-data encryption

> "The company wants to encrypt Kubernetes API data using AWS KMS."

**Answer: EKS envelope encryption**

---

# Pocket card

| Keyword                                    | Answer                              |
| ------------------------------------------ | ----------------------------------- |
| **Managed Kubernetes**                     | EKS                                 |
| **Node**                                   | Compute capacity that runs Pods     |
| **Pod**                                    | Smallest deployable Kubernetes unit |
| **Container**                              | Runs the application process        |
| **Already using Kubernetes**               | EKS                                 |
| **Kubernetes portability**                 | EKS                                 |
| **Kubernetes manifests/tools**             | EKS                                 |
| **Kubernetes + serverless compute**        | EKS + Fargate                       |
| **Kubernetes + EC2 control**               | EKS on EC2                          |
| **Ingress → ALB**                          | AWS Load Balancer Controller        |
| **Service → NLB**                          | AWS Load Balancer Controller        |
| **Path-based routing into EKS**            | ALB via AWS Load Balancer Controller|
| **Controller that creates load balancers** | AWS Load Balancer Controller        |
| **Metrics Server**                         | Provides CPU/memory metrics         |
| **More Pods**                              | HPA                                 |
| **Change Pod CPU/memory**                  | VPA                                 |
| **More/fewer Nodes**                       | Karpenter / Cluster Autoscaler      |
| **IAM user/role → EKS cluster**            | EKS Access Entry                    |
| **Older IAM → EKS mechanism**              | `aws-auth` ConfigMap                |
| **Pod → AWS services**                     | EKS Pod Identity                    |
| **Older pod IAM mechanism**                | IRSA                                |
| **Namespace-level Kubernetes permissions** | Role + RoleBinding                  |
| **Cluster-wide Kubernetes permissions**    | ClusterRole + ClusterRoleBinding    |
| **Encrypt Kubernetes API data**            | EKS envelope encryption             |
| **Customer-controlled encryption key**     | Customer-managed AWS KMS key        |
| **Kubernetes datastore**                   | etcd                                |
| **Kubernetes 1.28+ API-data encryption**   | Enabled by default                  |
