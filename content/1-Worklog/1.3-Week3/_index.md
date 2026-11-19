---
title: "Week 3 Worklog"
date: 2026-09-21
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---


### Week 3 Objectives:

* Understand how to scale applications automatically with EC2 Auto Scaling.
* Monitor resources with Amazon CloudWatch (metrics, alarms, logs, dashboards).
* Build foundational container knowledge with Docker.
* Deploy containers on Amazon ECS and compare the EC2 launch type with AWS Fargate.
* Read the official documentation on using CloudWatch alarms to detect failed ECS deployments and roll back.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Learn EC2 Auto Scaling <br>&emsp; + Launch templates, Auto Scaling groups <br>&emsp; + Desired/min/max capacity, scaling policies <br>&emsp; + Integration with a Load Balancer <br> - **Practice:** create a launch template and an Auto Scaling group behind a Load Balancer following the workshop, observe instances being added/removed automatically | 09/21/2026 | 09/21/2026 | <https://000006.awsfcaj.com> |
| 3 | - Learn Amazon CloudWatch <br>&emsp; + Metrics, Alarms and the OK/ALARM/INSUFFICIENT_DATA states <br>&emsp; + CloudWatch Logs <br>&emsp; + Dashboards <br> - **Practice:** view EC2 metrics, create a CPU-threshold alarm, build a simple dashboard | 09/22/2026 | 09/22/2026 | <https://000008.awsfcaj.com> |
| 4 | - Learn Docker <br>&emsp; + Images, containers, Dockerfile <br>&emsp; + Build, run, push images to a registry <br> - Read the container introduction in the learning path (Module 3, Module 5) <br> - **Practice:** write a Dockerfile, build an image and run a container for the sample application following the workshop | 09/23/2026 | 09/23/2026 | <https://000015.awsfcaj.com> <br> <https://cloudjourney.awsfcaj.com/3-optimize/> <br> <https://cloudjourney.awsfcaj.com/5-container/> |
| 5 | - Learn Amazon ECS <br>&emsp; + Clusters, task definitions, tasks, services <br>&emsp; + EC2 launch type <br>&emsp; + Integrating an ECS service with a Load Balancer <br> - **Practice:** create an ECS cluster, a task definition and a service to run the built image | 09/24/2026 | 09/24/2026 | <https://000016.awsfcaj.com> |
| 6 | - Learn ECS with AWS Fargate <br>&emsp; + Serverless compute for containers, comparison with the EC2 launch type <br> - Read AWS docs: detecting failed ECS deployments with CloudWatch alarms <br>&emsp; + Attach existing alarms to deploymentConfiguration (alarms={alarmNames=[...],enable=true,rollback=true}) <br>&emsp; + ECS calls DescribeAlarms during deployment; if an alarm enters ALARM before steady state + bake time, the deployment is marked FAILED <br>&emsp; + With rollback enabled, ECS creates a new deployment that restores the last completed deployment <br>&emsp; + Only for rolling updates (deployment controller type ECS); track rolloutState: IN_PROGRESS → COMPLETED / FAILED <br>&emsp; + Used together with the circuit breaker: whichever triggers first fails the deployment <br>&emsp; + Caveats: DescribeAlarms throttling, alarms already in ALARM at start are ignored, default bake time is under 5 minutes <br>&emsp; + Suggested metrics: ALB HTTPCode_ELB_5XX_Count/4XX_Count, CPUUtilization, MemoryUtilization, SQS ApproximateNumberOfMessagesNotVisible <br> - **Practice:** deploy a service on Fargate following the workshop; review week 3 and **clean up resources** (ECS service/cluster, Load Balancer, Auto Scaling group, alarms) | 09/25/2026 | 09/25/2026 | <https://000067.awsfcaj.com> <br> <https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-alarm-failure.html> |


### Week 3 Achievements:

* Understood Auto Scaling groups, launch templates, scaling policies and how they work with a Load Balancer.
* Able to create CloudWatch alarms, read metrics and build a basic dashboard.
* Wrote a Dockerfile, built and ran a container for the sample application.
* Understood ECS concepts (cluster, task definition, task, service) and the difference between the EC2 and Fargate launch types.
* Understood how ECS uses CloudWatch alarms to mark a deployment as FAILED and roll back automatically, the caveats involved, and how it relates to the deployment circuit breaker.
* Cost incurred this week (from Billing): [TO FILL: USD]
* Evidence (screenshots): [TO FILL: image paths]
