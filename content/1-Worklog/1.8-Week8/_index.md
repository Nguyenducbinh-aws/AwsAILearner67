---
title: "Week 8 Worklog"
date: 2026-10-26
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---


### Week 8 Objectives:

* Learn the AWS CLI and use it to manage AWS services from the command line.
* Create resources with the CLI: S3, SNS, IAM, networking and EC2.
* Continue the workshop and check the architecture and the information in the proposal.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Read the overview of the AWS CLI: profiles, configuration, output formats <br>- **Practice:** <br>&emsp; + Install the AWS CLI and configure a profile with aws configure <br>&emsp; + Check resources through the CLI | 10/26/2026 | 10/26/2026 | <https://000011.awsfcaj.com/> |
| 3 | - **Practice:** <br>&emsp; + Use the CLI with Amazon S3 <br>&emsp; + Use the CLI with Amazon SNS | 10/27/2026 | 10/27/2026 | <https://000011.awsfcaj.com/> |
| 4 | - **Practice:** <br>&emsp; + Create IAM groups, users and access keys with the CLI <br>&emsp; + Create a VPC, subnets, an internet gateway and a route table with the CLI | 10/28/2026 | 10/28/2026 | <https://000011.awsfcaj.com/> |
| 5 | - **Practice:** <br>&emsp; + Create a key pair and a security group, then launch an EC2 instance and connect over SSH with the CLI <br>&emsp; + Terminate the instance and clean up resources after the lab | 10/29/2026 | 10/29/2026 | <https://000011.awsfcaj.com/> |
| 6 | - Continue working on the workshop <br>- Check the architecture and the information in the proposal | 10/30/2026 | 10/30/2026 | |


### Week 8 Achievements:

* Learned what the AWS CLI is: an open-source tool with direct access to the public APIs of AWS services, so anything you can do in the console can also be done from a command line or a script.

* Set up and used the CLI:

  * Configured a profile with aws configure (access key, secret access key, Region and output format) and used named profiles with the --profile option
  * Picked between the json, yaml, text and table output formats depending on whether a person or a tool reads the result
  * Checked EC2 instances and S3 buckets from the CLI and tried the CLI auto-prompt for help with syntax

* Created and managed resources with the CLI:

  * Worked with Amazon S3 and Amazon SNS from the command line
  * Created an IAM group, a user and an access key
  * Created a VPC, subnets, an internet gateway and a route table
  * Created a key pair and a security group, launched an EC2 instance in the new network, connected over SSH and terminated it

* Learned to protect credentials: never share an access key, prefer IAM roles where possible, and delete key pairs and access keys that are no longer used.

* Continued building the workshop and checked the architecture and the information in the proposal.
