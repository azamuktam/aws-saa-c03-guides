# Section 5: Elastic Load Balancing & Auto Scaling

## The idea

**Elastic Load Balancing (ELB)** puts one entry point in front of multiple servers and distributes incoming traffic between them.

```text
Clients
   ↓
Load Balancer
   ↓
EC2 instances
```

The load balancer also performs **health checks**.

If an EC2 instance is unhealthy:

```text
Healthy instance   → receives traffic ✅
Unhealthy instance → stops receiving traffic ❌
```

**Auto Scaling Group (ASG)** manages the number of EC2 instances.

```text
High demand
   ↓
ASG adds instances

Low demand
   ↓
ASG removes instances
```

If an instance crashes, the ASG can automatically launch a replacement.

### Together

```text
Clients
   ↓
Load Balancer
   ↓
Auto Scaling Group
   ↓
EC2 instances
```

This gives you:

* traffic distribution
* health checks
* automatic scaling
* automatic replacement of failed instances

---

## OSI layers in 60 seconds

For load balancers, remember:

| Layer       | What it sees        | Example                 |
| ----------- | ------------------- | ----------------------- |
| **Layer 3** | IP addresses        | `10.0.1.10`             |
| **Layer 4** | IP + port + TCP/UDP | TCP 443, UDP 5000       |
| **Layer 7** | HTTP information    | URL path, host, headers |

### Why does this matter?

A Layer 4 load balancer cannot understand:

```text
/api/orders
/images/logo.png
```

because those are **HTTP-level details**.

A Layer 7 load balancer can.

So:

```text
Need URL/path/host routing
→ Layer 7 → ALB

Need TCP/UDP + very high performance
→ Layer 4 → NLB
```

---

## The four load balancers

|          | Layer | Protocols           | Main feature                              | Exam keyword                        |
| -------- | ----- | ------------------- | ----------------------------------------- | ----------------------------------- |
| **ALB**  | 7     | HTTP, HTTPS, gRPC   | Content-based routing                     | Path, host, header, web apps        |
| **NLB**  | 4     | TCP, UDP, TLS       | High performance + static IP              | UDP, millions of requests, fixed IP |
| **GWLB** | 3     | IP packets / GENEVE | Sends traffic through security appliances | Firewall, IDS, IPS                  |
| **CLB**  | 4/7   | Legacy              | Older load balancer                       | Usually wrong answer                |

---

## ALB — Application Load Balancer

ALB works at **Layer 7**, so it understands HTTP/HTTPS traffic.

This allows routing based on:

* URL path
* hostname
* HTTP headers
* query strings

### Examples

```text
/api/*       → API target group

/images/*    → Image target group
```

or:

```text
api.example.com
        ↓
API servers

admin.example.com
        ↓
Admin servers
```

### Important facts

* ALB target groups can contain:

  * EC2 instances
  * private IP addresses
  * Lambda functions
  * containers
* ALB performs health checks on targets.
* ALB can integrate with **AWS WAF**.
* ALB has a **DNS name**, not a fixed static IP.

### Client IP behind ALB

The target normally sees the ALB connection.

The original client IP is normally available in:

```text
X-Forwarded-For
```

So:

> **"Application behind ALB needs the original client IP."**

→ **X-Forwarded-For**

### Sticky sessions

Sticky sessions can keep a client connected to the same target.

Example:

```text
User A
  ↓
ALB
  ↓
EC2-A
```

The ALB can continue sending that user to EC2-A.

Use this when the application keeps session state locally on the instance.

A better architecture is often to store session state externally, for example in ElastiCache or DynamoDB.

### ECS dynamic port mapping

If several containers run on the same EC2 instance, each container can use a different port.

ALB can discover and route to those ports.

This is useful with ECS.

### Fixed IP requirement

ALB does **not** provide a fixed static IP.

If the requirement is:

> "Customers must whitelist fixed IP addresses."

Think:

→ **NLB**

or:

→ **Global Accelerator**

---

## NLB — Network Load Balancer

NLB works at **Layer 4**.

It mainly looks at:

* IP address
* port
* TCP/UDP/TLS connection information

It does not use HTTP URL paths like ALB.

### Main characteristics

* Very high throughput
* Very low latency
* Handles very large numbers of connections
* Supports **TCP**
* Supports **UDP**
* Supports **TLS**
* Provides a **static IP per Availability Zone**
* Can use **Elastic IP addresses**
* Preserves the client source IP by default

### Use NLB for

* UDP applications
* gaming
* IoT
* custom TCP protocols
* extremely high traffic
* fixed IP requirements

### Example

> "A gaming application uses UDP and requires a fixed IP."

→ **NLB**

---

## GWLB — Gateway Load Balancer

Gateway Load Balancer is used to send network traffic through **security appliances**.

Typical appliances:

* firewalls
* IDS
* IPS
* traffic inspection systems

```text
Traffic
   ↓
GWLB
   ↓
Security appliance
   ↓
Application
```

GWLB uses **GENEVE on port 6081**.

### Exam rule

> **Third-party firewall/IDS/IPS → GWLB**

You are not using GWLB to distribute normal web traffic like ALB.

---

## CLB — Classic Load Balancer

Classic Load Balancer is the **older generation**.

For modern architectures, prefer:

* **ALB**
* **NLB**
* **GWLB**

If CLB appears as a distractor in a modern architecture question, it is usually not the answer.

---

## Shared ELB features worth points

### Cross-zone load balancing

Without cross-zone load balancing, a load balancer node normally sends traffic to targets in its own AZ.

Example:

```text
AZ-A
2 instances

AZ-B
8 instances
```

Without cross-zone balancing, traffic can be uneven.

With cross-zone balancing, traffic can be distributed across targets in other AZs.

### Important defaults

* **ALB:** cross-zone load balancing is enabled by default.
* **NLB:** cross-zone load balancing is disabled by default at the load balancer level.

### Exam pattern

> **"Traffic is unevenly distributed between Availability Zones."**

→ Check **cross-zone load balancing**.

---

### Deregistration delay / connection draining

When an instance is being removed, the load balancer should not immediately kill existing connections.

Instead:

```text
New requests
→ stop sending to target

Existing requests
→ allowed to finish
```

This is called:

* **deregistration delay**
* **connection draining**

Use it when:

> **"Users receive errors when instances are removed during scale-in."**

---

### SNI

**SNI (Server Name Indication)** allows one HTTPS listener to use multiple certificates.

Example:

```text
example.com
api.example.com
admin.example.com
```

One ALB can serve different certificates based on the requested hostname.

### Exam pattern

> **"Host multiple HTTPS domains with different certificates on one load balancer."**

→ **SNI**

---

### TLS termination

The load balancer can terminate HTTPS.

```text
Client
  ↓ HTTPS
Load Balancer
  ↓ HTTP or HTTPS
EC2
```

The certificate is usually stored on the load balancer through **AWS Certificate Manager (ACM)**.

This removes TLS processing from the EC2 instances.

---

# Auto Scaling Groups

An **Auto Scaling Group (ASG)** manages a group of EC2 instances.

You define:

* minimum number of instances
* desired number
* maximum number
* how new instances should be launched

```text
Launch Template
      ↓
     ASG
      ↓
EC2 EC2 EC2 EC2
```

### Launch Template

A Launch Template contains the configuration for new instances, such as:

* AMI
* instance type
* security group
* user data
* key pair
* other launch settings

**Launch Configurations** are the older/legacy mechanism.

Use:

> **Launch Template**

for modern designs.

### Min / Desired / Max

Example:

```text
min     = 2
desired = 4
max     = 10
```

ASG tries to maintain the desired number of instances.

---

## ASG self-healing

If an EC2 instance fails:

```text
EC2 instance fails
       ↓
ASG detects it
       ↓
Instance terminated
       ↓
Replacement launched
```

This is one of the main purposes of an ASG.

---

## Scaling policies

Choose the scaling policy based on the requirement.

| Policy                 | What it does                                         | Exam keyword                       |
| ---------------------- | ---------------------------------------------------- | ---------------------------------- |
| **Target Tracking**    | Keeps a metric around a target value                 | "Keep CPU around 40%"              |
| **Step Scaling**       | Different scaling amounts for different alarm levels | "If CPU > 70%, add 2; >90%, add 4" |
| **Simple Scaling**     | Fixed adjustment after an alarm                      | Basic scaling                      |
| **Scheduled Scaling**  | Scales at known times                                | "Every Monday 9 AM"                |
| **Predictive Scaling** | Uses ML to predict future demand                     | "Scale before expected traffic"    |

### Target Tracking

Example:

> Keep average CPU at **40%**.

→ **Target Tracking**

This is usually the simplest choice when the question asks to maintain a specific metric.

### Step Scaling

Example:

```text
CPU > 60% → add 1
CPU > 80% → add 2
CPU > 90% → add 4
```

→ **Step Scaling**

### Scheduled Scaling

Use when demand is predictable.

Example:

> Traffic increases every weekday at 9 AM.

→ **Scheduled Scaling**

### Predictive Scaling

Use when traffic follows patterns that can be predicted.

It uses machine learning to anticipate demand and scale **before** the traffic arrives.

---

## Warm-up and cooldown

After launching or terminating instances, ASG may need time before evaluating the system again.

This prevents repeated scaling actions while a new instance is still starting.

### Exam pattern

> **"ASG keeps launching more instances before the previous instances are fully ready."**

→ Check **instance warm-up / cooldown settings**.

---

## THE health-check trap

By default, an ASG uses **EC2 health checks**.

That means it checks whether the EC2 instance itself is healthy.

But this can happen:

```text
EC2 = running ✅
Application = broken ❌
```

The ASG may still consider the instance healthy.

### ELB health checks

You can configure the ASG to use **ELB health checks**.

Then:

```text
ALB
 ↓
Application health check
 ↓
Unhealthy instance
 ↓
ASG replaces instance
```

### Exam pattern

> **"The load balancer marks instances unhealthy, but the ASG does not replace them."**

→ **Enable ELB health checks on the ASG**

---

## ASG + SQS pattern

Suppose EC2 workers process jobs from SQS.

```text
Application
    ↓
   SQS
    ↓
ASG workers
```

If the queue grows, you need more workers.

So the ASG can scale based on the queue.

A useful metric is:

> **Backlog per instance**

This is better than simply looking at CPU when the real problem is the number of pending jobs.

### Exam pattern

> **"Scale workers according to the number of unprocessed jobs."**

→ **Scale the ASG using SQS queue metrics**

---

## Termination policy — who is terminated first?

When an ASG needs to scale in, it has to decide **which instance to remove**.

With the default termination policy, the ASG first tries to keep the Availability Zones balanced.

Then it prefers instances using **older launch configurations or launch template versions/configurations**.

The important exam idea is:

> **Default scale-in prefers removing older configurations while maintaining AZ balance.**

This is useful when you have recently updated the Launch Template and want old instances to disappear gradually.

### Important trap

Do not confuse:

> **Oldest configuration**

with:

> **Oldest running instance**

The **`OldestInstance` termination policy** specifically prefers the instance that has been running the longest.

So:

```text
Default policy
→ keep AZs balanced + prefer older configuration

OldestInstance policy
→ terminate the oldest running instance
```

---

## The layered HA picture

Different AWS services solve different failure levels.

```text
Route 53
   ↓
Region-level failover
   ↓
Load Balancer
   ↓
Instance-level traffic failover
   ↓
Auto Scaling Group
   ↓
Replace failed instance
```

### Route 53

Route 53 can direct users to another Region using DNS-based routing/failover.

### Load Balancer

The load balancer quickly stops sending traffic to unhealthy instances.

### Auto Scaling Group

The ASG replaces failed instances.

So:

```text
Load Balancer
= stop sending traffic to bad instance

ASG
= replace bad instance
```

These are different jobs.

---

## Question patterns

> *"Route `/api/*` to one target group and `/images/*` to another"* → **ALB path-based routing**

> *"Route traffic based on hostname"* → **ALB host-based routing**

> *"UDP-based game needs low latency and a static IP"* → **NLB**

> *"Millions of TCP connections with very low latency"* → **NLB**

> *"Inspect traffic using third-party firewall/IDS/IPS appliances"* → **Gateway Load Balancer**

> *"The load balancer marks an instance unhealthy but the ASG doesn't replace it"* → **Enable ELB health checks on the ASG**

> *"Traffic increases every month-end at a predictable time"* → **Scheduled Scaling**

> *"Keep average CPU at 40%"* → **Target Tracking**

> *"Different scaling actions for different metric thresholds"* → **Step Scaling**

> *"Predict future traffic and scale before it arrives"* → **Predictive Scaling**

> *"Many HTTPS domains use different certificates on one ALB"* → **SNI**

> *"Users receive errors when instances are removed during scale-in"* → **Deregistration delay / connection draining**

> *"Application behind ALB needs the original client IP"* → **X-Forwarded-For**

> *"Clients must whitelist fixed load-balancer IP addresses"* → **NLB with Elastic IPs**

> *"Scale workers based on pending SQS jobs"* → **ASG scaling based on SQS queue metrics**

> *"Need to preserve AZ balance during scale-in"* → **Default ASG termination policy**

> *"Terminate the instance that has been running the longest"* → **OldestInstance termination policy**

## Pocket card

| Keyword                                         | Answer                                             |
| ----------------------------------------------- | -------------------------------------------------- |
| Path / host / header routing                    | **ALB**                                            |
| HTTP / HTTPS / gRPC                             | **ALB**                                            |
| WAF on load balancer                            | **ALB**                                            |
| UDP                                             | **NLB**                                            |
| Very high performance / millions of connections | **NLB**                                            |
| Static IP / Elastic IP                          | **NLB**                                            |
| Preserve client source IP                       | **NLB**                                            |
| PrivateLink endpoint service                    | **NLB**                                            |
| Third-party firewall / IDS / IPS                | **GWLB**                                           |
| GENEVE 6081                                     | **GWLB**                                           |
| Classic Load Balancer                           | **Legacy / usually wrong**                         |
| Multiple HTTPS certificates                     | **SNI**                                            |
| Errors during scale-in                          | **Deregistration delay**                           |
| Uneven traffic across AZs                       | **Cross-zone load balancing**                      |
| Client IP behind ALB                            | **X-Forwarded-For**                                |
| Keep CPU at X%                                  | **Target Tracking**                                |
| Different scaling steps                         | **Step Scaling**                                   |
| Known traffic schedule                          | **Scheduled Scaling**                              |
| Predict future demand                           | **Predictive Scaling**                             |
| ASG ignores application failure                 | **Enable ELB health checks**                       |
| Scale workers on jobs                           | **SQS queue metric**                               |
| Default scale-in                                | **Keep AZs balanced + prefer older configuration** |
| Oldest running instance                         | **OldestInstance policy**                          |
| Traffic failover within a Region                | **Load Balancer**                                  |
| Replace failed EC2                              | **Auto Scaling Group**                             |
| Cross-Region DNS failover                       | **Route 53**                                       |

## Final memory

```text
ALB
= HTTP/HTTPS
= Layer 7
= path / host / header routing

NLB
= TCP/UDP/TLS
= Layer 4
= high performance + static IP

GWLB
= security appliances
= firewall / IDS / IPS

ASG
= add/remove instances
= replace failed instances

Target Tracking
= keep metric at target

Step Scaling
= different amounts for different thresholds

Scheduled Scaling
= known schedule

Predictive Scaling
= forecast future demand

ELB health check
= detect application failure

ASG
= replace unhealthy instances
```

The key distinction to remember is:

```text
Load Balancer
= "Which healthy instance should receive this request?"

Auto Scaling Group
= "How many instances should exist?"
```
