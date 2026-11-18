---
title: "Workshop Overview"
date: 2026-11-18
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Workshop Overview

**"Cost Guard: Protecting Windows Lab Credits on AWS with Three Serverless Layers"**

### Problem Statement

A Windows EC2 instance is billed by the hour. A t3.medium left running 24/7 costs about $46.22 per month, more than the $20 credit budget of this project. AWS Budgets can warn about cost, but its billing data refreshes only a few times per day, so it cannot stop a forgotten machine in time.

This workshop builds Cost Guard, which stops the machine automatically using only AWS-native serverless services.

### What You Will Build

By the end of this workshop you will have:

- A **Windows EC2 instance** (t3.medium) with a 30 GB gp3 volume as the protected lab machine
- **Layer 1 – Schedule Guard:** EventBridge Scheduler and a Lambda function that stop the instance every day at 22:00
- **Layer 2 – Cost Meter:** a Lambda function, run every 5 minutes by EventBridge, that estimates cost and stops the instance when the threshold is reached
- **Layer 3 – Budget Backstop:** an AWS Budget that notifies SNS at 80% and 100% of $20 and triggers a Lambda function to stop the instance
- **Shared foundation:** Parameter Store for configuration, a DynamoDB table for session records, two SNS topics (alerts and errors) and a least-privilege IAM role for each function
- **Observability:** CloudWatch Logs and CloudWatch Alarms on the errors of all three functions

### Architecture

![Architecture Diagram](../../images/2-Proposal/architecture.png)

The numbers in the diagram follow the order of the flow:

1. The lab user starts the Windows EC2 instance manually.
2. Layer 1: EventBridge Scheduler invokes ScheduleHandler when lab hours end.
3. Layer 2: an EventBridge Scheduler schedule invokes CostMeter every 5 minutes.
4. Layer 3: AWS Budgets alerts go through SNS to BudgetBackstop.
5. The Lambda functions read the price and threshold and read or write session records.
6. A Lambda function stops EC2 (schedule ended, cost over threshold, or budget reached).
7. SNS emails the alert to the user.
8. CloudWatch Alarms watch the Lambda errors and send them through SNS to the user.

### Business Logic

- **Starting:** The user starts the instance manually. No layer starts it automatically.
- **Layer 1:** Every day at 22:00 (Asia/Ho_Chi_Minh), stop the instance if it is running. If it is already stopped, do nothing.
- **Layer 2:** Every 5 minutes, if the instance is running:
  - estimate the current session cost as running hours × price per hour
  - add the estimated cost of earlier sessions in the same calendar month
  - if the total reaches the threshold (about $14, 70% of $20), stop the instance, record the event and send an alert
  - otherwise, only record the estimate
- **Restarting after a Layer 2 stop:** The monthly total is still above the threshold, so the next run stops the instance again. The machine stays locked until the next calendar month.
- **Layer 3:** AWS Budgets tracks the actual account spend and notifies at 80% and 100% of $20. Each notification triggers BudgetBackstop, which stops the instance and sends a confirmation email. Billing data is delayed, so this layer is only the last safety net.
- **No conflict between layers:** Any layer can stop the machine, none can start it, and stopping a stopped machine does nothing.
- **Failure of one layer:** The other two layers stay active. Function errors raise a CloudWatch alarm and an email.
- **Cost scope:** The Layer 2 threshold covers EC2 running cost only. The EBS volume ($2.40 per month) is counted separately.
- **Configuration:** Price, threshold and budget limit are Parameter Store values. A change applies on the next run without redeploying.

### Key Design Decisions

- **Stop only, never start automatically:** Starting a machine nobody uses wastes credit.
- **Three layers instead of one:** A time schedule is reliable, a cost meter is fast, and Budgets uses real billing. Together a single failure does not leave the machine unprotected.
- **Cost meter based on running time:** It reacts within minutes and does not wait for billing data.
- **Configuration in Parameter Store:** The system can be reused for another instance type or account without changing code.
- **One IAM role per function:** Each role can stop only the target instance, so a bug cannot stop other machines.
- **Lock until next month after a threshold stop:** It is simple and safe for the credit. The cost of the lock is that the user cannot keep working in that month.

### Services and Pillars (Well-Architected)

- **Cost Optimization:** EventBridge Scheduler, Cost Meter and AWS Budgets stop spending that nobody needs.
- **Reliability:** Three independent layers; if one fails, the others still protect the machine.
- **Security:** One least-privilege IAM role per function, with stop permission limited to the target instance.
- **Operational Excellence:** CloudWatch Logs, CloudWatch Alarms, SNS notifications and the DynamoDB session record make behavior visible.
