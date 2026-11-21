---
title: "Week 9 Worklog"
date: 2026-11-02
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---


### Week 9 Objectives:

* Understand how to automate a Windows Server upgrade with AWS Systems Manager Automation.
* Deploy WordPress on AWS with a public EC2 tier, a private database and a Multi-AZ setup.
* Try Auto Scaling, Load Balancer, backup and restore, and CloudFront on the same application.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Read the blog about automating Windows Server upgrades with AWS Systems Manager | 11/02/2026 | 11/06/2026 | <https://aws.amazon.com/vi/blogs/modernizing-with-aws/how-to-automate-your-microsoft-windows-server-upgrades-using-aws-systems-manager-update/> |
| 3 | - Read the overview of the WordPress on AWS lab <br>- **Practice:** <br>&emsp; + Create a VPC and subnets <br>&emsp; + Create security groups for EC2 and for the database <br>&emsp; + Create an EC2 instance and an RDS database | 11/03/2026 | 11/03/2026 | <https://000021.awsfcaj.com/> |
| 4 | - **Practice:** <br>&emsp; + Install WordPress on the EC2 instance and connect it to the database | 11/04/2026 | 11/04/2026 | <https://000021.awsfcaj.com/> |
| 5 | - **Practice:** <br>&emsp; + Create an AMI and a launch template <br>&emsp; + Create a target group and a load balancer <br>&emsp; + Create an Auto Scaling group | 11/05/2026 | 11/05/2026 | <https://000021.awsfcaj.com/> |
| 6 | - **Practice:** <br>&emsp; + Create a database snapshot and restore from it <br>&emsp; + Create a CloudFront distribution <br>&emsp; + Clean up resources after the lab | 11/06/2026 | 11/06/2026 | <https://000021.awsfcaj.com/> |


### Week 9 Achievements:

* Learned from the blog how to automate an in-place Windows Server upgrade with AWS Systems Manager Automation:

  * Windows Server 2012 and 2012 R2 reached end of support on October 10, 2023, and extended support for Windows Server 2016 ends on January 12, 2027, so servers that are no longer supported stop getting security updates
  * The runbook AWSEC2-CloneInstanceAndUpgradeWindows clones the instance and upgrades the clone, and the result is a new AMI to launch a new instance from
  * Prerequisites: a working SSM Agent, TLS 1.2, at least 20 GB free on the root volume, an IAM instance profile for SSM and outbound access to the SSM endpoints
  * Limitations: domain controllers, clusters, desktop operating systems and servers with the Remote Desktop Session Host role are not supported
  * After the upgrade, test the application before putting it back into production

* Learned the parts of the WordPress on AWS architecture: CloudFront for content delivery, an Auto Scaling group and a load balancer for scaling and spreading traffic, and Amazon RDS Multi-AZ to keep the database available.

* Deployed WordPress on AWS:

  * Created the network, the security groups and the EC2 instance in a public subnet, with the database in a private subnet
  * Installed WordPress on EC2 and connected it to the RDS database
  * Created an AMI, a launch template, a target group, a load balancer and an Auto Scaling group so the application can scale and balance load
  * Created a database snapshot and restored from it
  * Put CloudFront in front of the application
