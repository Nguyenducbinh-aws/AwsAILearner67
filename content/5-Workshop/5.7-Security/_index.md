---
title: "Security and Cost Review"
date: 2026-11-18
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Security and Cost Review

This step reviews how Cost Guard handles security, what it costs and how it fits the AWS Well-Architected pillars.

### 1. Security

**One IAM role for each function.** The three Lambda functions have separate roles, so a mistake in one function cannot use the permissions of another. The permission to stop an instance is limited to the **ARN of the protected instance**:

```json
{
  "Effect": "Allow",
  "Action": "ec2:StopInstances",
  "Resource": "arn:aws:ec2:us-east-1:<account-id>:instance/<instance-id>"
}
```

`ec2:DescribeInstances` is allowed on all resources, because this action does not support resource-level restriction. The other permissions are limited to the parameters, the table and the topics that the function uses.

**No credentials in code.** The functions use their execution roles. The Parameter Store values are only the hourly price, the threshold, the budget limit and the instance ID.

**Services call each other with their own roles.** EventBridge Scheduler uses a role that can invoke only the two functions. The topic policy of `costguard-budgets` lets only the Budgets service in your own account publish.

**No open ports.** The template adds no inbound rule, and the lab instance uses the default security group of the default VPC.

**To check:** open **IAM → Roles**, choose each `CostGuard` role and read its policy. The only action that touches EC2 state is `StopInstances` on one instance.

### 2. Cost estimation

The estimate was made with the AWS Pricing Calculator for US East (N. Virginia), using on-demand prices and 120 lab hours per month. Prices are without the Free Tier.

- Amazon EC2 Windows (t3.medium): 120 hours × $0.06 = **$7.20**
- Amazon EBS (gp3, 30 GB): **$2.40**, billed for the whole month even while the instance is stopped
- Public IPv4 address: 120 hours × $0.005 = **$0.60**
- AWS Lambda: about **$0.02**
- Amazon DynamoDB (on-demand): about **$0.002**
- Amazon CloudWatch (3 alarms and logs): about **$0.31**
- EventBridge Scheduler and Amazon SNS: under **$0.01** each

The services in the Calculator (EC2, EBS, Lambda and DynamoDB) add up to about **$9.62 per month**. With the other services added, the total is about **$10.54 per month**, under the $20 budget. The same instance running 24/7 would cost about $46.22 per month, so Cost Guard avoids about $36.60 per month.

### 3. Cost optimization

- Limit the running hours. The scheduled stop and the manual start keep the machine near 120 hours instead of 730.
- Use pay-per-use services in their free allowance: Lambda, DynamoDB on-demand, SNS and EventBridge Scheduler.
- Use standard parameters in Parameter Store, which have no charge.
- Keep the 30 GB gp3 volume and add no extra IOPS or throughput.
- Do not attach an Elastic IP, which is billed while it is allocated.
- Delete all resources after the workshop (see 5.8).

### 4. Well-Architected pillars

- **Operational Excellence:** Lambda logs, CloudWatch alarms and the session records in DynamoDB show what the system did and why.
- **Security:** one role for each function, a stop permission limited to one instance, no credentials in code.
- **Reliability:** three independent layers, and every action is safe to repeat.
- **Performance Efficiency:** a t3.medium for one lab session, and Lambda functions of 128 MB that run for about a second.
- **Cost Optimization:** limited running hours, pay-per-use services and a cleanup at the end.
