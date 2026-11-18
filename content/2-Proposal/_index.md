---
title: "Proposal"
date: 2026-11-18
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cost Guard for Windows Labs on AWS
## A Serverless Three-Layer Credit Protection Solution

### 1. Executive Summary

Cost Guard is a serverless automation system for students and small teams who run Windows lab machines on an AWS Free Plan account. Windows EC2 instances are billed by the hour, so a machine left running for a few days can consume a large part of a small credit budget. The project addresses this with three automated protection layers: a scheduled stop, a near-real-time cost meter, and an AWS Budgets backstop. The supporting services all fit within the AWS Free Tier; the Windows EC2 instance is the only meaningful cost.

### 2. Problem Statement

***What's the Problem?***

AWS Free Plan credits are limited. A Windows EC2 instance (t3.medium) running 24/7 costs about $46.20 per month, which is more than a $20 credit budget. AWS Budgets can send cost alerts, but its billing data refreshes only a few times per day, so it is too slow to act as a real-time safeguard.

***The Solution***

Cost Guard uses three complementary layers:

- **Layer 1 — Schedule Guard:** An EventBridge Scheduler rule stops the instance outside lab hours. It does not depend on billing data, so it is the most reliable layer. The schedule only stops the machine; starting it stays a manual action so that no credit is spent while nobody is using it.
- **Layer 2 — Cost Meter (original contribution):** A Lambda function runs every 5 minutes. It estimates the cost of the running instance from its running time and a configured hourly price, and adds it to the estimated cost of earlier sessions in the current month. When the total crosses an internal threshold, the instance is stopped and an alert is sent through SNS.
- **Layer 3 — Budget Backstop:** AWS Budgets monitors the actual account spend. As spend approaches the limit, it triggers a Lambda function through SNS to stop the instance as the final safety net.

***Benefits and Return on Investment***

Under the planned usage of 120 lab hours per month, the estimated cost is about $9.60 per month. The same instance left running 24/7 would cost $46.20 per month. Cost Guard therefore avoids roughly $36.60 per month, about 79%, and keeps the project under the $20 budget. Layer 2 gives cost awareness within minutes of the machine starting, without waiting for billing data. The hourly price and the cost threshold are stored in Parameter Store instead of being hardcoded, so the system can be reused for other instance types or accounts.

### 3. Solution Architecture

The system runs in a single AWS Region (US East, N. Virginia) and uses only serverless or managed services. EventBridge drives the schedule and the periodic cost check. Lambda functions handle stop logic and cost estimation. DynamoDB records session activity. SNS delivers notifications. Parameter Store holds configuration. Each Lambda function has its own least-privilege IAM role.

![Architecture Diagram](../images/2-Proposal/architecture.png)

**AWS Services Used**

- **Amazon EC2 (Windows):** The lab machine being protected (t3.medium, Windows Server).
- **Amazon EBS (gp3, 30 GB):** The root volume of the lab machine. It is billed for the whole month, even while the instance is stopped.
- **Amazon EventBridge:** EventBridge Scheduler triggers the scheduled stop; an EventBridge rule runs the cost meter every 5 minutes.
- **AWS Lambda:** Runs the logic of each protection layer.
- **Amazon DynamoDB:** Stores session activity and cost estimates.
- **Amazon SNS:** Sends email alerts when a threshold is crossed or an error occurs.
- **AWS Budgets:** Monitors actual spend and triggers the final protection layer.
- **AWS Systems Manager Parameter Store:** Stores the hourly price, the cost threshold and the budget limit.
- **Amazon CloudWatch:** Collects Lambda logs and raises alarms on function errors.
- **AWS IAM:** Provides a least-privilege execution role for each Lambda function.
- **AWS CloudFormation:** Packages the stack for repeatable deployment and clean teardown.
- **AWS Cost Explorer:** Compares the estimated cost with the actual cost at the end of the project.

**Component Design**

- **Scheduler Layer:** EventBridge Scheduler → `ScheduleHandler` Lambda → stops the EC2 instance on a fixed schedule and writes the event to DynamoDB.
- **Cost Meter:** EventBridge rule (every 5 minutes) → `CostMeter` Lambda → reads price and threshold from Parameter Store → calculates the estimated cost from session records in DynamoDB → if the threshold is exceeded, stops the instance and publishes an SNS alert. The threshold applies to the EC2 running cost only; the EBS volume is accounted for separately in the budget.
- **Budget Backstop:** AWS Budgets alarm → SNS topic → `BudgetBackstop` Lambda → stops the instance → sends a confirmation email.
- **Notifications:** One SNS topic for cost alerts to the user; one SNS topic for operational errors.
- **Data:** A DynamoDB table that logs instance start and stop events and the periodic cost estimate of each session.
- **Security:** Each Lambda function has its own IAM role, and its stop permission is limited to the target instance.

**Business Logic**

- **Starting the machine:** The lab user starts the instance manually. No layer starts it automatically, so no credit is spent while nobody is working.
- **Layer 1 rule:** Every day at 22:00 (Asia/Ho_Chi_Minh), if the instance is running, stop it. If it is already stopped, do nothing.
- **Layer 2 rule:** Every 5 minutes, if the instance is running, estimate the cost of the current session (running hours × hourly price) and add the estimated cost of earlier sessions in the same calendar month. If the total reaches the threshold (about $14, 70% of the $20 budget), stop the instance, record the event and send an alert. Otherwise, only record the estimate. If the instance is stopped, do nothing.
- **Restarting after a Layer 2 stop:** The monthly total is still above the threshold, so the next run stops the instance again within 5 minutes. The machine is effectively locked until the next calendar month, when the count starts again.
- **Layer 3 rule:** AWS Budgets tracks the actual spend of the account and sends a notification at 80% and at 100% of the $20 limit. Each notification triggers `BudgetBackstop`, which stops the instance and sends a confirmation email. Billing data refreshes only a few times per day, so this layer is slow by design and acts only as the last safety net.
- **Order of protection:** Layer 1 is fixed by time, Layer 2 reacts within minutes, and Layer 3 follows actual billing. Any layer can stop the machine, none can start it, and stopping an already stopped machine does nothing, so the layers do not conflict.
- **Notifications:** Every automatic stop sends one email through the alerts topic. Function errors raise a CloudWatch alarm and an email through the errors topic.
- **Failure behavior:** If one layer fails, the other two stay active, so a single failure does not leave the machine unprotected.
- **Cost scope:** The Layer 2 threshold covers the EC2 running cost only. The EBS volume ($2.40 per month) is a fixed cost counted separately in the budget.
- **Configuration:** The hourly price, the threshold and the budget limit are values in Parameter Store. Changing them takes effect on the next run without redeploying.

### 4. Technical Implementation

**Implementation Phases**

1. **Foundation:** Create the IAM roles, SNS topics, DynamoDB table, Parameter Store entries and the AWS Budget. Confirm the instance type and region, and read the hourly price from the AWS Pricing Calculator.
2. **Layer 1:** Launch the Windows EC2 instance, deploy `ScheduleHandler`, configure the EventBridge Scheduler rule and verify that the scheduled stop works.
3. **Layer 2:** Deploy `CostMeter`, configure the periodic EventBridge rule and test the threshold logic with simulated cost data.
4. **Layer 3 and observability:** Configure the Budgets alarm, deploy `BudgetBackstop`, add CloudWatch alarms on Lambda errors and run an end-to-end test of all three layers.
5. **Demo and documentation:** Record a demo while the resources are live, clean up, compare costs in Cost Explorer and write the workshop.

### 5. Timeline & Milestones

**Project Timeline**

- **Month 0 (before the internship):** Join the FCAJ program, read the internship rules and prepare the working environment.
- **Month 1 (11/09 – 10/10/2026):** Learn the AWS foundations: account setup, MFA, IAM, Budgets, Support, Console and CLI.
- **Month 2 (11/10 – 10/11/2026):** Study the services used by the project (Lambda, EventBridge, SNS, DynamoDB, Parameter Store, CloudWatch) and design the three layers.
- **Month 3 (11/11 – 14/12/2026):** Build and document the solution.
  - Week 1: Foundation and Layer 1 (confirm instance type and price, create base resources, scheduled stop).
  - Week 2: Layer 2 (cost meter and threshold test) and the architecture diagram.
  - Week 3: Layer 3, CloudWatch alarms and the end-to-end test.
  - Week 4: Demo recording, cleanup, Cost Explorer comparison, workshop writing and publishing the site.
- **Post-deployment:** Keep the solution as a reusable template; extend it to other instance types or accounts by changing the Parameter Store values.

### 6. Budget Estimation

The estimate was built with the AWS Pricing Calculator for US East (N. Virginia), using on-demand pricing and a planned usage of 120 lab hours per month (4 hours per day for 30 days).

<!-- TODO: add the AWS Pricing Calculator share link and the exported PDF (static/attachments/budget_estimation.pdf), then reference them here. -->

**Infrastructure Costs**

- Amazon EC2 Windows (t3.medium): 120 hours × $0.06 per hour = $7.20 per month.
- Amazon EBS (gp3, 30 GB): $2.40 per month, billed for the whole month even when the instance is stopped.
- Public IPv4 address (if the instance uses one): about $0.60 per month for 120 hours. This item is not in the current Calculator estimate and needs to be added.
- AWS Lambda, Amazon DynamoDB, Amazon SNS, Amazon EventBridge, Amazon CloudWatch, Parameter Store (standard parameters), AWS Budgets: $0.00 per month within the Free Tier.

**Total:** about $9.60 per month (about $10.20 with the public IPv4 address), which is under the $20 budget.

For comparison, the same instance running 24/7 (730 hours) costs $46.20 per month: $43.80 for EC2 and $2.40 for EBS.

The internal threshold of Layer 2 is set at about 70% of the budget ($14). At $0.06 per hour this equals about 233 running hours, well above the planned 120 hours. The gap leaves room for the billing delay of Layer 3.

### 7. Risk Assessment

**Risk Matrix**

- Forgetting to stop the instance manually: high impact, medium probability.
- AWS Budgets alert arriving too late: high impact, high probability (by design).
- Layer 2 estimate diverging from actual billing: medium impact, medium probability.
- Lambda stopping the wrong instance because of a misconfigured IAM role: high impact, low probability.
- The chosen Windows instance type not available on the Free Plan: medium impact, low probability.
- Credentials or the account ID exposed in screenshots: medium impact, low probability.
- Falling behind schedule in the last month of the internship: medium impact, medium probability.

**Mitigation Strategies**

- Forgotten instance: Layer 1 stops the instance on a fixed schedule, regardless of cost data.
- Budgets delay: Layer 2 acts within minutes using running time and a configured price.
- Estimate drift: the threshold is set conservatively below the budget, and the estimate is compared with Cost Explorer at the end of the project.
- Wrong instance: each Lambda role can stop only the target instance.
- Unavailable instance type: the instance is validated in the first week, and a smaller type is used if needed.
- Exposed credentials: blur the account ID and any keys in every screenshot before publishing.
- Schedule delay: write the workshop while building each layer instead of leaving it to the end, and build the layers in order of importance.

**Contingency Plans**

- If a Lambda layer fails, stop the instance manually from the AWS Console and disable the EventBridge rules.
- If credits run out during the project, terminate the Windows instance immediately and finish the documentation with the screenshots already captured.
- If the $20 budget is not enough, reduce the daily lab hours or switch to a smaller instance type and update the estimate.

### 8. Expected Outcomes

**Technical Improvements**

- The Windows EC2 instance stops automatically when lab hours end or when the estimated cost crosses the threshold, with no manual action.
- Total spend stays within the $20 budget, confirmed by Cost Explorer after cleanup.
- All three layers can be observed through CloudWatch Logs, SNS email alerts and the DynamoDB activity log.

**Long-term Value**

Because the instance ID, hourly price and threshold are stored in Parameter Store, the system can be reused for any Windows lab account without changing code. The three-layer pattern (a fixed schedule, a near-real-time cost meter and a billing backstop) applies to any AWS environment where cost control matters and billing data latency is a limitation.
