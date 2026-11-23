---
title: "Introduction"
date: 2026-11-18
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Introduction to Cost Guard

A Windows EC2 instance is billed by the hour. A t3.small left running 24/7 costs about $31.04 per month, more than the $20 credit budget of this project. AWS Budgets can warn about cost, but its billing data refreshes only a few times per day, so it cannot stop a forgotten machine in time on its own.

**Cost Guard** is a serverless system that stops the lab instance automatically. It uses three independent layers, and each layer covers the weakness of the others. Any layer can stop the machine, no layer starts it, and stopping a machine that is already stopped does nothing.

- **Layer 1 – Schedule Guard:** stops the instance every day at 22:00 (Asia/Ho_Chi_Minh), whether or not anyone remembers.
- **Layer 2 – Cost Meter:** runs every 5 minutes, estimates the monthly cost from the running time and the hourly price, and stops the instance when the estimate reaches the threshold of about $14 (70% of $20).
- **Layer 3 – Budget Backstop:** an AWS Budget follows the real bill, and at 80% and 100% of $20 it triggers a Lambda function that stops the instance.

#### Workshop overview

In this workshop you deploy Cost Guard with one CloudFormation template, test each layer on purpose, review security and cost, and delete everything at the end so that no charge is left behind.

![Architecture Diagram](../../images/2-Proposal/architecture.png)

The numbers in the diagram follow the order of the flow:

1. The lab user starts the Windows EC2 instance manually.
2. Layer 1: EventBridge Scheduler invokes ScheduleHandler when lab hours end.
3. Layer 2: an EventBridge Scheduler schedule invokes CostMeter every 5 minutes.
4. Layer 3: AWS Budgets alerts go through SNS to BudgetBackstop.
5. The Lambda functions read their configuration from Parameter Store and read or write session records in DynamoDB.
6. A Lambda function stops EC2 when the schedule ends, the cost passes the threshold or the budget is reached.
7. SNS emails the alert to the user.
8. CloudWatch Alarms watch the Lambda errors and send them through SNS to the user.

You build the following resources:

- A **Windows EC2 instance** (t3.small) with a 30 GB gp3 volume as the protected lab machine
- Three **Lambda functions** for the three layers, with **EventBridge Scheduler** and **AWS Budgets** as their triggers
- **Parameter Store** for price, threshold and budget limit, and a **DynamoDB** table for session records
- Two **SNS topics** (alerts and errors) and one least-privilege **IAM role** for each function
- **CloudWatch Logs** and **CloudWatch Alarms** on the errors of all three functions

The Layer 2 threshold covers the EC2 running cost only. The EBS volume ($2.40 per month) is counted separately. After a Layer 2 stop, the monthly total stays above the threshold, so the machine remains locked until the next calendar month.
