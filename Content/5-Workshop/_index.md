---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Manage, Monitor and Operate Microsoft Workload Securely on AWS

#### Overview

This workshop covers how to build, manage, monitor, and secure Windows Server workloads on AWS using native AWS services — without opening RDP to the Internet.

You will use **EC2 Image Builder** to create a hardened golden AMI, **AWS Systems Manager** to manage the fleet by tag, **Amazon CloudWatch** to collect logs and metrics and trigger alerts, and **AWS Config + Amazon Inspector** to enforce security compliance.

#### AWS Services Used

| Service | Purpose |
|---|---|
| EC2 Image Builder | Build hardened golden AMI with CloudWatch Agent pre-installed |
| AWS Systems Manager | Fleet Manager, Run Command, State Manager, Patch Manager, Parameter Store, Session Manager |
| Amazon CloudWatch | Metrics, Logs, Metric Filters, Alarms, Dashboards |
| Amazon SNS | Email alerts from alarms and Config events |
| AWS Config | Enforce approved AMI and required tags |
| Amazon Inspector | CVE vulnerability scanning |
| Amazon VPC + Endpoints | Private network, no public RDP exposure |

#### Architecture

![Workshop Architecture Diagram](/images/5-Workshop/5.1-overview/architecture.png)

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Set Up Base Environment](5.3-Environment/)
4. [EC2 Image Builder – Golden AMI](5.4-ImageBuilder/)
5. [AWS Systems Manager](5.5-SystemsManager/)
6. [Amazon CloudWatch](5.6-CloudWatch/)
7. [Security – Config & Inspector](5.7-Security/)
8. [Clean Up](5.8-Cleanup/)
