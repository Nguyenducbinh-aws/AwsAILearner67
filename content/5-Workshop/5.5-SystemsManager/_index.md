---
title: "AWS Systems Manager"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Step 3 – AWS Systems Manager

**Goal:** Manage the Windows fleet by tag using Run Command, Parameter Store, State Manager, and Patch Manager — with no open inbound ports.

---

### 5.1 Run Command

**1.** Go to **Systems Manager → Node tools → Run Command → Run command**  
Document: `AWS-RunPowerShellScript`

**2.** Commands:
```powershell
whoami
Get-Process *agent*
```

**3.** Targets: **Specify instance tags** → Key: `workshop`, Value: `module2` → Add

**4.** Output options: disable S3, enable **CloudWatch Logs**, log group `/ssm/runcommand`

**5.** Click **Run** → wait for **Success** → open the Instance ID to see output  
(The user will be `nt authority\system`)

CLI equivalent:

```bash
aws ssm send-command \
  --document-name "AWS-RunPowerShellScript" \
  --targets "Key=tag:workshop,Values=module2" \
  --parameters 'commands=["whoami","Get-Process *agent*"]' \
  --cloud-watch-output-config CloudWatchOutputEnabled=true,CloudWatchLogGroupName=/ssm/runcommand
```

Also run `AWS-UpdateSSMAgent` on the tagged instances using the same method.

![Run Command success](/images/5-Workshop/5.5-ssm/run-command.png)

---

### 5.2 Parameter Store

Create a parameter:

```bash
aws ssm put-parameter \
  --name module2 \
  --type String \
  --value '{"FirstName":"Peter","LastName":"Parker"}'

aws ssm get-parameter --name module2 --query "Parameter.Value" --output text
```

Read the parameter from inside the instance (Session Manager):

```powershell
$params = Get-SSMParameter -Name "module2" | Select-Object -expand Value | ConvertFrom-Json
"Hello {0} {1}!" -f $params.FirstName, $params.LastName
```

---

### 5.3 CloudWatch Agent Config in Parameter Store

Create a parameter named `AmazonCloudWatch-windows` with the agent configuration. This tells the agent which metrics and logs to collect:

```json
{
  "agent": { "metrics_collection_interval": 60 },
  "metrics": {
    "namespace": "MyCustomNamespace",
    "metrics_collected": {
      "LogicalDisk": {
        "measurement": ["% Free Space"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "Memory": {
        "measurement": ["Available Bytes"],
        "metrics_collection_interval": 60
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "C:\\inetpub\\logs\\LogFiles\\HTTPERR\\*.log",
            "log_group_name": "/windows/httperr",
            "log_stream_name": "{instance_id}"
          }
        ]
      },
      "windows_events": {
        "collect_list": [
          {
            "event_name": "Application",
            "event_levels": ["ERROR","WARNING","CRITICAL"],
            "log_group_name": "/windows/eventlog/application",
            "log_stream_name": "{instance_id}"
          },
          {
            "event_name": "System",
            "event_levels": ["ERROR","WARNING","CRITICAL"],
            "log_group_name": "/windows/eventlog/system",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

Save to file `cwagent-windows.json`, then:

```bash
aws ssm put-parameter \
  --name AmazonCloudWatch-windows \
  --type String \
  --value file://cwagent-windows.json \
  --overwrite
```

---

### 5.4 State Manager

Create three associations — all targeting tag `workshop=module2`:

| Association | Document | Key Config | Schedule |
|---|---|---|---|
| AmazonCloudWatchSetConfig | `AmazonCloudWatch-ManageAgent` | action: configure, source: ssm, location: AmazonCloudWatch-windows, restart agent | every 30 min |
| UpdateSSMAgent | `AWS-UpdateSSMAgent` | allowDowngrade: false | daily 00:00 |
| InstallCloudWatchAgent | `AWS-ConfigureAWSPackage` | action: Install, name: AmazonCloudWatchAgent | daily 01:00 |

{{% notice tip %}}
For the SSM Agent update association, choose **"Apply association only at the next specified cron interval"** — this prevents upgrading SSM Agent immediately on a new boot, which can cause other associations to fail.
{{% /notice %}}

After running, verify in **State Manager → Association → Execution history** and in **Run Command → Command history** (State Manager uses Run Command internally).

![State Manager associations](/images/5-Workshop/5.5-ssm/state-manager.png)

---

### 5.5 Patch Manager

**1.** Go to **Systems Manager → Patch Manager → Patch baselines → Create patch baseline**

| Field | Value |
|---|---|
| Name | `workshop-Windows22-PatchBaseline` |
| OS | Windows |
| Product | WindowsServer2022 |
| Severity | All |
| Classification | All |

**2.** **Patch now** → Operation: **Scan** → Target: tag `workshop=module2`

**3.** View results in **Compliance reporting** tab. Click an instance ID to open Fleet Manager → **Patch** tab to see installed patches.

{{% notice warning %}}
A patch baseline only defines which patches are approved — it does not apply them automatically. **Scan** only reports status. To actually patch, choose **Scan and install** (instances may reboot — only do this on lab machines).
{{% /notice %}}

![Patch compliance report](/images/5-Workshop/5.5-ssm/patch-compliance.png)
