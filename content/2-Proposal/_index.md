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

Cost Guard is a serverless protection system designed for students and small teams running Windows lab machines on AWS Free Plan accounts. A Windows EC2 instance is billed by the hour, so a single forgotten session can drain a $20 credit budget within days. Cost Guard addresses this through three automated layers: a scheduled stop/start rule, a near-real-time cost meter built on Lambda and DynamoDB, and an AWS Budgets backstop. Together, they keep total spending under the $20 limit without requiring manual monitoring. All orchestration uses EventBridge, Lambda, SNS, and Systems Manager Parameter Store — services that fall within AWS Free Tier limits.

### 2. Problem Statement

#### What's the Problem?

AWS Free Plan credits are finite. A Windows EC2 instance costs roughly $0.02–$0.05 per hour depending on instance type and region, so a machine left running over a weekend can consume $1–$3 — a material fraction of a $20 budget. AWS Budgets does provide cost alerts, but its billing data refreshes at most three times per day (every 8–12 hours), making it too slow to act as a real-time circuit breaker.

#### The Solution

Cost Guard uses three complementary layers:

- **Layer 1 — Schedule Guard:** An EventBridge Scheduler rule stops the Windows instance outside lab hours and optionally restarts it on demand. This requires no billing data at all and is the most reliable layer.
- **Layer 2 — Cost Meter (original contribution):** A Lambda function (`CostMeter`) runs every 15 minutes. It reads the per-hour price from Systems Manager Parameter Store, multiplies by the accumulated running time recorded in DynamoDB, and compares the result against an internal threshold (set at ~70% of the $20 budget). If the threshold is crossed, it stops the instance and publishes an SNS alert. This layer operates near-real-time without waiting for billing data.
- **Layer 3 — Budget Backstop:** An AWS Budgets alarm covers the full account at $20. When triggered, it publishes to an SNS topic, which invokes a Lambda function (`BudgetBackstop`) to stop the instance and send a final email alert. This layer uses actual billing figures and serves as a last resort.

### Benefits and Return on Investment

- Prevents unplanned credit loss from forgotten instances.
- Layer 2 provides cost visibility within 15 minutes of a machine starting — compared to 8–12 hours with Budgets alone.
- Estimated monthly infrastructure cost: **~$0.10–$0.30/month** (see Budget Estimation section); the Windows instance itself is the dominant cost and is controlled by the system.
- Over 12 months, the system cost is approximately **$1.20–$3.60**, while protecting a $20 budget that would otherwise be exhausted in days if unguarded.
- Reusable across any lab account; the per-hour price and threshold are stored as parameters, not hardcoded.

### 3. Solution Architecture

The architecture spans one AWS Region and uses only serverless or low-cost managed services. EventBridge Scheduler drives Layer 1 on a cron schedule. A separate EventBridge rule triggers `CostMeter` every 15 minutes. AWS Budgets monitors actual billing and triggers Layer 3 via SNS when the account total approaches $20. All Lambda functions log to CloudWatch Logs; a CloudWatch Alarm on Lambda errors publishes to a separate SNS topic for operational alerts. IAM roles follow least privilege: each Lambda has permissions scoped to exactly the resources it needs.

![Architecture Diagram](/images/2-Proposal/architecture.png)

*Diagram to be drawn using draw.io with official AWS architecture icons. See Section 4 for component detail.*

### AWS Services Used

- **Amazon EC2 (Windows, t3.micro or t2.micro)**: The protected lab machine; stopped automatically by all three layers.
- **Amazon EventBridge Scheduler**: Triggers scheduled stop/start actions (Layer 1) and periodic CostMeter invocations (Layer 2).
- **AWS Lambda**: Three functions — `ScheduleHandler` (Layer 1), `CostMeter` (Layer 2), `BudgetBackstop` (Layer 3).
- **Amazon DynamoDB**: Stores instance start/stop events and running cost estimates per session.
- **Amazon SNS**: Delivers email alerts for cost threshold breaches and Lambda errors.
- **AWS Budgets**: Monitors actual account spend and triggers Layer 3 when the $20 limit is approached.
- **AWS Systems Manager Parameter Store**: Holds the per-hour instance price and internal cost threshold — no hardcoded values in Lambda code.
- **Amazon CloudWatch (Logs + Alarms)**: Collects Lambda execution logs; raises alarms on function errors.
- **AWS IAM**: Provides least-privilege execution roles for each Lambda function.
- **AWS CloudFormation (optional)**: Packages the full stack as infrastructure-as-code for repeatable deployment and clean teardown.

### Component Design

- **Scheduler Layer**: EventBridge Scheduler cron rule → `ScheduleHandler` Lambda → `ec2:StopInstances` / `ec2:StartInstances` on the target instance ID.
- **Cost Meter**: EventBridge rule (rate 15 minutes) → `CostMeter` Lambda → reads `price_per_hour` and `cost_threshold` from Parameter Store → reads last start timestamp from DynamoDB → calculates estimated cost → writes record to DynamoDB → if cost ≥ threshold: stops instance + publishes to SNS alert topic.
- **Budget Backstop**: AWS Budgets alarm → SNS topic → `BudgetBackstop` Lambda → stops instance → publishes confirmation to SNS email topic.
- **Notification**: One SNS topic for user-facing email alerts; one SNS topic for operational (Lambda error) alerts.
- **Data (DynamoDB)**: Table `CostGuardLog` with partition key `instanceId` and sort key `timestamp`; attributes: `action` (start/stop/meter), `estimatedCostUSD`, `runningMinutes`.
- **Security (IAM)**: Three Lambda execution roles. `ScheduleHandler` role: `ec2:StopInstances`, `ec2:StartInstances`, `ec2:DescribeInstances` scoped to the target instance ARN; `logs:CreateLogGroup`, `logs:PutLogEvents`. `CostMeter` role: adds `dynamodb:PutItem`, `dynamodb:GetItem` on the `CostGuardLog` table and `ssm:GetParameter` on the two parameter paths. `BudgetBackstop` role: `ec2:StopInstances` on the target instance and `sns:Publish` on the alert topic.

### 4. Technical Implementation

**Implementation Phases**

- **Phase 1 — Pricing and design (Day 1):** Determine the exact instance type and region; use AWS Pricing Calculator to get the on-demand per-hour price; set the $20 budget split; create IAM roles, SNS topics, DynamoDB table, and Parameter Store entries.
- **Phase 2 — Layer 1 (Day 2):** Launch the Windows EC2 instance; write and deploy `ScheduleHandler`; configure EventBridge Scheduler; verify stop/start by manual trigger.
- **Phase 3 — Layer 2 (Day 3):** Write and deploy `CostMeter`; configure 15-minute EventBridge rule; test threshold logic with a simulated high cost value in DynamoDB; verify SNS email and instance stop.
- **Phase 4 — Layer 3 + observability (Day 4):** Configure AWS Budgets alarm; write and deploy `BudgetBackstop`; add CloudWatch Alarms on Lambda error metrics; end-to-end test with all three layers active.
- **Phase 5 — Demo, cleanup, and documentation (Days 5–7):** Record demo video while resources are live; run `terraform destroy` or delete CloudFormation stack; capture Cost Explorer screenshot; write workshop content and Proposal.

**Technical Requirements**

- AWS account with Free Plan or remaining credits ≥ $5 at the start.
- AWS CLI v2 configured with an IAM user (least-privilege, no root keys).
- Python 3.12 runtime for Lambda; `boto3` available in the managed runtime (no extra layer needed).
- Working knowledge of: Lambda, EventBridge, SNS, DynamoDB, Parameter Store, IAM policy syntax.
- draw.io (desktop or web) for the architecture diagram.
- Hugo installed locally to preview the workshop site before pushing.

### 5. Timeline & Milestones

**Project Timeline**

| Day | Work | Deliverable |
|-----|------|-------------|
| 1 | Finalise instance type and region; run AWS Pricing Calculator; create IAM roles, SNS, DynamoDB, Parameter Store | Cost table, baseline infrastructure |
| 2 | Launch Windows EC2; deploy `ScheduleHandler`; configure EventBridge Scheduler; test | Layer 1 operational |
| 3 | Deploy `CostMeter`; configure 15-min rule; simulate threshold breach; verify SNS + stop | Layer 2 operational |
| 4 | Configure Budgets alarm; deploy `BudgetBackstop`; set up CloudWatch Alarms; full end-to-end test | Layer 3 operational, full observability |
| 5 | Record demo video (instance running); clean up all resources; capture Cost Explorer | Video, cleanup screenshots |
| 6 | Draw architecture diagram in draw.io; write workshop sections 5.1–5.8 (Vietnamese) | Draft workshop content |
| 7 | Write Proposal (this document); translate to English; run `hugo server`; push to repo; verify links | Complete site |

### 6. Budget Estimation

Pricing is based on AWS Pricing Calculator estimates for **ap-southeast-1 (Singapore)** using on-demand pricing. The Windows instance is kept stopped outside lab hours by Layer 1, limiting billable hours.

[AWS Pricing Calculator Estimate](https://calculator.aws/pricing/2/savedsettings) *(link to be updated with the exported share URL after completing the estimate)*

A PDF export of the estimate is available at: [`/attachments/budget_estimation.pdf`](/attachments/budget_estimation.pdf)

### Infrastructure Costs

| Service | Assumption | Monthly Cost |
|---------|-----------|-------------|
| EC2 Windows t3.micro (ap-southeast-1) | 2 hours/day × 30 days = 60 hours | ~$3.00 |
| EBS gp3 30 GB (root volume) | 30 GB × $0.096/GB | ~$0.29 |
| DynamoDB | < 25 GB storage, < 25 WCU/RCU (Free Tier) | $0.00 |
| Lambda | < 1M requests/month (Free Tier) | $0.00 |
| SNS | < 1,000 email notifications (Free Tier) | $0.00 |
| EventBridge Scheduler | < 14M invocations (Free Tier) | $0.00 |
| CloudWatch Logs | < 5 GB ingestion (Free Tier) | $0.00 |
| AWS Budgets | First 2 budgets free | $0.00 |
| Systems Manager Parameter Store | Standard parameters, free | $0.00 |
| **Total** | | **~$3.29/month** |
| **12-month projection** | | **~$39.48** |

> **Note:** The $20 project budget covers approximately 6–7 hours of Windows EC2 running time at the rates above (before Free Tier credits). Layer 1 and Layer 2 together enforce a hard cap well below this limit. The internal cost threshold in Layer 2 is set at $14 (~70% of $20) to leave headroom for billing lag in Layer 3. All other services stay within Free Tier limits under normal lab usage.

### 7. Risk Assessment

#### Risk Matrix

| Risk | Impact | Probability | Layer affected |
|------|--------|-------------|---------------|
| Forget to stop Windows instance manually | High | Medium | All layers mitigate |
| AWS Budgets alert delayed 8–12 hours | High | High (by design) | Layer 3 is last resort; Layers 1–2 act first |
| `CostMeter` estimated cost diverges from billing | Medium | Medium | Noted; reconciled with Cost Explorer in report |
| Lambda IAM permission error stops wrong instance | High | Low | Scope role to exact instance ARN |
| Windows instance type blocked on Free Plan | Medium | Low | Test on Day 1; switch to smaller type if needed |
| Credentials or Account ID exposed in screenshots | Medium | Low | Redact before saving/committing |
| Project runs behind schedule (1-week deadline) | Medium | Medium | Layers 1 and 2 have priority; Layer 3 is optional |

#### Mitigation Strategies

- **Forgotten instance:** EventBridge Scheduler stops the instance every day at the lab end time regardless of cost data.
- **Budgets delay:** Layer 2 (`CostMeter`) measures cost from first principles (running hours × price/hour) and acts within 15 minutes, independent of billing pipeline.
- **Cost estimate drift:** Internal threshold set 30% below total budget ($14 vs $20). Post-project reconciliation against Cost Explorer is a required section of the report.
- **Wrong-instance stop:** Each Lambda role's `ec2:StopInstances` resource ARN is restricted to the single target instance ID. Tested against a stopped instance before enabling production schedule.
- **Instance type blocked:** Day 1 task is to attempt to launch the chosen instance type; if Console shows a restriction, switch to `t2.micro` or `t2.small` before proceeding.

#### Contingency Plans

- If all Lambda layers fail: stop the instance manually from the AWS Console and disable the EventBridge rules.
- If credit runs out mid-project: use CloudFormation stack deletion or `aws ec2 terminate-instances` to remove the Windows instance immediately, then continue the workshop write-up using screenshots already captured.
- If the $20 budget is insufficient: reduce lab hours or switch to a smaller instance type; update the Pricing Calculator estimate accordingly.

### 8. Expected Outcomes

#### Technical Improvements

- The Windows EC2 instance stops automatically at the end of every lab session — no manual action required.
- Total project spend stays under $20, verified by Cost Explorer after cleanup.
- `CostMeter` provides a running cost estimate updated every 15 minutes, compared against a configurable threshold stored in Parameter Store.
- CloudWatch Alarms and SNS notifications provide immediate visibility into Lambda errors.
- All infrastructure can be torn down with a single CloudFormation stack delete, leaving no orphaned resources.

#### Long-term Value

The system is parameterised (instance ID, price, threshold all in Parameter Store) and can be reused for any Windows lab account without code changes. It scales to multiple instances by adding records to the schedule and adjusting the DynamoDB schema. The three-layer pattern — schedule, near-real-time meter, billing backstop — is applicable to any AWS account where cost control matters and billing data latency is a constraint.

> **Original contribution:** The core innovation in this project is the near-real-time cost estimation mechanism in Layer 2 (`CostMeter`). Rather than relying on AWS billing data, it derives a cost estimate from first principles using instance running time and a manually configured price. The three-layer protection design and the integration of these layers via EventBridge, Lambda, DynamoDB, and Parameter Store were designed independently for this project and are not derived from any AWS workshop template.
