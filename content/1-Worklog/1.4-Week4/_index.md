---
title: "Week 4 Worklog"
date: 2026-09-28
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---


### Week 4 Objectives:

* Get familiar with AWS CloudShell and the AWS CLI.
* Understand the basics of Amazon S3 and Amazon CloudFront.
* Practice storing data, hosting a static website and managing objects on S3.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Learn about AWS CloudShell <br> - **Practice:** <br>&emsp; + Create a CloudShell environment <br>&emsp; + Use basic commands: pwd, ls -la, mkdir, touch, vi, cat <br>&emsp; + Use the AWS CLI: check the version, list S3 buckets and EC2 instances <br>&emsp; + Upload a file to and download a file from CloudShell <br>&emsp; + Delete the CloudShell environment | 09/28/2026 | 09/28/2026 | <https://000049.awsfcaj.com/> |
| 3 | - Learn about Amazon S3 <br>&emsp; + Buckets and objects <br>&emsp; + Storage classes <br>&emsp; + Versioning, replication <br> - **Practice:** <br>&emsp; + Create an S3 bucket <br>&emsp; + Upload the website source code to the bucket | 09/29/2026 | 10/02/2026 | <https://000057.awsfcaj.com/> |
| 4 | - **Practice:** <br>&emsp; + Enable static website hosting and set index.html <br>&emsp; + Turn off Block public access and make objects public with ACLs <br>&emsp; + Open the website through the website endpoint and check the page load time | 09/30/2026 | 09/30/2026 | <https://000057.awsfcaj.com/> |
| 5 | - Learn about Amazon CloudFront <br> - **Practice:** <br>&emsp; + Turn Block all public access back on for the bucket <br>&emsp; + Create a CloudFront distribution with an S3 origin and add the bucket policy <br>&emsp; + Open the website through CloudFront and compare the page load time | 10/01/2026 | 10/01/2026 | <https://000057.awsfcaj.com/> |
| 6 | - Learn about object management in S3 <br>&emsp; + Versioning <br>&emsp; + Move objects <br>&emsp; + Cross-Region replication <br> - **Practice:** <br>&emsp; + Enable versioning, edit index.html and upload it again to see the versions <br>&emsp; + Create a new bucket and move objects to it <br>&emsp; + Create a replication rule to a bucket in N. Virginia <br>&emsp; + Clean up resources after the lab | 10/02/2026 | 10/02/2026 | <https://000057.awsfcaj.com/> |


### Week 4 Achievements:

* Used AWS CloudShell to run commands right in the browser without installing anything:

  * Used basic Linux commands to create directories, create and edit files, and view file permissions
  * Used the AWS CLI to check the version and list the S3 buckets and EC2 instances in the account
  * Uploaded and downloaded files between the local machine and CloudShell

* Understood Amazon S3 (buckets, objects, storage classes) and created a bucket to store the website source code.

* Hosted a static website on S3:

  * Enabled static website hosting and opened the website through the website endpoint (HTTP)
  * Turned off Block public access and made objects public with ACLs so the website can be reached

* Used CloudFront to serve the website without making the bucket public:

  * Turned Block all public access back on and let CloudFront read the bucket through a bucket policy
  * Saw the page load time drop and learned which PoP serves the content

* Managed objects in S3:

  * Used versioning to keep multiple versions of an object and restore older content
  * Moved objects to another bucket
  * Used replication to copy objects automatically to a bucket in another Region for backup
