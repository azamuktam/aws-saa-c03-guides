# Section 19: CloudWatch, CloudTrail & AWS Config

## The idea

Three services all "monitor" AWS, and the exam adores swapping them on you — this is **THE most-confused trio** on the test. Here's the split, and once you see it you can't unsee it.

Picture a **hospital**. **CloudWatch** is the bank of bedside monitors: heart rate, blood pressure, alarms when a value crosses a line — "**how is the patient doing?**" **CloudTrail** is the visitor logbook at the front desk: who came in, what they did, when, from where — "**WHO did WHAT?**" **AWS Config** is the patient's medical chart: what the patient's state *was* at any point in history, and whether it matches the doctor's orders — "**what did it look like at time T, and is it compliant?**"

Spell it out: **CloudWatch** = performance and health (**metrics**, **logs**, **alarms**). **CloudTrail** = **API audit** — every API call to AWS recorded. **AWS Config** = **configuration history + compliance** — snapshots of your resources' settings over time, checked against rules.

Read the question's verb: *slow/high/alert* → CloudWatch. *Who/deleted/changed by whom* → CloudTrail. *What did it look like / is it compliant* → Config.

## CloudWatch — "how is it doing?"

**Metrics.** Numbers over time. EC2 sends **CPU, network, and disk-ops by default** — but **MEMORY and disk-space usage are NOT default metrics**. To get those you install the **CloudWatch agent** (custom metrics). This is a classic trap: "alert when instance memory exceeds 80%" → the answer involves the **agent + custom metric**, never a default metric.

**Alarms.** Watch a metric, cross a threshold, take action: notify **SNS**, trigger **Auto Scaling**, or run **EC2 actions** (stop/terminate/recover). **Composite alarms** combine several alarms with AND/OR to **reduce alert noise** — "too many alarms firing at once" → composite alarm.

**Logs.** Log **groups** (per app) contain log **streams** (per instance/container). Retention configurable **1 day to 10 years** (default: forever). Three power features:

- **Metric filters**: turn a log pattern into a metric — "**count ERROR lines** and alarm when > 100/min" → metric filter + alarm.
- **Logs Insights**: SQL-ish **query language** for interactive log analysis.
- **Subscription filters**: stream log events in **real time** to **Kinesis or Lambda** — "process logs as they arrive" → subscription filter.

**Dashboards** are **cross-region** — one pane of glass for a global app.

## CloudTrail — "WHO did WHAT?"

Every API call — console click, CLI command, SDK call — is recorded: identity, time, source IP, parameters.

- **90-day event history is free** and automatic. For longer retention, create a **Trail → S3** (and make it a **multi-region trail** — a best practice and a favorite answer).
- **Management events** (control plane: create/terminate/modify resources) are logged **by default**. **DATA events** (S3 **object-level** reads/writes, Lambda invokes) are **opt-in and cost extra** — THE trap: *"who deleted the object in the bucket?"* → default CloudTrail can't say; you need **data events enabled**.
- **Log file integrity validation**: cryptographically prove logs weren't tampered with — the "forensics/court" keyword.
- **CloudTrail Insights**: detects **unusual API activity** (sudden spike in TerminateInstances).

## AWS Config — "what did it look like, and is it compliant?"

- A **configuration recorder** snapshots your resources' settings every time they change, giving you a **timeline** per resource — "what did this security group look like **last Tuesday**?" → Config.
- **Config rules** (AWS-**managed** or **custom** Lambda-backed) continuously evaluate compliance: "no security group may allow SSH from 0.0.0.0/0". Results roll up to a **compliance dashboard**.
- **Auto-remediation**: a noncompliant resource can trigger an **SSM Automation document** to fix itself — "automatically close world-open SSH" → **Config rule + remediation action**.

**THE trap: Config does NOT prevent anything.** It **detects** and (optionally) remediates *after the fact*. If the question says "**prevent** users from ever doing X" → that's **SCPs** (or IAM), not Config.

```
Question verb          →  Service
─────────────────────────────────
"is it slow/unhealthy?"→  CloudWatch
"who changed it?"      →  CloudTrail
"what was it / legal?" →  Config
```

**X-Ray one-liner:** distributed **tracing** across microservices — follows one request through every hop. *"Which microservice adds the latency?"* → **X-Ray** (not CloudWatch — CloudWatch tells you *a* service is slow; X-Ray tells you *which hop* in the request chain).

## Question patterns

> *"Determine who terminated a production EC2 instance last week."* → **CloudTrail** (API audit = who did what).

> *"Alert when application memory usage exceeds 80% on EC2."* → **CloudWatch agent + custom metric + alarm** (memory is not a default metric).

> *"Trigger an alarm when the app logs more than 100 ERROR lines in 5 minutes."* → **Metric filter on CloudWatch Logs + alarm**.

> *"Show the configuration of a security group as of last Tuesday, and flag any SG allowing SSH from the internet."* → **AWS Config** (timeline + managed rule).

> *"Automatically fix noncompliant security groups without manual work."* → **Config rule + SSM Automation remediation**.

> *"A request spans 10 microservices; find which one is slow."* → **X-Ray** (distributed tracing).

> *"Process log entries in real time as they are written."* → **CloudWatch Logs subscription filter → Kinesis/Lambda**.

> *"Identify who downloaded a specific S3 object."* → **CloudTrail data events** (object-level = opt-in, extra cost).

> *"Keep API activity records for 7 years, tamper-evident."* → **Trail → S3 + log file integrity validation** (free history is only 90 days).

> *"Prevent anyone in the org from disabling CloudTrail."* → **SCP** (Config detects; SCP prevents).

## Pocket card

| Keyword | Answer |
|---|---|
| Performance, health, alarms | CloudWatch |
| Who did what / API audit | CloudTrail |
| Config history / compliance | AWS Config |
| EC2 memory or disk-space metric | CloudWatch agent (not default) |
| Count log pattern → alarm | Metric filter |
| Query logs interactively | Logs Insights |
| Real-time log streaming | Subscription filter → Kinesis/Lambda |
| Too many alarms / noise | Composite alarm |
| S3 object-level "who" | CloudTrail data events (opt-in) |
| Logs beyond 90 days | Trail → S3, multi-region |
| Tamper-proof logs | Log file integrity validation |
| Unusual API spike | CloudTrail Insights |
| Auto-fix noncompliance | Config + SSM remediation |
| PREVENT actions | SCP (never Config) |
| Trace across microservices | X-Ray |

Now you know who's watching the house — next comes who holds the keys: KMS and CloudHSM.
