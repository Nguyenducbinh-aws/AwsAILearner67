---
title: "Security – Config & Inspector"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Step 5 – Security (AWS Config & Amazon Inspector)

**Goal:** Enforce approved AMIs and required tags with AWS Config, detect CVE vulnerabilities with Amazon Inspector, and confirm secure instance access via Session Manager and IMDSv2.

---

### 7.1 Hardened Image with DISA STIG

Create a second recipe for a STIG-hardened AMI:

**EC2 Image Builder → Image recipes → Create image recipe**

| Field | Value |
|---|---|
| Name | `Secure Image` |
| Version | `1.0.0` |
| Base image | Windows Server 2019 English Full Base (or 2022) |
| Build component | Amazon managed → `stig-build-windows` |

Create a pipeline named `Windows-STIG-Pipeline`, run it, then copy the resulting AMI ID from the pipeline's latest version (e.g., `1.0.0/1`).

Launch a new instance from this STIG AMI, add tags:
- `Workshop=SecureWindows`
- `Image=STIGImage`

Attach the SSM IAM profile. Do not open RDP.

{{% notice warning %}}
Using a STIG component does not guarantee passing a formal audit. State this clearly in your report.
{{% /notice %}}

![STIG pipeline result](/images/5-Workshop/5.7-security/stig-pipeline.png)

---

### 7.2 AWS Config – Restrict to Approved AMIs

**1.** Go to **AWS Config → 1-click setup → Confirm**

**2.** Go to **Rules → Add rule → AWS managed rule → `approved-amis-by-id`**  
Name: `infosec-approved-amis`  
Trigger: keep default (EC2 Instances)  
Parameter: enter the STIG AMI ID from the previous step.

**3.** Set up notification via EventBridge:
- Create SNS topic: `SecureWindowsNotifications` + email subscription
- Create an EventBridge rule that catches **Config rule compliance change** events for `infosec-approved-amis`
- Target: the SNS topic

(EventBridge provides richer notification with Instance ID details, compared to Config's built-in notification.)

Launch a **non-approved** instance to trigger a NON_COMPLIANT finding and confirm you receive the email.

![Config NON_COMPLIANT finding](/images/5-Workshop/5.7-security/config-noncompliant.png)

---

### 7.3 AWS Config – Enforce Required Tags

Add another managed rule: **`required-tags`**

Configure it to check for tag `Workshop = SecureWindows`.  
Reuse the `SecureWindowsNotifications` SNS topic.

---

### 7.4 Amazon Inspector

**1.** Go to **Amazon Inspector → Get started → Activate Amazon Inspector**  
(Single account; no need to delegate admin for this lab.)

{{% notice tip %}}
Inspector scans EC2 instances by using the SSM Agent. Make sure the agent is running on your instances.
{{% /notice %}}

**2.** View **Findings** to see CVE vulnerabilities detected on your instances.

**3.** Use **Patch Manager** (from Step 3.5) to apply OS-level patches.  
Note: Patch Manager updates OS patches only — it does not upgrade the major OS version.

![Inspector findings](/images/5-Workshop/5.7-security/inspector-findings.png)

---

### 7.5 Secure Instance Access

**Session Manager – verify IMDSv2:**

Connect to an instance via Session Manager and run:

```powershell
[string]$token = Invoke-RestMethod `
  -Headers @{"X-aws-ec2-metadata-token-ttl-seconds"="21600"} `
  -Method PUT `
  -Uri http://169.254.169.254/latest/api/token

Invoke-RestMethod `
  -Headers @{"X-aws-ec2-metadata-token"=$token} `
  -Method GET `
  -Uri http://169.254.169.254/latest/meta-data/instance-id
```

This confirms IMDSv2 is working (token-based, more secure than IMDSv1).

**Fleet Manager RDP:** Connect via **EC2 → Connect → RDP client → Connect using Fleet Manager** — no port 3389 open.

![Session Manager terminal](/images/5-Workshop/5.7-security/session-manager.png)

![Fleet Manager RDP](/images/5-Workshop/5.7-security/fleet-manager-rdp.png)

---

### Summary – Security Measures Applied

| Measure | Implementation |
|---|---|
| No open inbound ports | Security group has zero inbound rules |
| No RDP exposed | Access only via Session Manager / Fleet Manager |
| IMDSv2 enforced | Token-based metadata requests |
| Approved AMI policy | AWS Config rule `approved-amis-by-id` |
| Required tags | AWS Config rule `required-tags` |
| OS vulnerabilities | Amazon Inspector (CVE scanning) + Patch Manager |
| Hardened image | STIG build component in Image Builder |
| Least privilege | Dedicated roles with only required policies |
