---
title: "Clean Up"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Step 6 – Clean Up Resources

**Goal:** Delete all resources created in this workshop to avoid unexpected AWS charges.

{{% notice warning %}}
Always verify in **Billing → Cost Explorer** 1–2 days after cleanup to confirm no resources are still running.
{{% /notice %}}

---

### Cleanup Order

Follow this order — some resources depend on others being deleted first.

**1. Amazon Inspector**

Disable resource scanning before disabling the service. If you get an error, wait a few minutes and retry:

```bash
aws inspector2 disable --resource-types EC2 ECR LAMBDA
aws inspector2 disable
```

**2. AWS Config**

Go to **AWS Config → Rules** → delete both rules (`infosec-approved-amis`, `required-tags`)  
Then go to **Settings** → turn off the recorder.

**3. EventBridge Rules**

Delete the EventBridge rule created for Config notifications.

**4. Amazon CloudWatch**

```bash
# Delete alarms
aws cloudwatch delete-alarms --alarm-names LogonFailureAlarm

# Delete log groups (not removed with stack — must be done manually)
aws logs delete-log-group --log-group-name /ssm/runcommand
aws logs delete-log-group --log-group-name /windows/httperr
aws logs delete-log-group --log-group-name /windows/eventlog/application
aws logs delete-log-group --log-group-name /windows/eventlog/system
```

Delete the dashboard: **CloudWatch → Dashboards → `operate-windows-workshop` → Delete**

**5. Amazon SNS**

```bash
# Replace with your actual topic ARNs
aws sns delete-topic --topic-arn <LogonFailureAlarm-topic-arn>
aws sns delete-topic --topic-arn <SecureWindowsNotifications-topic-arn>
```

**6. AWS Systems Manager**

```bash
# Delete parameters
aws ssm delete-parameter --name module2
aws ssm delete-parameter --name AmazonCloudWatch-windows
```

Delete State Manager associations: **Systems Manager → State Manager** → select each association → **Delete**

**7. EC2 Instances**

Terminate all lab instances:  
**EC2 → Instances** → select all lab instances → **Instance state → Terminate**

**8. EC2 Image Builder**

In order:
- Delete **Lifecycle policies**: `my-ib-lifecycle-policy`
- Delete **Pipelines**: `my-first-pipeline`, `Windows-STIG-Pipeline`
- Delete **Recipes**: `my-first-recipe`, `Secure Image`
- Delete **Infrastructure configurations**: `workshop-infra-config`
- **Deregister AMIs**: EC2 → AMIs → select built AMIs → **Actions → Deregister AMI**
- **Delete EBS Snapshots**: EC2 → Snapshots → delete snapshots associated with the deregistered AMIs

**9. IAM Roles**

Delete roles: `lab-windows-ssm-role`, `lab-imagebuilder-role`

**10. Networking**

```bash
# Delete VPC endpoints, NAT Gateway (if used), release Elastic IPs
# Then delete subnets, route tables, internet gateway, and VPC
```

Go to **VPC → Endpoints** → delete all lab endpoints  
Go to **VPC → Your VPCs** → delete the lab VPC (this also removes subnets, route tables, and security groups)

---

### Verify Cleanup

After completing the steps above:

- [ ] No running EC2 instances from the lab
- [ ] No AMIs from Image Builder
- [ ] No EBS snapshots from the lab
- [ ] No NAT Gateway or Elastic IP charges
- [ ] Inspector and Config are disabled
- [ ] Log groups deleted
- [ ] Check **Billing → Cost Explorer** in 1–2 days

![Cost Explorer showing $0 after cleanup](/images/5-Workshop/5.8-cleanup/cost-explorer.png)
