---
title: "Week 10 Worklog"
date: 2026-11-09
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---


### Week 10 Objectives:

* Learn AWS WAF and use it to protect a web application.
* Learn Amazon GuardDuty and try detecting and handling threats.
* See how an automatic response is built with Amazon EventBridge and AWS Lambda.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Read the introduction to AWS WAF <br>- **Practice:** <br>&emsp; + Create an S3 bucket and deploy a sample web application <br>&emsp; + Create a web ACL with managed rules <br>&emsp; + Create a custom rule and an advanced custom rule <br>&emsp; + Test the new rules and set up logging <br>&emsp; + Clean up resources after the lab | 11/09/2026 | 11/13/2026 | <https://000026.awsfcaj.com/> |
| 3 | - Read the overview of Amazon GuardDuty: data sources and findings <br>- Go through and try the lab: <br>&emsp; + Prepare the environment with a CloudFormation template | 11/10/2026 | 11/10/2026 | <https://000098.awsfcaj.com/> |
| 4 | - Go through and try the lab: <br>&emsp; + Scenario 1: compromised EC2 instance, detected with GuardDuty and handled with EventBridge and Lambda | 11/11/2026 | 11/11/2026 | <https://000098.awsfcaj.com/> |
| 5 | - Go through and try the lab: <br>&emsp; + Scenario 2: compromised IAM credentials, handled manually | 11/12/2026 | 11/12/2026 | <https://000098.awsfcaj.com/> |
| 6 | - Go through and try the lab: <br>&emsp; + Scenario 3: IAM role credential exfiltration, handled with Lambda <br>&emsp; + Review the findings and clean up resources after the lab | 11/13/2026 | 11/13/2026 | <https://000098.awsfcaj.com/> |


### Week 10 Achievements:

* Learned what AWS WAF does: it is a web application firewall that protects web applications and APIs from attacks such as SQL injection and cross-site scripting, and lets you write custom rules that block or allow HTTP requests before they reach the application.

* Practiced AWS WAF:

  * Deployed a sample web application and created a web ACL with managed rules
  * Created a custom rule that blocks requests with a specific header, and an advanced custom rule
  * Sent test requests and saw the blocked ones in the dashboard and in the sampled requests
  * Turned on logging for the web ACL

* Learned how Amazon GuardDuty works:

  * A fully managed, regional service that analyzes VPC Flow Logs, CloudTrail logs and DNS logs without affecting the resources it watches
  * Results appear as findings with a readable type name, and GuardDuty sends a notification when a finding changes
  * Pricing depends on the number of CloudTrail events and the volume of flow logs and DNS logs, and each account has a 30-day trial in each Region

* Went through and tried the three threat scenarios in the lab:

  * A compromised EC2 instance (finding UnauthorizedAccess:EC2/MaliciousIPCaller.Custom), detected by GuardDuty and handled automatically with EventBridge rules and Lambda
  * Compromised IAM credentials (findings Recon:IAMUser/MaliciousIPCaller.Custom and UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom), finding who was calling the API and fixing it manually
  * IAM role credential exfiltration (finding UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration), handled with Lambda

* Saw the whole detect, notify and respond flow in cloud security.
