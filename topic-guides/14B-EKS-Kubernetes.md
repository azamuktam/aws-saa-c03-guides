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

EKS can run supported Kubernetes pods using Fargate.

AWS manages the underlying compute infrastructure.

The key distinction is:

```text
EKS = Kubernetes orchestration

Fargate = serverless container compute
```

Fargate can therefore be used with EKS as well as ECS.

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

Suppose a Kubernetes pod needs to access:

* S3
* DynamoDB
* SQS
* Secrets Manager

Do not confuse this with human access to the EKS cluster.

The pod needs an IAM role:

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

It allows a pod to use an IAM role without giving that role to the entire worker node.

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

> "The company wants to run Kubernetes pods without managing the underlying servers."

**Answer: EKS with Fargate**

---

### 5. IAM user/role needs cluster access

> "A developer needs to use `kubectl` against an EKS cluster."

**Answer: EKS Access Entry**

Older questions may use:

**`aws-auth` ConfigMap**

---

### 6. Pod needs AWS service access

> "A Kubernetes pod needs permission to read from S3."

**Answer: EKS Pod Identity**

Know **IRSA** as the older/alternative approach.

---

### 7. Kubernetes RBAC

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

### 8. Kubernetes API-data encryption

> "The company wants to encrypt Kubernetes API data using AWS KMS."

**Answer: EKS envelope encryption**

---

# Pocket card

| Keyword                                    | Answer                           |
| ------------------------------------------ | -------------------------------- |
| **Managed Kubernetes**                     | EKS                              |
| **Already using Kubernetes**               | EKS                              |
| **Kubernetes portability**                 | EKS                              |
| **Kubernetes manifests/tools**             | EKS                              |
| **Kubernetes + serverless compute**        | EKS + Fargate                    |
| **Kubernetes + EC2 control**               | EKS on EC2                       |
| **IAM user/role → EKS cluster**            | EKS Access Entry                 |
| **Older IAM → EKS mechanism**              | `aws-auth` ConfigMap             |
| **Pod → AWS services**                     | EKS Pod Identity                 |
| **Older pod IAM mechanism**                | IRSA                             |
| **Namespace-level Kubernetes permissions** | Role + RoleBinding               |
| **Cluster-wide Kubernetes permissions**    | ClusterRole + ClusterRoleBinding |
| **Encrypt Kubernetes API data**            | EKS envelope encryption          |
| **Customer-controlled encryption key**     | Customer-managed AWS KMS key     |
| **Kubernetes datastore**                   | etcd                             |
| **Kubernetes 1.28+ API-data encryption**   | Enabled by default               |

---

# The most important distinctions

```text
ECS vs EKS
──────────
ECS = AWS-native container orchestration
EKS = Managed Kubernetes


EKS vs Fargate
──────────────
EKS = orchestration / Kubernetes
Fargate = compute


EKS on EC2 vs EKS on Fargate
─────────────────────────────
EC2     = manage worker infrastructure
Fargate = AWS manages underlying infrastructure


IAM user/role → EKS
───────────────────
Access Entry = modern method
aws-auth     = legacy method


Pod → AWS services
──────────────────
EKS Pod Identity = modern method
IRSA             = older/alternative method


Kubernetes RBAC
───────────────
Role + RoleBinding
→ namespace permissions

ClusterRole + ClusterRoleBinding
→ cluster-wide permissions


KMS encryption
──────────────
EKS 1.28+
→ all Kubernetes API data encrypted by default
```

The key idea is simple:

**EKS provides managed Kubernetes.
EC2 or Fargate provides the compute for Kubernetes workloads.
IAM users/roles access the EKS cluster through EKS access management.
Kubernetes RBAC controls what those identities can do.
Pods use EKS Pod Identity or IRSA to access AWS services.
AWS KMS provides the encryption layer for Kubernetes API data.**
