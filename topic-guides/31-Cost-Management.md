# Section 31: Cost Management Tools

## The idea

AWS provides several tools for understanding, controlling, and optimizing costs. The main difference is **what you want to do with the cost information**.

The exam usually gives you a verb or goal such as:

* **Analyze** spending → Cost Explorer
* **Get alerted** when spending reaches a threshold → AWS Budgets
* **Get detailed billing data** → Cost and Usage Report (CUR)
* **Detect unusual spending** → Cost Anomaly Detection
* **Rightsize resources** → Compute Optimizer
* **Track costs by team/project/department** → Cost Allocation Tags

## The main tools

| Tool                            | What it does                                                                                                                                                   | Typical exam keywords                                                                     |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Cost Explorer**               | Visualize, filter, and analyze historical AWS costs and usage. Can also forecast future costs and provide Reserved Instance / Savings Plans recommendations.   | "analyze", "visualize", "historical spending", "which service costs the most", "forecast" |
| **AWS Budgets**                 | Set cost or usage thresholds and send alerts when actual or forecasted spending exceeds them. Can also trigger configured budget actions.                      | "notify", "alert", "80% of budget", "exceeds budget"                                      |
| **Cost and Usage Report (CUR)** | Provides highly detailed billing and usage data, delivered to Amazon S3. The data can be analyzed with Athena, QuickSight, and other tools.                    | "most detailed", "line items", "granular billing data", "SQL", "S3"                       |
| **Cost Anomaly Detection**      | Uses machine learning to identify unusual spending patterns and send alerts.                                                                                   | "unexpected increase", "unusual spending", "anomaly", "spending spike"                    |
| **Compute Optimizer**           | Provides machine-learning-based recommendations for rightsizing supported AWS resources.                                                                       | "overprovisioned", "underutilized", "rightsize", "instance recommendation"                |
| **Cost Allocation Tags**        | Lets you categorize AWS costs using resource tags such as department, team, or project. Tags must be activated as cost allocation tags in the Billing console. | "cost by department", "cost by team", "cost by project", "chargeback"                     |

## The most important differences

### Cost Explorer

Use **Cost Explorer** when you want to **understand or analyze spending**.

Examples:

> Which AWS service caused last month's cost increase?

> How much did we spend on EC2?

> Show our historical spending by service.

> Forecast future spending.

Cost Explorer is mainly an **analysis and visualization tool**.

---

### AWS Budgets

Use **AWS Budgets** when you want to **set a threshold and receive an alert**.

Example:

> Notify the finance team when monthly spending reaches 80% of the budget.

You can create budgets based on cost or usage and configure alerts based on actual or forecasted values.

**Remember:**

> **Budgets = thresholds and alerts**

---

### Cost and Usage Report (CUR)

Use **CUR** when you need **very detailed billing and usage information**.

The report is delivered to **Amazon S3** and can be queried with services such as **Amazon Athena**.

Example:

> Finance needs detailed line-item billing data that analysts can query using SQL.

The answer is:

**CUR + S3 + Athena**

**Remember:**

> **CUR = detailed raw billing data**

---

### Cost Anomaly Detection

Use **Cost Anomaly Detection** when the requirement is to detect **unexpected or unusual spending**.

Example:

> Alert the finance team when AWS spending suddenly increases beyond its normal pattern.

Unlike AWS Budgets, the scenario does not necessarily require you to define a fixed threshold such as 80% or $10,000.

**Remember:**

> **Anomaly Detection = unusual spending**

---

### Compute Optimizer

Use **Compute Optimizer** when the problem is **resource sizing**.

Example:

> Several EC2 instances are overprovisioned. Recommend more appropriate instance types.

Compute Optimizer analyzes usage and provides rightsizing recommendations for supported resources.

**Remember:**

> **Compute Optimizer = rightsize resources**

---

### Cost Allocation Tags

Use **Cost Allocation Tags** when the company wants to **attribute costs to departments, teams, projects, or other groups**.

Example:

```text
Department=Engineering
Department=Marketing
Department=Finance
```

After the tags are activated as cost allocation tags, AWS can use them in cost and billing analysis.

Example question:

> Finance needs a monthly report showing total AWS spending for each department.

Answer:

**Tag resources with the department name and enable the tags as cost allocation tags.**

**Remember:**

> **Cost Allocation Tags = who is responsible for the cost**

## Common exam traps

### Budgets vs Cost Explorer

**"Analyze spending"** → Cost Explorer

**"Send an alert when spending reaches X%"** → AWS Budgets

Cost Explorer helps you understand spending; Budgets is designed for thresholds and alerts.

---

### Cost Allocation Tags vs Budgets

**"Track costs by department/team/project"** → Cost Allocation Tags

**"Alert when spending exceeds a limit"** → AWS Budgets

They solve different problems.

---

### CUR vs Cost Explorer

**"Visualize and analyze spending"** → Cost Explorer

**"Need highly detailed line-item billing data"** → CUR

If the question mentions **S3, Athena, SQL, or granular billing data**, CUR is usually the important clue.

---

### Cost Anomaly Detection vs Budgets

**"Unexpected/unusual spending spike"** → Cost Anomaly Detection

**"Spending reaches 80% of budget"** → AWS Budgets

The key difference is:

* **Budgets** → predefined threshold
* **Anomaly Detection** → unusual spending pattern

---

### Cost Explorer vs Savings Plans / Reserved Instances

Savings Plans and Reserved Instances are **pricing/commitment mechanisms**.

Cost Explorer can provide **purchase recommendations** based on your historical usage.

So:

> "How much Savings Plan commitment should we purchase?"

→ **Cost Explorer recommendations**

## Other useful tool

### Billing Conductor

Billing Conductor is used to create **customized billing views and rates**, especially when an organization needs customized internal billing or chargeback.

It is much less common than the other tools in typical SAA questions.

## Quick decision table

| Question wording                          | Answer                     |
| ----------------------------------------- | -------------------------- |
| Visualize or analyze AWS spending         | **Cost Explorer**          |
| Forecast spending                         | **Cost Explorer**          |
| Alert when spending reaches a threshold   | **AWS Budgets**            |
| Most detailed billing / line-item data    | **CUR**                    |
| Query billing data with SQL               | **CUR + Athena**           |
| Unexpected spending spike                 | **Cost Anomaly Detection** |
| Rightsize an EC2/resource                 | **Compute Optimizer**      |
| Track cost by team/department/project     | **Cost Allocation Tags**   |
| Savings Plan / RI purchase recommendation | **Cost Explorer**          |
| Customized internal billing views         | **Billing Conductor**      |

## Pocket Card

| Keyword                                      | Answer                 |
| -------------------------------------------- | ---------------------- |
| **Analyze / visualize / forecast**           | Cost Explorer          |
| **Alert / threshold / 80%**                  | AWS Budgets            |
| **Detailed / line items / SQL / S3**         | CUR + Athena           |
| **Unexpected / unusual / spike**             | Cost Anomaly Detection |
| **Rightsize / overprovisioned**              | Compute Optimizer      |
| **Department / team / project / chargeback** | Cost Allocation Tags   |
| **Savings Plan / RI recommendation**         | Cost Explorer          |
| **Custom billing views**                     | Billing Conductor      |

## The decision rule

**Analyze** → Cost Explorer
**Alert** → AWS Budgets
**Detailed billing data** → CUR
**Unusual spending** → Cost Anomaly Detection
**Rightsize** → Compute Optimizer
**Attribute costs to teams/departments** → Cost Allocation Tags
