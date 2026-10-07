# Section 5: Elastic Load Balancing & Auto Scaling

## The idea

**Elastic Load Balancing (ELB)** distributes incoming traffic across multiple targets.

```text
Clients
   ↓
Load Balancer
   ↓
EC2 instances
```

The load balancer performs health checks:

```text
Healthy instance   → receives traffic ✅
Unhealthy instance → stops receiving traffic ❌
```

**Auto Scaling Group (ASG)** manages the number of EC2 instances.

```text
High demand → scale out
Low demand  → scale in
Failure     → replace instance
```

Together:

```text
Clients
   ↓
Load Balancer
   ↓
Auto Scaling Group
   ↓
EC2 instances
```

---

# OSI layers

| Layer  | What it sees        | Example             |
| ------ | ------------------- | ------------------- |
| **L3** | IP                  | `10.0.1.10`         |
| **L4** | IP + port + TCP/UDP | TCP 443, UDP 5000   |
| **L7** | HTTP details        | Path, host, headers |

```text
URL/path/host/header routing
→ ALB

TCP/UDP, static IP, very high performance
→ NLB
```

---

# Load Balancers

|          | Layer | Main use                                 | Exam keyword       |
| -------- | ----- | ---------------------------------------- | ------------------ |
| **ALB**  | 7     | HTTP/HTTPS/gRPC, content routing         | Path, host, header |
| **NLB**  | 4     | TCP/UDP/TLS, static IP, high performance | UDP, fixed IP      |
| **GWLB** | 3     | Security appliances                      | Firewall, IDS, IPS |
| **CLB**  | 4/7   | Legacy                                   | Usually avoid      |

> **ALB and NLB both support Weighted Target Groups.**

---

# ALB — Application Load Balancer

Layer 7.

Supports routing based on:

* URL path
* Hostname
* HTTP headers
* Query strings

Examples:

```text
/api/*       → API target group
/images/*    → Image target group
```

```text
api.example.com
   ↓
API servers

admin.example.com
   ↓
Admin servers
```

### Important facts

* Targets can be EC2, IP addresses, Lambda, or containers.
* Performs health checks.
* Supports AWS WAF.
* Uses a DNS name, not fixed static IPs.
* Supports Weighted Target Groups.

### Weighted Target Groups

```text
ALB
 ├── 80% → Version A
 └── 20% → Version B
```

Useful for:

* Blue/green deployments
* Canary deployments
* A/B testing
* Gradual migration

> **Weighted Target Groups = traffic split at the load balancer.**
> **Route 53 Weighted Routing = traffic split at DNS level.**

### Client IP behind ALB

The original client IP is normally available through:

```text
X-Forwarded-For
```

### Sticky sessions

Keeps a client on the same target.

```text
User → ALB → EC2-A
```

Useful when session state is stored locally.

A better architecture is often an external session store such as:

* ElastiCache
* DynamoDB

### ECS dynamic port mapping

ALB can route to different container ports on the same EC2 instance.

### Fixed IP

ALB does not provide fixed static IPs.

```text
Fixed IP requirement
→ NLB
```

---

# NLB — Network Load Balancer

Layer 4.

Supports:

* TCP
* UDP
* TLS

Main characteristics:

* Very high throughput
* Very low latency
* Static IP per AZ
* Can use Elastic IPs
* Preserves client source IP
* Supports Weighted Target Groups

Use NLB for:

* UDP applications
* Gaming
* IoT
* Custom TCP protocols
* Very high connection volumes
* Fixed IP requirements

Example:

> UDP gaming application requiring a fixed IP

→ **NLB**

---

# NLB Weighted Target Groups

```text
NLB
 ├── 50% → AWS
 └── 50% → On-prem
```

Useful for:

* Migration
* Blue/green
* Canary deployments

Important:

* New connections use the new weights.
* Existing connections are not immediately moved.
* Weight `0` means no new connections to that target group.

---

# GWLB — Gateway Load Balancer

Used to pass traffic through security appliances.

Examples:

* Firewalls
* IDS
* IPS
* Traffic inspection appliances

```text
Traffic
   ↓
GWLB
   ↓
Security appliance
   ↓
Application
```

Uses **GENEVE on port 6081**.

> **Firewall / IDS / IPS → GWLB**

---

# CLB — Classic Load Balancer

Legacy generation.

For modern architectures, prefer:

* ALB
* NLB
* GWLB

---

# Traffic splitting

### ALB/NLB Weighted Target Groups

```text
AWS app      → 80%
On-prem app  → 20%
```

Load-balancer-level splitting.

### Route 53 Weighted Routing

```text
example.com
 ├── 80% → AWS
 └── 20% → On-prem
```

DNS-level splitting.

### Weighted vs Failover

```text
Multiple resources receive traffic
→ Weighted

Primary + backup
→ Failover
```

---

# Direct Connect

Private connectivity between on-premises and AWS:

```text
On-premises
     |
Direct Connect
     |
   AWS VPC
```

> **Private AWS ↔ data center connectivity → Direct Connect or VPN**

---

# Cross-zone load balancing

Without cross-zone balancing, a load balancer node normally sends traffic to targets in its own AZ.

With cross-zone balancing, traffic can be distributed across targets in other AZs.

### Defaults

* **ALB:** enabled by default.
* **NLB:** disabled by default at the load balancer level.

> Uneven traffic across AZs → check **cross-zone load balancing**.

---

# Deregistration delay

When a target is removed:

```text
New requests
→ stop going to target

Existing connections
→ allowed to finish
```

Also called:

* Deregistration delay
* Connection draining

> Errors during scale-in → **deregistration delay**

---

# SNI

**SNI = Server Name Indication**

Allows one HTTPS listener to serve multiple certificates based on hostname.

```text
example.com
api.example.com
admin.example.com
```

> Multiple HTTPS domains with different certificates → **SNI**

---

# TLS termination

The load balancer can terminate HTTPS:

```text
Client
  ↓ HTTPS
ALB
  ↓ HTTP/HTTPS
EC2
```

Certificates are commonly stored in **ACM**.

---

# Auto Scaling Groups

An **ASG** manages a group of EC2 instances.

It uses a **Launch Template** to define how new instances are launched.

Launch Template can contain:

* AMI
* Instance type
* Security groups
* User data
* Key pair

### Min / Desired / Max

```text
min     = 2
desired = 4
max     = 10
```

### Self-healing

```text
Instance fails
     ↓
ASG detects it
     ↓
Replace instance
```

---

# ECS scaling

With ECS using EC2:

| Scaling                      | Scales               | Purpose                              |
| ---------------------------- | -------------------- | ------------------------------------ |
| **ECS Service Auto Scaling** | Tasks                | Application workload                 |
| **ECS Cluster Auto Scaling** | EC2 instances        | Container capacity                   |
| **Capacity Provider**        | EC2 capacity for ECS | Connects task demand to EC2 capacity |


```text
High application load
→ More ECS tasks

Not enough EC2 capacity
→ More EC2 instances

                ECS
                /   \
             EC2    Fargate
              ↓        ↓
           Tasks     Tasks
              ↓        ↓
         Containers  Containers

```

---

# ASG scaling policies

## Target Tracking

Maintain a metric around a target.

```text
Target CPU = 40%
```

> Keep CPU around a specific value → **Target Tracking**

## Step Scaling

Different actions at different thresholds.

```text
CPU 70% → +1
CPU 90% → +3
```

> Different scaling actions by threshold → **Step Scaling**

## Scheduled Scaling

For predictable traffic.

```text
09:00 → scale out
18:00 → scale in
```

> Predictable schedule → **Scheduled Scaling**

## Predictive Scaling

Uses historical patterns to forecast demand.

```text
Historical traffic
       ↓
Forecast
       ↓
Scale ahead of demand
```

> Predict future demand → **Predictive Scaling**

---

# ASG Lifecycle Hooks

Lifecycle hooks pause an instance during launch or termination.

### Launch

```text
Pending
   ↓
Pending:Wait
   ↓
Custom action
   ↓
Pending:Proceed
   ↓
InService
```

> Custom setup before instance enters service → **`Pending:Wait`**

### Termination

```text
Terminating
   ↓
Terminating:Wait
   ↓
Custom action
   ↓
Terminating:Proceed
   ↓
Terminated
```

> Collect logs / cleanup before termination → **`Terminating:Wait`**

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
Collect logs
 ↓
Complete lifecycle action
 ↓
Terminate
```

---

# ASG health checks

By default, ASG uses **EC2 health checks**.

An instance can be:

```text
EC2 = running ✅
Application = broken ❌
```

ASG may still consider it healthy.

Enable **ELB health checks** when the application itself must determine instance health.

```text
ALB
 ↓
Unhealthy target
 ↓
ASG
 ↓
Replace instance
```

---

# ASG + SQS

For EC2 workers processing SQS jobs:

```text
Application
    ↓
   SQS
    ↓
Workers
```

Scale based on queue metrics such as **backlog per instance**.

> More pending jobs → more worker capacity.

---

# ASG Termination Policy

When an ASG scales in, AWS must decide which instance to terminate.

## Default termination behavior

The default policy first tries to **maintain AZ balance**.

```text
AZ-A = 10
AZ-B = 8
AZ-C = 7

Scale in
   ↓
Consider AZ-A first
```

Then it prefers instances with **outdated/older launch configuration or launch template versions**.

If candidates are still tied, it uses the **instance closest to the next billing hour**.

### Default shortcut

```text
DEFAULT
   ↓
AZ with MOST instances
   ↓
Outdated configuration
   ↓
Closest to next billing hour
```

### Important distinction

Do not confuse:

```text
Oldest configuration
```

with:

```text
Oldest running instance
```

**OldestInstance** is a separate termination policy that specifically prefers the instance that has been running the longest.

### Useful termination policies

| Policy                        | Meaning                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------- |
| **Default**                   | Maintain AZ balance + prefer outdated configuration + billing-hour criterion |
| **OldestInstance**            | Terminate oldest running instance                                            |
| **NewestInstance**            | Terminate newest running instance                                            |
| **OldestLaunchConfiguration** | Prefer instances using oldest launch configuration                           |
| **ClosestToNextInstanceHour** | Prefer instance closest to next billing hour                                 |

---

# Route 53 + Load Balancer + ASG

Different layers solve different problems:

```text
Route 53
→ DNS-level routing

Load Balancer
→ stop traffic to unhealthy targets

ASG
→ replace failed instances
```

### Route 53 policies

| Requirement                 | Policy                    |
| --------------------------- | ------------------------- |
| Split traffic by percentage | **Weighted**              |
| Primary + standby           | **Failover**              |
| Lowest-latency Region       | **Latency-Based Routing** |
| User location               | **Geolocation**           |
| Geographic distance + bias  | **Geoproximity**          |

Memory:

```text
50/50
→ Weighted

Primary + Backup
→ Failover

Lowest latency
→ Latency
```

---

# Common question patterns

> `/api/*` → one target group, `/images/*` → another
> → **ALB path-based routing**

> Route based on hostname
> → **ALB host-based routing**

> UDP + fixed IP
> → **NLB**

> Very high TCP connection volume
> → **NLB**

> Firewall / IDS / IPS
> → **GWLB**

> Split traffic between two application versions
> → **Weighted Target Groups**

> Route users to the lowest-latency Region
> → **Route 53 Latency-Based Routing**

> Gradually migrate HTTP application from on-premises to AWS
> → **ALB Weighted Target Groups** or **Route 53 Weighted Routing**

> Primary + standby application
> → **Route 53 Failover**

> Load balancer says unhealthy but ASG does not replace instance
> → **Enable ELB health checks**

> Traffic increases predictably every morning
> → **Scheduled Scaling**

> Keep CPU around 40%
> → **Target Tracking**

> Different scaling actions at different thresholds
> → **Step Scaling**

> Predict future demand
> → **Predictive Scaling**

> Multiple HTTPS certificates on one ALB
> → **SNI**

> Errors when instances are removed during scale-in
> → **Deregistration delay**

> Original client IP behind ALB
> → **X-Forwarded-For**

> Fixed load balancer IP
> → **NLB**

> Scale workers based on pending SQS jobs
> → **SQS queue metrics**

> Terminate oldest running instance
> → **OldestInstance policy**

> Collect logs before termination
> → **Lifecycle hook + `Terminating:Wait`**

> Perform custom setup before launch completes
> → **Lifecycle hook + `Pending:Wait`**
> Default ASG scale-in termination behavior
> → **Most-populated AZ → outdated configuration → closest to next billing hour**
---

# Pocket Card

| Keyword                               | Answer                                                 |
| ------------------------------------- | ------------------------------------------------------ |
| Path / host / header routing          | **ALB**                                                |
| HTTP / HTTPS / gRPC                   | **ALB**                                                |
| WAF                                   | **ALB**                                                |
| Weighted Target Groups                | **ALB or NLB**                                         |
| UDP                                   | **NLB**                                                |
| Static IP / Elastic IP                | **NLB**                                                |
| Very high performance                 | **NLB**                                                |
| Firewall / IDS / IPS                  | **GWLB**                                               |
| GENEVE 6081                           | **GWLB**                                               |
| Multiple HTTPS certificates           | **SNI**                                                |
| Client IP behind ALB                  | **X-Forwarded-For**                                    |
| Scale-in connection errors            | **Deregistration delay**                               |
| Uneven AZ traffic                     | **Cross-zone load balancing**                          |
| Keep CPU at X%                        | **Target Tracking**                                    |
| Different scaling steps               | **Step Scaling**                                       |
| Predictable traffic schedule          | **Scheduled Scaling**                                  |
| Forecast future demand                | **Predictive Scaling**                                 |
| Application health should replace EC2 | **ELB health checks on ASG**                           |
| Scale workers by jobs                 | **SQS metrics**                                        |
| Oldest running instance               | **OldestInstance**                                     |
| Default ASG termination               | **Most-populated AZ → outdated config → billing hour** |
| Before instance enters service        | **`Pending:Wait`**                                     |
| Before instance termination           | **`Terminating:Wait`**                                 |
| Lowest-latency Region                 | **Route 53 Latency**                                   |
| Percentage traffic split              | **Route 53 Weighted**                                  |
| Primary + backup                      | **Route 53 Failover**                                  |
| Replace failed EC2                    | **ASG**                                                |
| ECS application scaling               | **Service Auto Scaling → tasks**                       |
| ECS EC2 capacity scaling              | **Capacity Provider / Cluster Auto Scaling**           |

---
### Trap

**Oldest configuration ≠ oldest running instance.**

> "Terminate the instance that has been running the longest."

→ **`OldestInstance` termination policy**
