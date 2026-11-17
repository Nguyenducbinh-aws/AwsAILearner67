---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cost Guard for Windows Labs on AWS
## A Serverless Three-Layer Credit Protection Solution

### 1. Executive Summary

Cost Guard is a serverless automation system built for students and small teams running Windows lab machines on AWS Free Plan accounts. Because Windows EC2 instances are billed by the hour, a forgotten session can drain a limited credit budget in a matter of days. This project addresses that problem through three automated protection layers — a scheduled stop rule, a near-real-time cost meter, and an AWS Budgets backstop — using only services available within the AWS Free Tier.

### 2. Problem Statement

#### What's the Problem?

AWS Free Plan credits are limited. A Windows EC2 instance left running unattended can consume a significant portion of a small budget over just a few days. While AWS Budgets provides cost alerts, its billing data only refreshes a few times per day, making it too slow to serve as a real-time safeguard.

#### The Solution

Cost Guard uses three complementary layers of protection:

- **Layer 1 — Schedule Guard:** An EventBridge Scheduler rule automatically stops the Windows instance outside of designated lab hours. This layer does not depend on billing data and is the most reliable.
- **Layer 2 — Cost Meter (original contribution):** A Lambda function runs periodically to estimate the current session cost based on instance running time and a configured price per hour. If the estimate exceeds an internal threshold, the instance is stopped and an alert is sent via SNS.
- **Layer 3 — Budget Backstop:** An AWS Budgets alarm monitors actual account spend. When the budget limit is approached, it triggers a Lambda function via SNS to stop the instance as a final safety net.

### Benefits and Return on Investment

- Prevents unplanned credit loss from forgotten or idle instances.
- Layer 2 provides cost awareness within minutes of a machine starting, without waiting for billing data to refresh.
- All supporting services (Lambda, EventBridge, DynamoDB, SNS, Parameter Store) remain within Free Tier limits under normal lab usage.
- The price per hour and cost threshold are stored in Parameter Store rather than hardcoded, making the system reusable across different instance types or accounts.

### 3. Solution Architecture

The system runs within a single AWS Region and relies entirely on serverless or managed services. EventBridge drives scheduling and periodic invocations. Lambda functions handle the stop/start logic and cost estimation. DynamoDB records session activity. SNS delivers notifications. Parameter Store holds configuration values. IAM roles are scoped to the minimum permissions each function needs.

![Architecture Diagram](/images/2-Proposal/architecture.png)

### AWS Services Used

- **Amazon EC2 (Windows)**: The lab machine being protected; stopped automatically by all three layers.
- **Amazon EventBridge**: Triggers scheduled stop/start actions and periodic cost meter runs.
- **AWS Lambda**: Implements the logic for each protection layer.
- **Amazon DynamoDB**: Stores instance activity logs and cost estimates.
- **Amazon SNS**: Sends email alerts when thresholds are crossed or errors occur.
- **AWS Budgets**: Monitors actual account spend and activates the final protection layer.
- **AWS Systems Manager Parameter Store**: Stores the hourly price and cost threshold without hardcoding them in Lambda code.
- **Amazon CloudWatch**: Collects Lambda logs and raises alarms on function errors.
- **AWS IAM**: Provides least-privilege execution roles for each Lambda function.
- **AWS CloudFormation**: Packages the full stack for repeatable deployment and clean teardown (optional).

### Component Design

- **Scheduler Layer**: EventBridge Scheduler → `ScheduleHandler` Lambda → stop or start the EC2 instance on a fixed schedule.
- **Cost Meter**: EventBridge rule (periodic) → `CostMeter` Lambda → reads price and threshold from Parameter Store → calculates estimated cost from session data in DynamoDB → if threshold exceeded: stops instance and publishes SNS alert.
- **Budget Backstop**: AWS Budgets alarm → SNS topic → `BudgetBackstop` Lambda → stops instance → sends confirmation email.
- **Notifications**: One SNS topic for user-facing cost alerts; one for operational errors.
- **Data**: DynamoDB table logging instance start/stop events and periodic cost estimates per session.
- **Security**: Each Lambda has its own IAM role with permissions limited to only the resources it touches.

### 4. Technical Implementation

**Implementation Phases**

- **Phase 1 — Foundation:** Set up base infrastructure — IAM roles, SNS topics, DynamoDB table, Parameter Store entries, and an AWS Budget. Determine instance type and region; use AWS Pricing Calculator to get the actual hourly price.
- **Phase 2 — Layer 1:** Launch the Windows EC2 instance, deploy `ScheduleHandler`, configure the EventBridge Scheduler rule, and verify that stop/start works correctly.
- **Phase 3 — Layer 2:** Deploy `CostMeter`, set up the periodic EventBridge rule, and test the threshold logic using simulated cost data.
- **Phase 4 — Layer 3 + Observability:** Configure the Budgets alarm, deploy `BudgetBackstop`, add CloudWatch Alarms on Lambda errors, and run an end-to-end test of all three layers.
- **Phase 5 — Demo and documentation:** Record a demo video while resources are live, clean up, capture a Cost Explorer screenshot, then write the workshop content.

**Technical Requirements**

- AWS account with Free Plan or sufficient remaining credits.
- AWS CLI configured with a least-privilege IAM user (no root keys).
- Python runtime for Lambda with `boto3` (available in the managed runtime).
- Familiarity with Lambda, EventBridge, SNS, DynamoDB, Parameter Store, and IAM.
- draw.io for the architecture diagram.
- Hugo for local site preview before publishing.

### 5. Timeline & Milestones

**Project Timeline**

| Day | Work | Deliverable |
|-----|------|-------------|
| 1 | Confirm instance type and region; run Pricing Calculator; create IAM, SNS, DynamoDB, Parameter Store, Budget | Base infrastructure ready |
| 2 | Launch Windows EC2; deploy Layer 1; test scheduled stop/start | Layer 1 working |
| 3 | Deploy CostMeter; test threshold and SNS alert | Layer 2 working |
| 4 | Deploy BudgetBackstop; add CloudWatch Alarms; full end-to-end test | All layers working |
| 5 | Record demo video; clean up all resources; capture Cost Explorer | Demo video + cleanup proof |
| 6 | Draw architecture diagram; write workshop sections 5.1–5.8 | Draft workshop content |
| 7 | Write Proposal; translate to English; preview with Hugo; push and verify | Complete site published |

### 6. Budget Estimation

Cost estimates are based on AWS Pricing Calculator for the chosen region, using on-demand pricing. The Windows instance is the dominant cost; all other services fall within Free Tier limits under normal usage.

A detailed cost breakdown was produced using AWS Pricing Calculator for the selected region and instance type. A PDF export of the estimate is attached for reference.

### Infrastructure Costs

| Service | Assumption | Estimated Monthly Cost |
|---------|------------|----------------------|
| EC2 Windows (small instance) | Limited to lab hours only | Low; dominant cost of the project |
| EBS root volume | Standard size, gp3 | Minimal |
| DynamoDB | Within Free Tier limits | $0.00 |
| Lambda | Within Free Tier limits | $0.00 |
| SNS | Within Free Tier limits | $0.00 |
| EventBridge | Within Free Tier limits | $0.00 |
| CloudWatch Logs | Within Free Tier limits | $0.00 |
| AWS Budgets | First 2 budgets free | $0.00 |
| Systems Manager Parameter Store | Standard parameters | $0.00 |
| **Total** | | **Under $20 for the full project** |

> The internal cost threshold in Layer 2 is set below the total budget (around 70%) to leave room for the billing lag inherent in Layer 3.

### 7. Risk Assessment

#### Risk Matrix

| Risk | Impact | Probability |
|------|--------|-------------|
| Forgetting to stop the instance manually | High | Medium |
| AWS Budgets alert arriving too late | High | High (by design) |
| Cost estimate in Layer 2 diverging from actual billing | Medium | Medium |
| Lambda stopping the wrong instance due to misconfigured IAM | High | Low |
| Chosen Windows instance type unavailable on Free Plan | Medium | Low |
| Credentials or account ID exposed in screenshots | Medium | Low |
| Falling behind schedule in a one-week window | Medium | Medium |

#### Mitigation Strategies

- **Forgotten instance:** Layer 1 stops the instance on a fixed schedule regardless of cost data.
- **Budgets delay:** Layer 2 acts within minutes based on running time and a configured price, without waiting for billing to refresh.
- **Cost estimate drift:** The internal threshold is set conservatively below the total budget. Actual vs. estimated cost is reconciled using Cost Explorer at the end of the project.
- **Wrong instance:** Each Lambda role's stop permission is scoped to the specific target instance ARN.
- **Instance type unavailable:** Instance selection is validated on Day 1; an alternative smaller type is used if needed.

#### Contingency Plans

- If Lambda layers fail: stop the instance manually from the AWS Console and disable the EventBridge rules.
- If credits run out mid-project: terminate the Windows instance immediately, then continue documentation using screenshots already captured.
- If the $20 budget is insufficient: reduce daily lab hours or switch to a smaller instance type and update the estimate accordingly.

### 8. Expected Outcomes

#### Technical Improvements

- The Windows EC2 instance stops automatically when lab hours end or when the cost estimate crosses the configured threshold — no manual intervention needed.
- Total project spend stays within the $20 budget, confirmed by Cost Explorer after cleanup.
- All three layers are observable through CloudWatch Logs, SNS email alerts, and the DynamoDB activity log.

#### Long-term Value

Because the instance ID, hourly price, and threshold are all stored in Parameter Store, the system can be reused for any Windows lab account without changing code. The three-layer pattern — fixed schedule, near-real-time cost meter, billing backstop — applies to any AWS environment where cost control matters and billing data latency is a limitation.


