---
title: "EC2 Image Builder – Golden AMI"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Step 2 – EC2 Image Builder (Golden AMI)

**Goal:** Build a golden AMI based on Windows Server 2025 with the CloudWatch Agent pre-installed, versioned, and managed by a lifecycle policy.

---

### How Image Builder Works

Image Builder runs a pipeline in three phases:

1. **Build** — launches a temporary EC2 instance, installs components via SSM Run Command
2. **Test** — launches a second instance from the new AMI to run test components
3. **Distribute** — makes the AMI available in your account (and optionally other regions)

![Image Builder pipeline flow](/images/5-Workshop/5.4-imagebuilder/pipeline-flow.png)

---

### 4.1 Create the Recipe

Go to **EC2 Image Builder → Image recipes → Create image recipe**

| Field | Value |
|---|---|
| Image type | Amazon Machine Image (AMI) |
| Name | `my-first-recipe` |
| Version | `1.0.0` |
| Base image | Managed images → Windows → Windows Server 2025 English Full Base x86 → **Use latest available OS version** |
| Build components | Amazon managed → `amazon-cloudwatch-agent-windows` (latest) |
| Test components | Amazon managed → `simple-boot-test-windows` |

{{% notice tip %}}
A recipe cannot be edited after creation. To change a component, create a new version (1.0.1, 1.0.2…).
{{% /notice %}}

![Recipe creation](/images/5-Workshop/5.4-imagebuilder/recipe.png)

---

### 4.2 Create the Pipeline

**1.** Go to **Image pipelines → Create image pipeline**  
Name: `my-first-pipeline`  
Enable: **Enhanced metadata collection** and **Enable EC2 image scanning**

**2.** Build schedule: keep default. For the lab, you can set **manual** to avoid unexpected costs.

**3.** Select the recipe created above. Image creation process: **Default workflows**.

**4.** Infrastructure configuration:
- Name: `workshop-infra-config`
- IAM role: `lab-imagebuilder-role`
- Instance type: `m7i-flex.large` (or `m5.large` as fallback)
- VPC, subnet, security group: use the ones from Step 1

**5.** Distribution: **Create distribution settings using service defaults** (private AMI in your account).

**6.** Review → **Create pipeline**

**7.** Image pipelines → select the pipeline → **Actions → Run pipeline**

Watch the status change: `Pending → Building → Testing → Available`  
This takes **15–30 minutes**. You will see a temporary build instance appear in EC2.

![Pipeline running](/images/5-Workshop/5.4-imagebuilder/pipeline-running.png)

![AMI available](/images/5-Workshop/5.4-imagebuilder/ami-available.png)

---

### 4.3 Lifecycle Policy

**EC2 Image Builder → Lifecycle policies → Create lifecycle policy**

| Field | Value |
|---|---|
| Resource type | AMI |
| Name | `my-ib-lifecycle-policy` |
| IAM role | `lab-imagebuilder-role` |
| Rule scope | Recipes → `my-first-recipe` |
| Lifecycle rule | Disable after 6 months |

This prevents stale AMIs and their EBS snapshots from accumulating.

---

### 4.4 Explore the Pipeline Internals

These are good screenshots for the academic section of your report:

**Component document:**  
EC2 Image Builder → Components → `amazon-cloudwatch-agent-windows`  
Examine the structure: `phases → steps → action (ExecutePowerShell…)`

**Build logs:**  
CloudWatch → Log groups → `/aws/imagebuilder/my-first-recipe*`  
Filter keywords: `Step`, `Std`, `CmdExecution`

**SSM Run Command history:**  
Systems Manager → Run Command → Command history  
Image Builder uses Run Command internally to install components.

![Component document structure](/images/5-Workshop/5.4-imagebuilder/component-doc.png)

![Build logs in CloudWatch](/images/5-Workshop/5.4-imagebuilder/build-logs.png)

---

### 4.5 Write a Custom Component (Advanced)

On a Windows instance (via Fleet Manager), install VS Code and download `awstoe.exe` from AWS documentation. Create `my-first-doc.yml`:

```yaml
name: Hello World
description: Hello world testing document for Windows.
schemaVersion: 1.0
phases:
  - name: 'build'
    steps:
      - name: SamplePowerShellStep
        action: ExecutePowerShell
        timeoutSeconds: 120
        onFailure: Abort
        maxAttempts: 3
        inputs:
          commands:
            - 'Write-Host "Hello world!"'
```

Validate and run locally:

```powershell
.\awstoe.exe validate -d .\my-first-doc.yml
.\awstoe.exe run -d .\my-first-doc.yml -t
```

**IIS test with Pester** — run before installing IIS (should fail), then after (should pass):

```powershell
Describe "IIS validation test" {
    BeforeAll { $httpResponse = Invoke-WebRequest localhost }
    It "HTTP Headers should have key: server, value: Microsoft-IIS" {
        $httpResponse.Headers['server'] | Should -Match 'Microsoft-IIS'
    }
    It "IIS should return HTTP 200 OK" {
        $httpResponse.StatusCode | Should -BeExactly 200
    }
}
```

Install IIS:

```powershell
Install-WindowsFeature -Name web-server
```

Then run the Pester test again — both tests should now pass.
