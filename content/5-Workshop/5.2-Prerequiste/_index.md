---
title: "Prerequisites"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Prerequisites

### AWS Account Setup

1. Use a **dedicated AWS account** for this lab. Do not use a shared or production account.
2. **Do not use the root user.** Create an IAM user or IAM Identity Center user with administrator permissions for the lab, and enable MFA on both root and the lab user.
3. **Set a Budget before creating anything.**

```
Billing → Budgets → Create budget → Cost budget (Monthly)
Amount: $20 USD
Alerts: 50% and 80%
```

{{% notice warning %}}
The full workshop infrastructure (NAT Gateway, Managed AD, multiple instances) costs approximately **$3–5 USD/hour**. Do not leave it running overnight.
{{% /notice %}}

### Local Tools

Install the following on your machine:

| Tool | Purpose | Install |
|---|---|---|
| AWS CLI v2 | Run AWS commands | [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) |
| Git | Version control | [git-scm.com](https://git-scm.com) |
| VS Code | Edit files, write components | [code.visualstudio.com](https://code.visualstudio.com) |
| Hugo Extended | Local preview of report | [gohugo.io](https://gohugo.io/installation/) |
| Session Manager plugin | SSM terminal from CLI | [docs.aws.amazon.com/systems-manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) |

Verify AWS CLI is working:

```bash
aws --version
aws configure          # use lab IAM user key — never commit to GitHub
aws sts get-caller-identity
```

{{% notice warning %}}
Never hard-code Access Keys or passwords in code or in a GitHub repository. Add `.env` and credentials files to `.gitignore`.
{{% /notice %}}

### Region

Pick one region and do not change it throughout the workshop. Recommended: **us-east-1** or **ap-southeast-1**.

```bash
aws configure set region ap-southeast-1
```

### Screenshot

![Budget setup](/images/5-Workshop/5.2-prerequisite/budget.png)

![IAM user with MFA](/images/5-Workshop/5.2-prerequisite/iam-mfa.png)
