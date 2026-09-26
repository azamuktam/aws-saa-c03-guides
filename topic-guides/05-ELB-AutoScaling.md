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

## The four load balancers

|          | Layer | Protocols           | Main feature                                          | Exam keyword                     |
| -------- | ----- | ------------------- | ----------------------------------------------------- | -------------------------------- |
| **ALB**  | 7     | HTTP, HTTPS, gRPC   | Content-based routing + weighted target groups        | Path, host, header, web apps     |
| **NLB**  | 4     | TCP, UDP, TLS       | High performance + static IP + weighted target groups | UDP, very high traffic, fixed IP |
| **GWLB** | 3     | IP packets / GENEVE | Sends traffic through security appliances             | Firewall, IDS, IPS               |
| **CLB**  | 4/7   | Legacy              | Older load balancer                                   | Usually wrong answer             |

> **Important current AWS update:** NLB gained **Weighted Target Groups on November 19, 2025**. Therefore, do not memorize “weighted target groups = ALB only.” Both **ALB and NLB** can now distribute traffic between weighted target groups.

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

The weights determine the relative proportion of traffic sent to the target groups. AWS supports weights from **0 to 999**.

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

This is useful for:

* blue/green deployments
* canary deployments
* A/B testing
* gradual application migration
* hybrid/on-premises-to-AWS migration

AWS specifically documents weighted ALB target groups as a way to perform zero-downtime migration between **on-premises and cloud** environments.

### Gradual migration example

```text
On-premises   AWS
    90%        10%

     ↓

    70%        30%

     ↓

    50%        50%

     ↓

    10%        90%

     ↓

     0%       100%
```

### Important exam distinction

**Weighted target groups** split traffic **inside the load balancer**.

This is different from **Route 53 Weighted Routing**, which splits traffic at the **DNS level**.

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

The ALB can continue sending that user's requests to EC2-A.

Use this when the application keeps session state locally on the instance.

A better architecture is often to store session state externally, for example in:

* ElastiCache
* DynamoDB

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

**Since November 19, 2025, NLB supports Weighted Target Groups.**

You can assign each target group a weight from **0 to 999**.

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

or:

```text
Old application → 90
New application → 10
```

Then gradually:

```text
90 / 10
→ 70 / 30
→ 50 / 50
→ 20 / 80
→ 0 / 100
```

AWS specifically identifies **application migration, blue/green deployments, and canary deployments** as use cases for NLB weighted target groups.

### Important behavior

When weights change:

* **new connections** are routed according to the new weights
* **existing connections** are not immediately moved
* a target group with weight `0` receives no new connections

### ALB vs NLB weighted target groups

|                        | ALB                   | NLB |
| ---------------------- | --------------------- | --- |
| Weighted target groups | ✅                     | ✅   |
| Main layer             | L7                    | L4  |
| HTTP path routing      | ✅                     | ❌   |
| TCP/UDP                | ❌/limited by protocol | ✅   |
| Static IP              | ❌                     | ✅   |
| Application migration  | ✅                     | ✅   |

So the question should first be interpreted as:

> **Can this application use Layer 7 HTTP routing or does it require Layer 4 networking?**

If HTTP/HTTPS application-level routing is appropriate:

→ **ALB**

If TCP/UDP/static-IP requirements are important:

→ **NLB**

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

A very common exam scenario is:

> An application currently runs on-premises. A new version is running in AWS. The company wants to move traffic gradually without downtime.

The important concept is:

> **Traffic splitting**

Possible AWS mechanisms include:

### 1. ALB Weighted Target Groups

```text
                    ALB
                     |
          +----------+----------+
          |                     |
       Weight 50             Weight 50
          |                     |
       AWS app              On-prem app
```

This is especially appropriate for an HTTP/HTTPS application.

ALB weighted target groups support application migration between on-premises and AWS.

---

### 2. NLB Weighted Target Groups

For an application appropriate for Layer 4 load balancing:

```text
                    NLB
                     |
          +----------+----------+
          |                     |
       Weight 50             Weight 50
          |                     |
       AWS app              On-prem app
```

This capability is available because NLB supports weighted target groups since November 2025.

---

### 3. Route 53 Weighted Routing

Route 53 can also distribute traffic between resources using different weights.

Example:

```text
example.com
     |
     +---- Weight 50 → AWS
     |
     +---- Weight 50 → On-prem
```

You can gradually change the weights:

```text
50 / 50
→ 80 / 20
→ 100 / 0
```

Route 53 Weighted Routing is explicitly designed to route traffic to multiple resources in proportions you specify.

### Important difference

Route 53 works at the **DNS level**.

Therefore, it is not the same as a load balancer making a decision for every HTTP request.

DNS caching and TTL behavior mean the actual distribution seen by users can be approximate rather than an exact per-request 50/50 split.

So:

```text
ALB/NLB weighted target groups
→ Load-balancer traffic splitting

Route 53 weighted routing
→ DNS-level traffic splitting
```

---

## Weighted vs Failover routing

This is a major exam trap.

### Weighted routing

Use when you want traffic to go to **multiple resources simultaneously**.

Example:

```text
AWS       → 50%
On-prem   → 50%
```

→ **Weighted**

AWS defines Weighted Routing as routing traffic to multiple resources in specified proportions.

### Failover routing

Use for **active-passive** architecture.

Example:

```text
Primary AWS
    ↓
takes traffic

If unhealthy
    ↓
Secondary on-prem
```

→ **Failover**

Route 53 documents Failover Routing as an active-passive mechanism.

### Memory trick

```text
50% + 50%
→ Weighted

Primary + Backup
→ Failover
```

---

## Direct Connect + on-premises application

If an application is running on-premises and needs private connectivity to AWS:

```text
On-premises network
        |
        | Direct Connect
        |
      AWS VPC
```

Direct Connect provides a dedicated network connection between the on-premises environment and AWS.

This allows AWS resources to communicate with private on-premises resources.

### Exam pattern

> **"AWS VPC must privately communicate with the company's data center."**

Think:

→ **Direct Connect**

or:

→ **Site-to-Site VPN**

depending on the requirement.

---

## Cross-zone load balancing

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

## Deregistration delay / connection draining

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

→ **Deregistration delay / connection draining**

---

## SNI

**SNI (Server Name Indication)** allows one HTTPS listener to use multiple certificates.

Example:

```text
example.com
api.example.com
admin.example.com
```

One ALB can serve different certificates based on the requested hostname.

### Exam pattern

> **"Host multiple HTTPS domains with different certificates on one ALB."**

→ **SNI**

---

## TLS termination

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

# ASG Lifecycle Hooks

**Lifecycle hooks let you pause an EC2 instance during an Auto Scaling lifecycle transition and perform custom actions before the instance continues.** AWS provides lifecycle hooks for both launching and terminating instances. ([docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks-overview.html?utm_source=chatgpt.com))

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

The instance remains in the wait state until you complete the lifecycle action or the timeout expires. ([docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-lifecycle.html?utm_source=chatgpt.com))

---

## Pending:Wait

Used during **instance launch**.

Example:

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

Typical use:

* bootstrap the instance
* install software
* configure the application
* perform initialization before allowing normal traffic

### Exam clue

> **"Perform custom setup before the new instance enters service."**

→ **Launch lifecycle hook → `Pending:Wait`**

---

## Terminating:Wait

Used during **instance termination**.

This is the important state for preserving logs or other local data before an EC2 instance disappears.

```text
EC2 instance
     ↓
Selected for termination
     ↓
Terminating
     ↓
Terminating:Wait  ← PAUSE HERE
     ↓
Collect logs / cleanup / other actions
     ↓
Complete lifecycle action
     ↓
Terminating:Proceed
     ↓
Terminated
```

AWS specifically describes using a termination lifecycle hook to pause an instance before termination and download **logs or other data** while the instance is still available. ([docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html?utm_source=chatgpt.com))

### Exam clue

> **"Need to collect logs before an EC2 instance is terminated."**

→ **Termination lifecycle hook → `Terminating:Wait`**

---

## Lifecycle Hook + EventBridge

When a lifecycle hook puts an instance into a wait state, EC2 Auto Scaling sends an event to **Amazon EventBridge**.

The termination event type is:

```text
EC2 Instance-terminate Lifecycle Action
```

EventBridge can then invoke services such as:

* AWS Lambda
* Amazon SNS
* Amazon SQS
* other supported targets

([docs.aws.amazon.com](https://docs.aws.amazon.com/eventbridge/latest/ref/events-ref-autoscaling.html?utm_source=chatgpt.com))

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

The EventBridge lifecycle event contains information such as the **EC2 instance ID**, Auto Scaling group name, lifecycle hook name, and lifecycle action token. ([docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-event-reference.html?utm_source=chatgpt.com))

---

## Log collection before termination

Suppose local application logs exist only on the EC2 instance.

If ASG terminates the instance immediately:

```text
EC2
 ↓
Terminate
 ↓
Local logs lost ❌
```

Instead:

```text
EC2
 ↓
Terminating:Wait
 ↓
EventBridge
 ↓
Lambda
 ↓
CloudWatch Agent / log collection
 ↓
CloudWatch Logs
 ↓
CompleteLifecycleAction
 ↓
Terminate
```

This gives the system time to collect the logs before the instance disappears.

### Example

> "Instances are automatically terminated after failing ALB health checks, but application logs are stored locally. The company needs the logs for root cause analysis."

Think:

**Termination lifecycle hook + `Terminating:Wait` + EventBridge/Lambda + CloudWatch Logs**

### Important trap

Do **not** wait for:

```text
EC2 Instance Terminate Successful
```

That event happens after termination has completed.

By then, local logs may already be gone.

Instead, use:

```text
EC2 Instance-terminate Lifecycle Action
```

while the instance is in the lifecycle-hook wait state. ([docs.aws.amazon.com](https://docs.aws.amazon.com/eventbridge/latest/ref/events-ref-autoscaling.html?utm_source=chatgpt.com))

---

## Complete lifecycle action

After the custom action has finished, the workflow must tell Auto Scaling to continue.

Conceptually:

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

If more time is needed, the lifecycle action can be extended using a heartbeat.

AWS documents `CompleteLifecycleAction` for completing the lifecycle action and `RecordLifecycleActionHeartbeat` for extending the wait period. ([docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CompleteLifecycleAction.html?utm_source=chatgpt.com))

### Memory

```text
Pending:Wait
→ do something BEFORE instance enters service

Terminating:Wait
→ do something BEFORE instance is terminated
```

---

# THE health-check trap

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

So the ASG can scale based on the queue.

A useful metric is:

> **Backlog per instance**

This is better than simply looking at CPU when the real problem is the number of pending jobs.

### Exam pattern

> **"Scale workers according to the number of unprocessed jobs."**

→ **Scale the ASG using SQS queue metrics**

---

# Termination policy — who is terminated first?

When an ASG needs to scale in, it has to decide **which instance to remove**.

With the default termination policy, the ASG first tries to keep the Availability Zones balanced.

It then prefers instances using **older launch configurations or older launch template versions/configurations**.

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

# The layered HA picture

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

Route 53 can direct users between resources using DNS-based routing policies such as:

* Weighted
* Failover
* Latency
* Geolocation
* Geoproximity

### Load Balancer

The load balancer quickly stops sending traffic to unhealthy targets.

### Auto Scaling Group

The ASG replaces failed instances.

So:

```text
Load Balancer
= stop sending traffic to bad target

ASG
= replace bad instance
```

These are different jobs.

---

# Question patterns

> *"Route `/api/*` to one target group and `/images/*` to another"* → **ALB path-based routing**

> *"Route traffic based on hostname"* → **ALB host-based routing**

> *"UDP-based game needs low latency and a static IP"* → **NLB**

> *"Millions of TCP connections with very low latency"* → **NLB**

> *"Third-party firewall/IDS/IPS appliances"* → **Gateway Load Balancer**

> *"Split traffic 50/50 between two application versions"* → **Weighted Target Groups** or **Route 53 Weighted Routing**, depending on where the traffic split should occur

> *"Gradually migrate an HTTP application from on-premises to AWS"* → **ALB Weighted Target Groups** or **Route 53 Weighted Routing**

> *"Gradually migrate a TCP/UDP application from on-premises to AWS"* → **NLB Weighted Target Groups** or **Route 53 Weighted Routing**

> *"Primary application receives traffic; secondary receives traffic only if primary fails"* → **Failover routing**

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

> *"Terminate the instance that has been running the longest"* → **OldestInstance policy**

> *"Need to collect logs before ASG terminates an unhealthy instance"* → **Termination lifecycle hook → `Terminating:Wait`**

> *"Perform custom actions before an EC2 instance enters service"* → **Launch lifecycle hook → `Pending:Wait`**

> *"React when an instance enters a termination lifecycle hook"* → **EventBridge `EC2 Instance-terminate Lifecycle Action`**

> *"Automatically collect logs before termination"* → **Lifecycle hook + EventBridge/Lambda + CloudWatch Logs**

> *"Primary application receives traffic; secondary receives traffic only if primary fails"* → **Failover routing**

> *"AWS VPC needs private connectivity to an on-premises data center"* → **Direct Connect or Site-to-Site VPN**, depending on the requirement

---

# Pocket card

| Keyword                                     | Answer                                                    |
| ------------------------------------------- | --------------------------------------------------------- |
| Path / host / header routing                | **ALB**                                                   |
| HTTP / HTTPS / gRPC                         | **ALB**                                                   |
| WAF on load balancer                        | **ALB**                                                   |
| Weighted target groups                      | **ALB or NLB**                                            |
| Blue/green migration                        | **Weighted Target Groups**                                |
| Canary deployment                           | **Weighted Target Groups**                                |
| Gradual on-prem → AWS migration             | **Weighted Target Groups / Route 53 Weighted**            |
| UDP                                         | **NLB**                                                   |
| Very high performance / many connections    | **NLB**                                                   |
| Static IP / Elastic IP                      | **NLB**                                                   |
| Preserve client source IP                   | **NLB**                                                   |
| PrivateLink endpoint service                | **NLB**                                                   |
| Third-party firewall / IDS / IPS            | **GWLB**                                                  |
| GENEVE 6081                                 | **GWLB**                                                  |
| Classic Load Balancer                       | **Legacy / usually wrong**                                |
| Multiple HTTPS certificates                 | **SNI**                                                   |
| Errors during scale-in                      | **Deregistration delay**                                  |
| Uneven traffic across AZs                   | **Cross-zone load balancing**                             |
| Client IP behind ALB                        | **X-Forwarded-For**                                       |
| Keep CPU at X%                              | **Target Tracking**                                       |
| Different scaling steps                     | **Step Scaling**                                          |
| Known traffic schedule                      | **Scheduled Scaling**                                     |
| Predict future demand                       | **Predictive Scaling**                                    |
| ASG ignores application failure             | **Enable ELB health checks**                              |
| Scale workers on jobs                       | **SQS queue metric**                                      |
| Need to preserve AZ balance during scale-in | **Default ASG termination policy**                        |
| Oldest running instance                     | **OldestInstance policy**                                 |
| Need action before launch                   | **`Pending:Wait` lifecycle hook**                         |
| Need action before termination              | **`Terminating:Wait` lifecycle hook**                     |
| React to termination lifecycle event        | **EventBridge**                                           |
| Collect logs before termination             | **Lifecycle Hook + EventBridge/Lambda + CloudWatch Logs** |
| Termination event                           | **`EC2 Instance-terminate Lifecycle Action`**             |
| 50/50 traffic split                         | **Weighted routing**                                      |
| Primary + backup                            | **Failover routing**                                      |
| DNS-level traffic split                     | **Route 53 Weighted Routing**                             |
| Load-balancer-level traffic split           | **Weighted Target Groups**                                |
| Private AWS ↔ on-prem connectivity          | **Direct Connect / VPN**                                  |
| Traffic failover within a Region            | **Load Balancer**                                         |
| Replace failed EC2                          | **Auto Scaling Group**                                    |

---

# Lifecycle hook decision rule

```text
Need to do something BEFORE a new instance enters service?
→ Launch lifecycle hook
→ Pending:Wait

Need to do something BEFORE an instance is terminated?
→ Termination lifecycle hook
→ Terminating:Wait

Need to react to lifecycle events externally?
→ EventBridge

Need custom code / automation?
→ Lambda

Need to preserve local logs before termination?
→ Terminating:Wait
→ EventBridge / Lambda
→ CloudWatch Logs
→ CompleteLifecycleAction
```
