---
title: "Week 5 Worklog"
date: 2026-10-05
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---


### Week 5 Objectives:

* Come up with ideas for the proposal and the workshop together with the team.
* Learn the basics of Amazon RDS and deploy a database.
* Find out how to reach Windows instances with Fleet Manager and how to observe AI agents with CloudWatch Omni.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Come up with ideas for the proposal and the workshop together with the team | 10/05/2026 | 10/05/2026 | |
| 3 | - Read the documentation about Amazon RDS <br>&emsp; + Supported database engines <br>&emsp; + Multi-AZ, Read Replica <br>&emsp; + Backups and snapshots <br>&emsp; + Security: encryption, VPC, IAM | 10/06/2026 | 10/06/2026 | <https://000005.awsfcaj.com/> |
| 4 | - **Practice:** <br>&emsp; + Create a VPC, Security Groups and a DB subnet group <br>&emsp; + Create an RDS MySQL database <br>&emsp; + Launch EC2 and connect to RDS with the MySQL client <br>&emsp; + Create a database and a table and insert data <br>&emsp; + Deploy an application that connects to RDS <br>&emsp; + Create a snapshot and restore RDS <br>&emsp; + Clean up resources after the lab | 10/07/2026 | 10/07/2026 | <https://000005.awsfcaj.com/> |
| 5 | - Read the blog about AWS Systems Manager Fleet Manager | 10/08/2026 | 10/08/2026 | <https://aws.amazon.com/vi/blogs/mt/console-based-access-to-windows-instances-using-aws-systems-manager-fleet-manager/> |
| 6 | - Read the blog about Amazon CloudWatch Omni | 10/09/2026 | 10/09/2026 | <https://aws.amazon.com/vi/blogs/aws/introducing-amazon-cloudwatch-omni-ai-powered-observability-for-generative-ai-and-agentic-workloads/> |


### Week 5 Achievements:

* Came up with the first ideas for the proposal and the workshop together with the team.

* Learned Amazon RDS: database engines, Multi-AZ, backups and snapshots, and security.

* Deployed a database on Amazon RDS:

  * Created an RDS MySQL database in a VPC and placed it in private subnets through a DB subnet group
  * Connected from EC2 with the MySQL client to create a database and a table and insert data
  * Deployed an application that connects to RDS
  * Created a snapshot and restored it to a new instance

* Learned AWS Systems Manager Fleet Manager from the blog about reaching Windows instances from the console:

  * Open an RDP session to a Windows instance in the browser through Session Manager
  * No need to open the RDP port, use a public IP or run a bastion host, so the attack surface is smaller
  * Works with an EC2 key pair or Windows credentials and shows up to 4 instances at once
  * Needs the SSM Agent, an instance profile with Systems Manager permissions and access to the SSM endpoint

* Learned Amazon CloudWatch Omni from the blog about observing AI agents:

  * An agent does not behave the same way every time, so traces of every LLM and tool call are needed to find where it went wrong
  * 17 built-in evaluators (coherence, helpfulness, faithfulness and others) score response quality, because latency and error rate do not say whether an answer is good
  * Playground, Experiments and Prompt Management compare prompts and configurations and roll back to older versions
  * Works in VS Code, Kiro or on the web, is built on the open standards OpenInference and ADOT, and supports frameworks such as LangChain, CrewAI and Strands
