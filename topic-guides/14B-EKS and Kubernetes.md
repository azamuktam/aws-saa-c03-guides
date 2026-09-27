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

# EKS Secrets encryption with AWS KMS

EKS stores Kubernetes API data in the managed Kubernetes control plane, with **etcd** used as the datastore.

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

### 5. Kubernetes API-data encryption

> "The company wants to encrypt Kubernetes API data using AWS KMS."

**Answer: EKS envelope encryption / AWS KMS**

---

# Pocket card

| Keyword                                  | Answer                       |
| ---------------------------------------- | ---------------------------- |
| **Managed Kubernetes**                   | EKS                          |
| **Already using Kubernetes**             | EKS                          |
| **Kubernetes portability**               | EKS                          |
| **Kubernetes manifests/tools**           | EKS                          |
| **Kubernetes + serverless compute**      | EKS + Fargate                |
| **Kubernetes + EC2 control**             | EKS on EC2                   |
| **Encrypt Kubernetes API data**          | EKS envelope encryption      |
| **Customer-controlled encryption key**   | Customer-managed AWS KMS key |
| **Kubernetes datastore**                 | etcd                         |
| **Kubernetes 1.28+ API-data encryption** | Enabled by default           |

---

# The most important distinction

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


KMS encryption
──────────────
EKS can use AWS KMS for envelope encryption
of Kubernetes API data.
```

The key idea is simple:

**EKS provides managed Kubernetes.
EC2 or Fargate provides the compute for Kubernetes workloads.
AWS KMS can be used for envelope encryption of Kubernetes API data.**
