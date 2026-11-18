---
title: "Workshop"
date: 2026-11-18
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Cost Guard: Protecting Windows Lab Credits on AWS with Three Serverless Layers

#### Overview

A Windows EC2 instance is billed by the hour, and a forgotten machine can use up a small credit budget in days. This workshop builds **Cost Guard**, a serverless system with three layers that stop the lab instance automatically: a daily schedule, a cost meter that reacts within minutes, and a budget alert that follows the real bill.

You deploy the whole system with one CloudFormation template, test each layer on purpose, review security and cost, and delete everything at the end.

#### AWS Services Used

- **Amazon EC2 and Amazon EBS:** the Windows lab machine that Cost Guard protects.
- **Amazon EventBridge Scheduler:** the daily stop and the 5-minute run of the cost meter.
- **AWS Lambda:** the logic of the three layers.
- **Amazon DynamoDB:** the record of every action and cost estimate.
- **Amazon SNS:** email alerts and the link between Budgets and Lambda.
- **AWS Budgets:** the monthly budget and its 80% and 100% alerts.
- **AWS Systems Manager Parameter Store:** price, threshold and budget limit.
- **Amazon CloudWatch:** logs and alarms on function errors.
- **AWS IAM:** one least-privilege role for each function.
- **AWS CloudFormation:** the whole system in one stack.
- **AWS Cost Explorer:** the comparison of real cost and estimate after cleanup.

#### Architecture

![Cost Guard architecture](../images/2-Proposal/architecture.png)

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisite/)
3. [Deploy the Foundation](5.3-Foundation/)
4. [Layer 1 – Schedule Guard](5.4-Layer1-Schedule/)
5. [Layer 2 – Cost Meter](5.5-Layer2-CostMeter/)
6. [Layer 3 – Budget Backstop and Monitoring](5.6-Layer3-Backstop/)
7. [Security and Cost Review](5.7-Security/)
8. [Clean Up](5.8-Cleanup/)
