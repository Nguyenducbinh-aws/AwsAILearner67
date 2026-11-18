---
title: "Deploy the Foundation"
date: 2026-11-18
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Deploy the Foundation

In this step you deploy one CloudFormation stack that creates the lab machine and everything the three layers share: configuration, data, notifications and IAM roles. The layers themselves are explained and tested in 5.4, 5.5 and 5.6.

### What the stack creates

- **Lab machine:** a Windows Server 2022 `t3.medium` instance with a 30 GB gp3 root volume, tagged `Project=CostGuard`. If you pass `ExistingInstanceId`, the stack protects that instance instead.
- **Parameter Store:** `/costguard/instance-id`, `/costguard/price-per-hour`, `/costguard/cost-threshold` and `/costguard/budget-limit`.
- **DynamoDB:** table `CostGuardSessions` in on-demand mode, with `instance_id` as partition key and `timestamp` as sort key.
- **SNS:** topics `costguard-alerts`, `costguard-errors` and `costguard-budgets`. Your email is subscribed to the first two.
- **IAM:** one role for each Lambda function, plus one role for EventBridge Scheduler.
- **Lambda, schedules, Budget, alarms:** created here and described in 5.4–5.6.

### Parameters

- `AlertEmail` (required): the mailbox that receives alerts.
- `ExistingInstanceId`: leave empty to create a new lab instance.
- `PricePerHour`: 0.06 by default. This is the hourly price of the instance in the Pricing Calculator.
- `CostThreshold`: 14 by default, about 70% of the budget.
- `BudgetLimit`: 20 by default.
- `StopSchedule` and `StopScheduleTimezone`: `cron(0 22 * * ? *)` and `Asia/Ho_Chi_Minh` by default.
- `CostMeterRateMinutes`: 5 by default.

### Step 1 – Create the stack

**Console**

1. Open **CloudFormation** in `us-east-1` and choose **Create stack → With new resources**.
2. Choose **Upload a template file** and select `cost-guard.yaml`.
3. Name the stack `CostGuard`.
4. Enter your email in `AlertEmail`. Keep the other parameters at their defaults.
5. On the last page, tick **I acknowledge that AWS CloudFormation might create IAM resources**, then choose **Submit**.

**AWS CLI**

```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name CostGuard \
  --template-file cost-guard.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides AlertEmail=you@example.com
```

### Step 2 – Confirm the email subscriptions

1. Open the mailbox. You receive two messages from Amazon SNS, one for `costguard-alerts` and one for `costguard-errors`.
2. Choose **Confirm subscription** in each one. Until you confirm, SNS sends nothing to that address.

### Step 3 – Check the result

1. When the stack status is **CREATE_COMPLETE**, open the **Outputs** tab. `ProtectedInstanceId` is the instance that Cost Guard protects.
2. Check each resource in its own console:
   - **EC2:** the instance `CostGuard-Lab` is running.
   - **Systems Manager → Parameter Store:** the four `/costguard/` parameters exist.
   - **DynamoDB:** the table `CostGuardSessions` is active.
   - **SNS:** the three topics exist, and the subscriptions of the first two show **Confirmed**.
   - **Lambda:** `CostGuard-ScheduleHandler`, `CostGuard-CostMeter` and `CostGuard-BudgetBackstop` exist.
   - **EventBridge → Scheduler:** `costguard-daily-stop` and `costguard-cost-meter` are enabled.

**Expected result:** all resources are present and the instance is running. The instance has now started to cost money, so continue to 5.4 without long pauses.

### Things to check if something goes wrong

- **`ROLLBACK_COMPLETE`:** open the **Events** tab, find the first `CREATE_FAILED` row and read its status reason. A stack in this state cannot be updated, so delete it and create it again.
- **Capability error:** the stack needs `CAPABILITY_IAM`. On the console, tick the acknowledgement box. With the CLI, keep `--capabilities CAPABILITY_IAM`.
- **Instance creation fails:** a missing default VPC is the most likely cause (see 5.2). Launch an instance yourself and pass its ID in `ExistingInstanceId`.
- **No email arrives:** look in the spam folder and check that the address in `AlertEmail` has no typo.
