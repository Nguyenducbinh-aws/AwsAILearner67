---
title: "Set Up Base Environment"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Step 1 – Set Up Base Environment

**Goal:** Have 2 Windows Server EC2 instances in a private subnet, registered as SSM managed nodes, tagged `workshop=module2`.

---

### 3.1 IAM Role for EC2 Instances (SSM + CloudWatch Agent)

Go to **IAM → Roles → Create role → AWS service → EC2**, attach these two policies:

- `AmazonSSMManagedInstanceCore`
- `CloudWatchAgentServerPolicy`

**Role name:** `lab-windows-ssm-role`

(The console automatically creates an instance profile with the same name.)

![IAM role creation](/images/5-Workshop/5.3-environment/iam-role.png)

---

### 3.2 IAM Role for Image Builder

Create a second role with the same trust (`ec2.amazonaws.com`) and attach:

- `EC2InstanceProfileForImageBuilder`
- `AmazonSSMManagedInstanceCore`

**Role name:** `lab-imagebuilder-role`

---

### 3.3 Network – VPC and Endpoints

**1.** Create a VPC with 1 private subnet. Use **VPC Endpoints** instead of a NAT Gateway (cheaper and keeps traffic off the public Internet).

Create the following interface endpoints, all pointing to the private subnet's security group on port 443:

| Endpoint | Service |
|---|---|
| `com.amazonaws.<region>.ssm` | Systems Manager |
| `com.amazonaws.<region>.ssmmessages` | Session Manager |
| `com.amazonaws.<region>.ec2messages` | EC2 messages |
| `com.amazonaws.<region>.logs` | CloudWatch Logs |
| `com.amazonaws.<region>.monitoring` | CloudWatch Metrics |

**2.** Security group for instances: **no inbound rules**.  
Security group for endpoints: allow HTTPS 443 inbound from the instance security group.

![VPC and endpoints](/images/5-Workshop/5.3-environment/vpc-endpoints.png)

{{% notice tip %}}
If you need internet access temporarily (e.g., to download the CloudWatch Agent), you can use a public subnet for the lab, but still keep no RDP port open. Access only via Session Manager.
{{% /notice %}}

---

### 3.4 Launch 2 Windows EC2 Instances

The command below fetches the latest Windows Server 2022 AMI from Parameter Store automatically:

```bash
aws ec2 run-instances \
  --image-id $(aws ssm get-parameter \
     --name /aws/service/ami-windows-latest/Windows_Server-2022-English-Full-Base \
     --query Parameter.Value --output text) \
  --instance-type t3.medium --count 2 \
  --iam-instance-profile Name=lab-windows-ssm-role \
  --subnet-id <your-subnet-id> \
  --security-group-ids <your-sg-id> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=workshop,Value=module2},{Key=Name,Value=lab-win}]'
```

**Verify:** After a few minutes, go to **Systems Manager → Fleet Manager**. Both instances must appear as **Online** managed nodes.

If they do not appear: check the IAM role is attached, the VPC endpoints exist, and try rebooting the instances (SSM Agent registers on boot).

![Fleet Manager showing managed nodes](/images/5-Workshop/5.3-environment/fleet-manager.png)

---

### 3.5 Set Administrator Password (for Fleet Manager RDP)

Connect via **Session Manager**, then run:

```powershell
$pw = -join ((48..57)+(65..90)+(97..122)+(33,35,36,37,38,42) | `
  Get-Random -Count 20 | ForEach-Object {[char]$_})
net user administrator $pw
Write-Host "Password set successfully"
```

Then connect via: **EC2 → Connect → RDP client → Connect using Fleet Manager**

{{% notice warning %}}
Do not include the password in your report or screenshots.
{{% /notice %}}
