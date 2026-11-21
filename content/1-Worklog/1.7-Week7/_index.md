---
title: "Week 7 Worklog"
date: 2026-10-19
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---


### Week 7 Objectives:

* Learn Amazon CloudWatch: metrics, logs, alarms and dashboards.
* Learn how hybrid DNS works between an on-premises DNS system and Amazon Route 53.
* Practice with CloudFormation templates and clean up resources after each lab.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Read the overview of Amazon CloudWatch <br>- **Practice:** <br>&emsp; + Deploy three EC2 instances with the CloudWatch agent from a CloudFormation template <br>&emsp; + View metrics <br>&emsp; + Use search expressions, math expressions and dynamic labels | 10/19/2026 | 10/19/2026 | <https://000008.awsfcaj.com/> |
| 3 | - **Practice:** <br>&emsp; + Read logs and query them with CloudWatch Logs Insights <br>&emsp; + Create a metric filter that counts error lines <br>&emsp; + Create an alarm that sends an email through Amazon SNS | 10/20/2026 | 10/20/2026 | <https://000008.awsfcaj.com/> |
| 4 | - **Practice:** <br>&emsp; + Create a CloudWatch dashboard <br>&emsp; + Clean up resources after the lab | 10/21/2026 | 10/21/2026 | <https://000008.awsfcaj.com/> |
| 5 | - Read the overview of hybrid DNS and Route 53 Resolver <br>- **Practice:** <br>&emsp; + Create a key pair and launch the network with a CloudFormation template <br>&emsp; + Configure security groups and connect to the Remote Desktop Gateway <br>&emsp; + Deploy AWS Managed Microsoft AD to simulate the on-premises DNS | 10/22/2026 | 10/22/2026 | <https://000010.awsfcaj.com/> |
| 6 | - **Practice:** <br>&emsp; + Create the Route 53 Resolver outbound endpoint and resolver rule <br>&emsp; + Create the Route 53 Resolver inbound endpoint <br>&emsp; + Test name resolution with nslookup and Resolve-DnsName <br>&emsp; + Clean up resources after the lab | 10/23/2026 | 10/23/2026 | <https://000010.awsfcaj.com/> |


### Week 7 Achievements:

* Learned what Amazon CloudWatch does: it collects metrics and logs, lets you query and graph them, and raises alarms when something goes wrong.

* Deployed the CloudWatch lab from a CloudFormation template:

  * Three EC2 instances on Amazon Linux 2023 that run a sample workload and the CloudWatch agent
  * An IAM role with only the permissions the agent needs, instead of AdministratorAccess

* Worked with metrics and logs:

  * Combined metrics with search expressions, math expressions and dynamic labels
  * Queried logs with CloudWatch Logs Insights
  * Created a metric filter that counts the ERROR lines of the application and publishes the count as a metric (a filter only counts events that arrive after it is created)

* Created an alarm and a dashboard:

  * An alarm on the error count that sends an email through Amazon SNS when errors go above a threshold
  * A dashboard that puts the metrics in one place

* Learned how hybrid DNS works with Route 53 Resolver:

  * An outbound endpoint sends queries from AWS to the on-premises DNS
  * An inbound endpoint receives queries from on-premises for domains hosted on AWS
  * Resolver rules decide which domains are forwarded

* Built the hybrid DNS lab:

  * Launched the network (VPC, public and private subnets, Remote Desktop Gateway) from a CloudFormation template
  * Deployed AWS Managed Microsoft AD with the domain onprem.example.com to stand in for the on-premises DNS
  * Created the outbound endpoint, the inbound endpoint and the resolver rule
  * Checked from the Remote Desktop Gateway that the domain name resolves with nslookup and Resolve-DnsName
