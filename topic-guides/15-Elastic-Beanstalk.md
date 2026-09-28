# Section 15: Elastic Beanstalk

## The idea

**Elastic Beanstalk (EB)** is an application deployment service that provisions and manages the infrastructure needed to run your application.

You upload code—supported platforms include **Java, Python, Node.js, .NET, Go, Ruby, PHP, and Docker**—and Beanstalk can provision:

* **EC2 instances**
* **Auto Scaling Group**
* **Load Balancer**
* **CloudWatch monitoring**

You can still inspect and customize the underlying resources.

### Two exam-critical facts

* **EB is an orchestrator, not a black box** → you retain **full control of the underlying resources**.
* **EB itself is free** → you pay for the AWS resources it creates, such as EC2 and ALB.

Configuration customizations can be defined in **`.ebextensions`** YAML/JSON files in the source bundle.

### Worker environment

A **Worker environment** does not serve normal web traffic. It pulls background jobs from an **SQS queue**.

```text
Web environment
→ serves application requests

Worker environment
→ consumes SQS jobs
→ performs background processing
```

---

## App Runner

**AWS App Runner** is a fully managed service for deploying **web applications and APIs** directly from:

* source code repositories
* container images

AWS handles the underlying infrastructure, deployment, load balancing, and automatic scaling.

Use App Runner when the requirement is:

> **"Deploy a web application/API without managing servers or infrastructure."**

### App Runner vs Elastic Beanstalk

| Requirement                           | App Runner                   | Elastic Beanstalk                                |
| ------------------------------------- | ---------------------------- | ------------------------------------------------ |
| Deploy web app/API quickly            | ✅                            | ✅                                                |
| Manage servers/infrastructure for you | ✅                            | ✅                                                |
| Underlying infrastructure control     | Limited                      | **Full control**                                 |
| Deploy source code                    | ✅                            | ✅                                                |
| Deploy container image                | ✅                            | ✅                                                |
| Built-in load balancing/scaling       | ✅                            | ✅                                                |
| Typical use                           | Simple managed web apps/APIs | Applications needing more infrastructure control |

### Exam signal

> **"Deploy a web app/API with minimal infrastructure management."** → **App Runner**

> **"Deploy quickly without managing infrastructure but retain control of the underlying resources."** → **Elastic Beanstalk**

Do not confuse App Runner with **Run Command**:

* **App Runner** → deploy/run an application
* **Systems Manager Run Command** → configure/manage existing EC2 instances without SSH/RDP

---

## THE tested table: deployment policies

| Policy                            | How it works                                                        | Downtime? | Capacity during deploy               | Extra cost?             | Rollback                     | Pick when...                                   |
| --------------------------------- | ------------------------------------------------------------------- | --------- | ------------------------------------ | ----------------------- | ---------------------------- | ---------------------------------------------- |
| **All at once**                   | Update every instance simultaneously                                | **YES**   | Drops to zero briefly                | No                      | Redeploy old version (slow)  | Fastest; **dev/test, downtime OK**             |
| **Rolling**                       | Update in batches, batch by batch                                   | No        | **Reduced** (a batch is always down) | **No**                  | Slow (roll batches back)     | **No downtime + no extra cost**                |
| **Rolling with additional batch** | Spin up one extra batch first, then roll                            | No        | **FULL capacity maintained**         | Small (one extra batch) | Slow                         | Can't afford reduced capacity                  |
| **Immutable**                     | Build a **whole new fleet** alongside the old, swap when healthy    | No        | Full                                 | Double, briefly         | **Safest: delete new fleet** | Production, **safest rollback**                |
| **Blue/Green**                    | Clone the entire **environment**, test it, then **swap DNS CNAMEs** | No        | Full                                 | Double while both exist | **Instant: swap CNAME back** | Test new version with real URL, instant switch |

### Constraint matching

* **Fastest, downtime acceptable** → **All at once**
* **No downtime, no additional cost** → **Rolling**
* **Must maintain full capacity** → **Rolling with additional batch**
* **Safest / easiest rollback if instances fail** → **Immutable**
* **Test new version, then instant cutover / instant rollback via DNS** → **Blue/Green (CNAME swap)**

---

## THE RDS trap

If you create an **RDS database inside a Beanstalk environment**, its lifecycle is tied to the environment.

```text
Terminate/rebuild EB environment
        ↓
RDS created inside environment
        ↓
Database can be deleted with environment
```

### Production rule

Keep RDS **outside the Beanstalk environment** and provide the connection string through environment variables.

```text
Production
[EB Environment]
EC2 + ASG + ALB
       │
       └── environment variables
                   ↓
                [RDS]
             independent
              lifecycle
```

### Migration path

If RDS is already inside the environment:

```text
RDS snapshot
   ↓
Restore as standalone RDS
   ↓
Point Beanstalk to standalone RDS
   ↓
Use environment variables
   ↓
Enable deletion protection
```

---

## When Beanstalk is the distractor

| Question really wants                                            | Correct answer     |
| ---------------------------------------------------------------- | ------------------ |
| Fine-grained, repeatable **infrastructure as code**              | **CloudFormation** |
| **Microservices at scale** / container orchestration             | **ECS / EKS**      |
| **Event-driven**, sub-15-minute functions                        | **Lambda**         |
| Fully managed web app/API with minimal infrastructure management | **App Runner**     |

Beanstalk is for:

> **Classic application deployment where developers want speed but still need control of the underlying resources.**

---

## Question patterns

> *"Developers want to deploy code quickly without managing infrastructure but must retain full control of the underlying resources"*
> → **Elastic Beanstalk**

> *"Deploy a web application or API without managing servers or infrastructure"*
> → **App Runner**

> *"Deploy new version as fast as possible; brief downtime is acceptable (dev environment)"*
> → **All at once**

> *"Deploy with no downtime and no additional cost"*
> → **Rolling**

> *"Deploy with no downtime while maintaining full capacity"*
> → **Rolling with additional batch**

> *"Deployment must be safest possible with quick rollback if health checks fail"*
> → **Immutable**

> *"Test the new version against a separate URL, then switch all traffic instantly with instant rollback"*
> → **Blue/Green with CNAME swap**

> *"Beanstalk app's database was lost when the environment was terminated — prevent this in production"*
> → **Create RDS outside the environment; connect via environment variables**

> *"Migrate a database out of an existing Beanstalk environment"*
> → **Snapshot the RDS instance → restore as standalone RDS**

> *"Long-running background jobs from a queue alongside a Beanstalk web app"*
> → **Worker environment** (SQS-consuming tier)

> *"Customize packages and configuration of Beanstalk instances at deploy time"*
> → **`.ebextensions`**

> *"Existing EC2 instances must be configured without SSH/RDP"*
> → **Systems Manager Run Command**, not App Runner

---

## Pocket card

| Keyword                                                | Answer                            |
| ------------------------------------------------------ | --------------------------------- |
| Quickly deploy, no infra management, **KEEP control**  | **Elastic Beanstalk**             |
| Managed web app/API, minimal infrastructure management | **App Runner**                    |
| Fastest deploy, downtime OK                            | **All at once**                   |
| No downtime, no extra cost                             | **Rolling**                       |
| No downtime, full capacity                             | **Rolling + additional batch**    |
| Safest rollback (delete new fleet)                     | **Immutable**                     |
| DNS CNAME swap, instant rollback                       | **Blue/Green**                    |
| DB tied to environment                                 | **Create RDS OUTSIDE env**        |
| Move DB out of env                                     | **Snapshot → standalone RDS**     |
| SQS-driven background tier                             | **Worker environment**            |
| Instance/env customization files                       | **`.ebextensions`**               |
| Beanstalk pricing                                      | **Free — pay for resources only** |
| Fine-grained IaC                                       | **CloudFormation**                |
| Fully managed web app/API                              | **App Runner**                    |
| Existing EC2 configuration without SSH/RDP             | **Systems Manager Run Command**   |

Beanstalk fronts web applications with a load balancer automatically. When an application needs a dedicated API front door and API-management features, the next topic is **API Gateway**.
