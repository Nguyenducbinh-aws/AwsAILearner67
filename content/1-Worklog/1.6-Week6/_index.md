---
title: "Week 6 Worklog"
date: 2026-10-12
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---


### Week 6 Objectives:

* Learn Amazon Lightsail and deploy two applications on it.
* Learn how to secure, back up and resize an application on Lightsail.
* Learn Amazon EC2 Auto Scaling together with Elastic Load Balancing and compare the scaling solutions.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Read the overview of Amazon Lightsail <br>- **Practice:** <br>&emsp; + Create a Lightsail managed MySQL database <br>&emsp; + Deploy a WordPress blog from an Ubuntu blueprint, attach a static IP and configure WordPress <br>&emsp; + Deploy a PrestaShop store from an application blueprint, attach a static IP and set the domain | 10/12/2026 | 10/12/2026 | <https://000045.awsfcaj.com/> |
| 3 | - **Practice:** <br>&emsp; + Restrict the SSH rule of the firewall <br>&emsp; + Create snapshots <br>&emsp; + Move WordPress to a larger plan from a snapshot <br>&emsp; + Create a CPU alarm <br>&emsp; + Clean up resources after the lab | 10/13/2026 | 10/13/2026 | <https://000045.awsfcaj.com/> |
| 4 | - Read the overview of EC2 Auto Scaling Group and Elastic Load Balancing <br>- **Practice:** <br>&emsp; + Set up the network <br>&emsp; + Launch an EC2 instance and a database instance, add data to the database <br>&emsp; + Deploy the web server | 10/14/2026 | 10/14/2026 | <https://000006.awsfcaj.com/> |
| 5 | - **Practice:** <br>&emsp; + Create a launch template <br>&emsp; + Create a target group and an Application Load Balancer <br>&emsp; + Test the application through the load balancer <br>&emsp; + Create an Auto Scaling group <br>&emsp; + Prepare metrics for predictive scaling | 10/15/2026 | 10/15/2026 | <https://000006.awsfcaj.com/> |
| 6 | - **Practice:** <br>&emsp; + Test manual, scheduled, dynamic and predictive scaling <br>&emsp; + Clean up resources after the lab | 10/16/2026 | 10/16/2026 | <https://000006.awsfcaj.com/> |


### Week 6 Achievements:

* Learned Amazon Lightsail: a plan bundles a virtual server, SSD storage, data transfer and a static IP for a fixed monthly price, so there is no need to design a VPC, security groups or EBS volumes first.

* Deployed two applications on Lightsail in the Singapore Region:

  * A WordPress blog from an Ubuntu 24.04 blueprint with a Lightsail managed MySQL database
  * A PrestaShop store from a ready-made application blueprint
  * A static IP on each instance so the address does not change

* Secured and operated the applications:

  * Restricted the SSH rule so that only the Lightsail browser SSH client can connect, instead of any address on the internet
  * Created snapshots and used one to move WordPress to a larger plan, since a Lightsail instance cannot change its plan in place
  * Moved the static IP to the new instance, and the blog kept all its content because the data is in the managed database
  * Created a CPU alarm

* Learned what an EC2 Auto Scaling group gives you: failing instances are replaced automatically, capacity follows demand, Elastic Load Balancing spreads the load, and cost drops when traffic is low.

* Deployed the FCAJ Management application on EC2 with a database and a web server, then built the scaling setup around it:

  * A launch template that holds the AMI, instance type, security groups and user data
  * A target group and an Application Load Balancer in front of the instances
  * An Auto Scaling group across 3 public subnets with desired capacity 1, minimum 1 and maximum 3, load balancer health checks, CloudWatch group metrics and SNS notifications

* Compared four scaling solutions:

  * Manual scaling: change the desired capacity by hand for a one-off need
  * Scheduled scaling: change capacity on a schedule when traffic is predictable
  * Dynamic scaling: respond to real-time metrics such as CPU utilization or request count
  * Predictive scaling: use historical data to plan capacity for the next 48 to 72 hours, which needs metrics prepared beforehand
