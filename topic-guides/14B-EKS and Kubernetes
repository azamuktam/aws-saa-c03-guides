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
