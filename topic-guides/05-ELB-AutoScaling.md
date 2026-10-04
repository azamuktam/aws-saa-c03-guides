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

A Layer 4 load balancer cannot make decisions based on:

```text
/api/orders
/images/logo.png
```

because those are **HTTP-level details**.

A Layer 7 load balancer can.

So:

```text
Need URL/path/host/header routing
→ Layer 7 → ALB

Need TCP/UDP + very high performance
→ Layer 4 → NLB
```

---

# The four load balancers

|          | Layer | Protocols           | Main feature                                          | Exam keyword                     |
| -------- | ----- | ------------------- | ----------------------------------------------------- | -------------------------------- |
| **ALB**  | 7     | HTTP, HTTPS, gRPC   | Content-based routing + weighted target groups        | Path, host, header, web apps     |
| **NLB**  | 4     | TCP, UDP, TLS       | High performance + static IP + weighted target groups | UDP, very high traffic, fixed IP |
| **GWLB** | 3     | IP packets / GENEVE | Sends traffic through security appliances             | Firewall, IDS, IPS               |
| **CLB**  | 4/7   | Legacy              | Older load balancer                                   | Usually wrong answer             |

> **Important:** Both **ALB and NLB** support **Weighted Target Groups**.

---

# ALB — Application Load Balancer

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
* ALB supports **Weighted Target Groups**.

---

## ALB Weighted Target Groups

ALB can forward traffic to multiple target groups and assign each group a weight.

Example:

```text
                ALB
                 |
       +---------+---------+
       |                   |
   Weight 50            Weight 50
       |                   |
    AWS app           On-prem app
```

Or:

```text
AWS version    → weight 90
New version    → weight 10
```

For example:

```text
Target Group A = 80
Target Group B = 20
```

approximately produces:

```text
80% → A
20% → B
```

Useful for:

* blue/green deployments
* canary deployments
* A/B testing
* gradual application migration
* hybrid/on-premises-to-AWS migration

### Important exam distinction

**Weighted Target Groups** split traffic **inside the load balancer**.

**Route 53 Weighted Routing** splits traffic at the **DNS level**.

---

## Client IP behind ALB

The target normally sees the ALB connection.

The original client IP is normally available in:

```text
X-Forwarded-For
```

So:

> **"Application behind ALB needs the original client IP."**

→ **X-Forwarded-For**

---

## Sticky sessions

Sticky sessions can keep a client connected to the same target.

Example:

```text
User A
  ↓
ALB
  ↓
EC2-A
```

Use this when the application keeps session state locally on the instance.

A better architecture is often to store session state externally, for example in:

* ElastiCache
* DynamoDB

### Important distinction

> **Sticky session** → user stays with the **same target**

> **Shared session store** → any healthy server can retrieve the **same session**

---

## ECS dynamic port mapping

If several containers run on the same EC2 instance, each container can use a different port.

ALB can discover and route to those ports.

This is useful with ECS.

---

## Fixed IP requirement

ALB does **not** provide fixed static IP addresses for clients to whitelist.

If the requirement is:

> "Customers must whitelist fixed IP addresses."

Think:

→ **NLB**

or:

→ **Global Accelerator**

---

# NLB — Network Load Balancer

NLB works mainly at **Layer 4**.

It primarily handles:

* IP address
* port
* TCP/UDP/TLS connection information

It does not use HTTP URL paths like:

```text
/api/*
/images/*
```

for normal application routing.

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
* Supports **Weighted Target Groups**

### Use NLB for

* UDP applications
* gaming
* IoT
* custom TCP protocols
* extremely high traffic
* fixed IP requirements
* TCP/TLS services where Layer 4 routing is sufficient

### Example

> "A gaming application uses UDP and requires a fixed IP."

→ **NLB**

---

# NLB Weighted Target Groups

NLB supports **Weighted Target Groups**.

Example:

```text
                NLB
                 |
       +---------+---------+
       |                   |
   Weight 50            Weight 50
       |                   |
    AWS app           On-prem app
```

Useful for:

* application migration
* blue/green deployments
* canary deployments

### Important behavior

When weights change:

* **new connections** use the new weights
* **existing connections** are not immediately moved
* weight `0` means no new connections to that target group

### ALB vs NLB weighted target groups

|                        |                   ALB | NLB |
| ---------------------- | --------------------: | --: |
| Weighted target groups |                     ✅ |   ✅ |
| Main layer             |                    L7 |  L4 |
| HTTP path routing      |                     ✅ |   ❌ |
| TCP/UDP                | ❌/limited by protocol |   ✅ |
| Static IP              |                     ❌ |   ✅ |
| Application migration  |                     ✅ |   ✅ |

---

# GWLB — Gateway Load Balancer

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

# CLB — Classic Load Balancer

Classic Load Balancer is the **older generation**.

For modern architectures, prefer:

* **ALB**
* **NLB**
* **GWLB**

If CLB appears as a distractor in a modern architecture question, it is usually not the answer.

---

# Traffic splitting and migration

A common exam scenario is:

> An application currently runs on-premises. A new version is running in AWS. The company wants to move traffic gradually without downtime.

Possible mechanisms:

### ALB Weighted Target Groups

For HTTP/HTTPS:

```text
                    ALB
                     |
          +----------+----------+
          |                     |
       Weight 50             Weight 50
          |                     |
       AWS app              On-prem app
```

### NLB Weighted Target Groups

For Layer 4 applications:

```text
                    NLB
                     |
          +----------+----------+
          |                     |
       Weight 50             Weight 50
          |                     |
       AWS app              On-prem app
```

### Route 53 Weighted Routing

DNS-level traffic splitting:

```text
example.com
     |
     +---- Weight 50 → AWS
     |
     +---- Weight 50 → On-prem
```

### Key distinction

```text
ALB/NLB Weighted Target Groups
→ load-balancer-level traffic splitting

Route 53 Weighted Routing
→ DNS-level traffic splitting
```

---

# Weighted vs Failover routing

### Weighted routing

Use when multiple resources should receive traffic simultaneously.

```text
AWS       → 50%
On-prem   → 50%
```

→ **Weighted**

### Failover routing

Use for active-passive architecture.

```text
Primary AWS
    ↓
receives traffic

If unhealthy
    ↓
Secondary
```

→ **Failover**

### Memory trick

```text
50% + 50%
→ Weighted

Primary + Backup
→ Failover
```

---

# Direct Connect + on-premises application

If an application is running on-premises and needs private connectivity to AWS:

```text
On-premises network
        |
        | Direct Connect
        |
      AWS VPC
```

Direct Connect provides a dedicated network connection between the on-premises environment and AWS.

### Exam pattern

> **"AWS VPC must privately communicate with the company's data center."**

Think:

→ **Direct Connect**

or:

→ **Site-to-Site VPN**

depending on the requirement.

---

# Cross-zone load balancing

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

→ Check **cross-zone load balancing**

---

# Deregistration delay / connection draining

When a target is being removed, the load balancer should not immediately kill existing connections.

```text
New requests
→ stop sending to target

Existing requests
→ allowed to finish
```

This is called:

* **deregistration delay**
* **connection draining**

### Exam clue

> **"Users receive errors when instances are removed during scale-in."**

→ **Deregistration delay / connection draining**

---

# SNI

**SNI (Server Name Indication)** allows one HTTPS listener to use multiple certificates.

Example:

```text
example.com
api.example.com
admin.example.com
```

One ALB can serve different certificates based on the requested hostname.

### Exam clue

> **"Host multiple HTTPS domains with different certificates on one ALB."**

→ **SNI**

---

# TLS termination

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

A Launch Template contains configuration for new instances, such as:

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

---

## Min / Desired / Max

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

# ECS Service vs Cluster Auto Scaling

With **ECS using the EC2 launch type**, scaling can happen at two levels.

| Scaling                      | Scales                      | Purpose                     |
| ---------------------------- | --------------------------- | --------------------------- |
| **ECS Service Auto Scaling** | **ECS tasks**               | Handle application workload |
| **ECS Cluster Auto Scaling** | **EC2 container instances** | Provide capacity for tasks  |

### Service scaling

```text
High CPU / memory / ALB requests
            ↓
ECS Service Auto Scaling
            ↓
       More ECS tasks
```

Common service scaling metrics:

* CPU utilization
* Memory utilization
* `ALBRequestCountPerTarget`

### Cluster scaling

```text
More task capacity needed
          ↓
ECS Capacity Provider
          ↓
   More EC2 instances
```

> **Service → tasks**
> **Cluster → EC2 instances**
> **Capacity Provider → manages EC2 capacity for ECS**

### Current AWS nuance

With ECS Capacity Provider managed scaling, the underlying EC2 capacity is managed using **`CapacityProviderReservation`** rather than directly using the ECS service CPU metric.

---

# ASG Scaling Policies

## Target Tracking

Keep a metric around a target value.

Example:

```text
Target CPU = 40%

CPU > 40%
→ scale out

CPU < 40%
→ scale in
```

### Exam clue

> **"Maintain average CPU utilization around 40%."**

→ **Target Tracking**

---

## Step Scaling

Different scaling actions occur at different thresholds.

Example:

```text
CPU < 40%       → remove 1
CPU 40–70%      → no change
CPU 70–90%      → add 1
CPU > 90%       → add 3
```

### Exam clue

> **"Take different scaling actions depending on how far the metric exceeds the threshold."**

→ **Step Scaling**

---

## Scheduled Scaling

Used when traffic patterns are predictable.

Example:

```text
Every weekday at 09:00
→ scale out

Every weekday at 18:00
→ scale in
```

### Exam clue

> **"Traffic increases every day at a predictable time."**

→ **Scheduled Scaling**

---

## Predictive Scaling

Predictive Scaling uses historical patterns to forecast future demand and scale ahead of it.

```text
Historical traffic
        ↓
Forecast
        ↓
Scale before demand arrives
```

### Exam clue

> **"Predict future demand and scale before the traffic spike."**

→ **Predictive Scaling**

---

# ASG Lifecycle Hooks

**Lifecycle hooks let you pause an EC2 instance during an Auto Scaling lifecycle transition and perform custom actions before the instance continues.**

The two important wait states are:

```text
Launch:
Pending
   ↓
Pending:Wait
   ↓
Pending:Proceed
   ↓
InService
```

and:

```text
Terminate:
Terminating
   ↓
Terminating:Wait
   ↓
Terminating:Proceed
   ↓
Terminated
```

### Pending:Wait

Used during **instance launch**.

```text
New EC2 instance
      ↓
Pending
      ↓
Pending:Wait
      ↓
Install/configure application
      ↓
Complete lifecycle action
      ↓
Pending:Proceed
      ↓
InService
```

### Exam clue

> **"Perform custom setup before the new instance enters service."**

→ **Launch lifecycle hook → `Pending:Wait`**

---

## Terminating:Wait

Used during **instance termination**.

```text
EC2 instance
     ↓
Selected for termination
     ↓
Terminating
     ↓
Terminating:Wait  ← PAUSE HERE
     ↓
Collect logs / cleanup
     ↓
Complete lifecycle action
     ↓
Terminating:Proceed
     ↓
Terminated
```

### Exam clue

> **"Need to collect logs before an EC2 instance is terminated."**

→ **Termination lifecycle hook → `Terminating:Wait`**

---

## Lifecycle Hook + EventBridge

Typical pattern:

```text
ASG
 ↓
Terminating:Wait
 ↓
EventBridge
 ↓
Lambda
 ↓
Perform custom action
 ↓
Complete lifecycle action
 ↓
Terminate
```

EventBridge can invoke services such as:

* AWS Lambda
* Amazon SNS
* Amazon SQS

---

## Log collection before termination

Suppose local application logs exist only on the EC2 instance.

```text
EC2
 ↓
Terminating:Wait
 ↓
EventBridge
 ↓
Lambda
 ↓
Collect logs
 ↓
CloudWatch Logs / S3
 ↓
CompleteLifecycleAction
 ↓
Terminate
```

### Exam clue

> **"Collect logs before the instance is terminated."**

→ **Termination lifecycle hook + `Terminating:Wait`**

### Important trap

Do not wait for an event that occurs **after termination** if the data exists only on the instance.

The lifecycle hook must pause termination while the instance still exists.

---

## Complete lifecycle action

After the custom action finishes:

```text
Collect logs
     ↓
Success
     ↓
CompleteLifecycleAction
     ↓
Terminating:Proceed
     ↓
Terminated
```

If more time is needed, use a lifecycle heartbeat.

### Memory

```text
Pending:Wait
→ do something BEFORE instance enters service

Terminating:Wait
→ do something BEFORE instance is terminated
```

---

# The health-check trap

By default, an ASG uses **EC2 health checks**.

This can happen:

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

# ASG + SQS pattern

Suppose EC2 workers process jobs from SQS.

```text
Application
    ↓
   SQS
    ↓
ASG workers
```

If the queue grows, you need more workers.

A useful metric is:

> **Backlog per instance**

This is better than simply looking at CPU when the real problem is the number of pending jobs.

### Exam pattern

> **"Scale workers according to the number of unprocessed jobs."**

→ **Scale the ASG using SQS queue metrics**

---

# Termination policy

When an ASG scales in, it has to decide which instance to remove.

### Important distinction

Do not confuse:

> **Older configuration**

with:

> **Oldest running instance**

The **`OldestInstance` termination policy** specifically prefers the instance that has been running the longest.

```text
Default behavior
→ maintain AZ balance + consider older configuration

OldestInstance policy
→ terminate the oldest running instance
```

---

# Route 53 + Load Balancer + ASG

Different AWS services solve different failure levels.

```text
Route 53
   ↓
DNS-level traffic routing / failover
   ↓
Load Balancer
   ↓
Instance-level traffic distribution
   ↓
Auto Scaling Group
   ↓
Replace failed instance
```

### Route 53

Route 53 can use:

* Weighted
* Failover
* Latency
* Geolocation
* Geoproximity

### Load Balancer

Stops sending traffic to unhealthy targets.

### Auto Scaling Group

Replaces failed instances.

So:

```text
Load Balancer
= stop sending traffic to bad target

ASG
= replace bad instance
```

---

# Route 53 Latency-Based Routing

**Latency-based routing** chooses the configured endpoint with the lowest network latency for the requester.

Example:

```text
                    Route 53
               Latency-based routing
                  /      |      \
                 ↓       ↓       ↓
              ALB Ohio  ALB Cali  ALB Ireland
                 ↓       ↓       ↓
               EC2s    EC2s     EC2s
```

### Exam clue

> **"Route users to the Region with the lowest latency."**

→ **Route 53 Latency-Based Routing**

### One Region only

If all EC2 instances are in one Region:

```text
Users
  ↓
ALB/NLB
  ↓
EC2 EC2 EC2
```

Use:

→ **ALB** for HTTP/HTTPS application routing

→ **NLB** for TCP/UDP/static-IP/high-performance Layer 4 requirements

**ALB does not provide Route 53-style cross-Region latency-based routing.**

---

# Route 53 Weighted vs Failover vs Latency

| Requirement                                       | Route 53 policy           |
| ------------------------------------------------- | ------------------------- |
| Send traffic to multiple resources in proportions | **Weighted**              |
| Primary + standby                                 | **Failover**              |
| Lowest-latency Region                             | **Latency-Based Routing** |
| Route based on user location                      | **Geolocation**           |
| Route based on geographic distance + bias         | **Geoproximity**          |

### Memory

```text
50% + 50%
→ Weighted

Primary + Backup
→ Failover

Lowest network latency
→ Latency
```

---

# Cross-zone load balancing

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

→ Check **cross-zone load balancing**

---

# Question patterns

> *"Route `/api/*` to one target group and `/images/*` to another"*
> → **ALB path-based routing**

> *"Route traffic based on hostname"*
> → **ALB host-based routing**

> *"UDP-based game needs low latency and a static IP"*
> → **NLB**

> *"Millions of TCP connections with very low latency"*
> → **NLB**

> *"Third-party firewall/IDS/IPS appliances"*
> → **Gateway Load Balancer**

> *"Split traffic 50/50 between two application versions"*
> → **Weighted Target Groups** or **Route 53 Weighted Routing**, depending on where the split occurs

> *"Route users to the Region with the lowest latency"*
> → **Route 53 Latency-Based Routing**

> *"Gradually migrate an HTTP application from on-premises to AWS"*
> → **ALB Weighted Target Groups** or **Route 53 Weighted Routing**

> *"Gradually migrate a TCP/UDP application from on-premises to AWS"*
> → **NLB Weighted Target Groups** or **Route 53 Weighted Routing**

> *"Primary application receives traffic; secondary receives traffic only if primary fails"*
> → **Failover routing**

> *"The load balancer marks instances unhealthy, but the ASG doesn't replace them"*
> → **Enable ELB health checks on the ASG**

> *"Traffic increases at a predictable time"*
> → **Scheduled Scaling**

> *"Keep average CPU at 40%"*
> → **Target Tracking**

> *"Different scaling actions for different metric thresholds"*
> → **Step Scaling**

> *"Predict future traffic and scale before it arrives"*
> → **Predictive Scaling**

> *"Multiple HTTPS domains use different certificates on one ALB"*
> → **SNI**

> *"Users receive errors when instances are removed during scale-in"*
> → **Deregistration delay / connection draining**

> *"Application behind ALB needs the original client IP"*
> → **X-Forwarded-For**

> *"Clients must whitelist fixed load-balancer IP addresses"*
> → **NLB with Elastic IPs**

> *"Scale workers based on pending SQS jobs"*
> → **ASG scaling using SQS metrics**

> *"Terminate the instance that has been running the longest"*
> → **OldestInstance policy**

> *"Need to collect logs before ASG terminates an instance"*
> → **Termination lifecycle hook → `Terminating:Wait`**

> *"Perform custom actions before an EC2 instance enters service"*
> → **Launch lifecycle hook → `Pending:Wait`**

> *"React to lifecycle events"*
> → **EventBridge**

> *"ECS service has high CPU or memory"*
> → **ECS Service Auto Scaling → more tasks**

> *"ECS cluster lacks EC2 capacity"*
> → **ECS Capacity Provider / Cluster Auto Scaling → more EC2 instances**

---

# Pocket card

| Keyword                                  | Answer                                                            |
| ---------------------------------------- | ----------------------------------------------------------------- |
| Path / host / header routing             | **ALB**                                                           |
| HTTP / HTTPS / gRPC                      | **ALB**                                                           |
| WAF on load balancer                     | **ALB**                                                           |
| Weighted target groups                   | **ALB or NLB**                                                    |
| Blue/green migration                     | **Weighted Target Groups**                                        |
| Canary deployment                        | **Weighted Target Groups**                                        |
| Gradual on-prem → AWS migration          | **Weighted Target Groups / Route 53 Weighted**                    |
| Lowest-latency Region                    | **Route 53 Latency-Based Routing**                                |
| UDP                                      | **NLB**                                                           |
| Very high performance / many connections | **NLB**                                                           |
| Static IP / Elastic IP                   | **NLB**                                                           |
| Preserve client source IP                | **NLB**                                                           |
| PrivateLink endpoint service             | **NLB**                                                           |
| Third-party firewall / IDS / IPS         | **GWLB**                                                          |
| GENEVE 6081                              | **GWLB**                                                          |
| Classic Load Balancer                    | **Legacy / usually wrong**                                        |
| Multiple HTTPS certificates              | **SNI**                                                           |
| Errors during scale-in                   | **Deregistration delay**                                          |
| Uneven traffic across AZs                | **Cross-zone load balancing**                                     |
| Client IP behind ALB                     | **X-Forwarded-For**                                               |
| Keep CPU at X%                           | **Target Tracking**                                               |
| Different scaling steps                  | **Step Scaling**                                                  |
| Known traffic schedule                   | **Scheduled Scaling**                                             |
| Predict future demand                    | **Predictive Scaling**                                            |
| ASG ignores application failure          | **Enable ELB health checks**                                      |
| Scale workers on jobs                    | **SQS queue metric**                                              |
| Oldest running instance                  | **OldestInstance policy**                                         |
| Need action before launch                | **`Pending:Wait` lifecycle hook**                                 |
| Need action before termination           | **`Terminating:Wait` lifecycle hook**                             |
| React to lifecycle events                | **EventBridge**                                                   |
| Collect logs before termination          | **Lifecycle Hook + EventBridge/Lambda + CloudWatch Logs**         |
| 50/50 traffic split                      | **Weighted routing**                                              |
| Primary + backup                         | **Failover routing**                                              |
| DNS-level traffic split                  | **Route 53 Weighted Routing**                                     |
| Load-balancer-level traffic split        | **Weighted Target Groups**                                        |
| Private AWS ↔ on-prem connectivity       | **Direct Connect / VPN**                                          |
| Replace failed EC2                       | **Auto Scaling Group**                                            |
| ECS service overloaded                   | **ECS Service Auto Scaling → more tasks**                         |
| ECS cluster lacks capacity               | **Capacity Provider / Cluster Auto Scaling → more EC2 instances** |
| Capacity Provider                        | **Manages EC2 capacity for ECS**                                  |

---

# Core mental model

```text
                         Route 53
                    DNS-level routing
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          ALB/NLB     ALB/NLB     ALB/NLB
             │           │           │
            EC2         EC2         EC2
             │
            ASG
             │
       Self-healing + scaling
```

For ECS on EC2:

```text
ECS Service
    ↓
Service Auto Scaling
    ↓
More ECS tasks

ECS Capacity Provider
    ↓
Cluster Auto Scaling
    ↓
More EC2 container instances
```

### Final shortcuts

```text
ALB
→ Layer 7 / HTTP / path / host

NLB
→ Layer 4 / TCP / UDP / static IP

GWLB
→ security appliances / firewall / IPS

Route 53 Latency
→ choose lowest-latency Region

ASG
→ manage EC2 instances

ECS Service Auto Scaling
→ manage ECS tasks

ECS Cluster Auto Scaling
→ manage EC2 container capacity

Capacity Provider
→ connect ECS task demand to EC2 capacity
```
