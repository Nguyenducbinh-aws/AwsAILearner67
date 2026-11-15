---
title: "AWS Systems Manager"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Bước 3 – AWS Systems Manager

**Mục tiêu:** Quản lý fleet Windows theo tag bằng Run Command, Parameter Store, State Manager và Patch Manager — không mở cổng inbound nào.

---

### 5.1 Run Command

**1.** Vào **Systems Manager → Node tools → Run Command → Run command**  
Document: `AWS-RunPowerShellScript`

**2.** Commands:
```powershell
whoami
Get-Process *agent*
```

**3.** Targets: **Specify instance tags** → Key: `workshop`, Value: `module2` → Add

**4.** Output options: tắt S3, bật **CloudWatch Logs**, log group `/ssm/runcommand`

**5.** Nhấn **Run** → chờ **Success** → mở Instance ID để xem output  
(User sẽ là `nt authority\system`)

CLI tương đương:

```bash
aws ssm send-command \
  --document-name "AWS-RunPowerShellScript" \
  --targets "Key=tag:workshop,Values=module2" \
  --parameters 'commands=["whoami","Get-Process *agent*"]' \
  --cloud-watch-output-config CloudWatchOutputEnabled=true,CloudWatchLogGroupName=/ssm/runcommand
```

Cũng chạy `AWS-UpdateSSMAgent` trên các instance được gắn tag bằng cách tương tự.

![Run Command thành công](/images/5-Workshop/5.5-ssm/run-command.png)

---

### 5.2 Parameter Store

Tạo parameter:

```bash
aws ssm put-parameter \
  --name module2 \
  --type String \
  --value '{"FirstName":"Peter","LastName":"Parker"}'

aws ssm get-parameter --name module2 --query "Parameter.Value" --output text
```

Đọc parameter từ bên trong instance (Session Manager):

```powershell
$params = Get-SSMParameter -Name "module2" | Select-Object -expand Value | ConvertFrom-Json
"Hello {0} {1}!" -f $params.FirstName, $params.LastName
```

---

### 5.3 Cấu hình CloudWatch Agent trong Parameter Store

Tạo parameter tên `AmazonCloudWatch-windows` với cấu hình agent. Cấu hình này cho agent biết cần thu thập metrics và logs nào:

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

Lưu vào file `cwagent-windows.json`, rồi:

```bash
aws ssm put-parameter \
  --name AmazonCloudWatch-windows \
  --type String \
  --value file://cwagent-windows.json \
  --overwrite
```

---

### 5.4 State Manager

Tạo ba association — tất cả target theo tag `workshop=module2`:

| Association | Document | Cấu hình chính | Lịch |
|---|---|---|---|
| AmazonCloudWatchSetConfig | `AmazonCloudWatch-ManageAgent` | action: configure, source: ssm, location: AmazonCloudWatch-windows, restart agent | mỗi 30 phút |
| UpdateSSMAgent | `AWS-UpdateSSMAgent` | allowDowngrade: false | hằng ngày 00:00 |
| InstallCloudWatchAgent | `AWS-ConfigureAWSPackage` | action: Install, name: AmazonCloudWatchAgent | hằng ngày 01:00 |

{{% notice tip %}}
Với association nâng cấp SSM Agent, chọn **"Apply association only at the next specified cron interval"** — tránh nâng cấp SSM Agent ngay khi instance mới boot, có thể làm các association khác bị lỗi.
{{% /notice %}}

Sau khi chạy, kiểm tra ở **State Manager → Association → Execution history** và **Run Command → Command history** (State Manager dùng Run Command bên dưới).

![State Manager associations](/images/5-Workshop/5.5-ssm/state-manager.png)

---

### 5.5 Patch Manager

**1.** Vào **Systems Manager → Patch Manager → Patch baselines → Create patch baseline**

| Trường | Giá trị |
|---|---|
| Tên | `workshop-Windows22-PatchBaseline` |
| OS | Windows |
| Product | WindowsServer2022 |
| Severity | All |
| Classification | All |

**2.** **Patch now** → Operation: **Scan** → Target: tag `workshop=module2`

**3.** Xem kết quả ở tab **Compliance reporting**. Nhấn Instance ID để mở Fleet Manager → tab **Patch** xem bản vá.

{{% notice warning %}}
Patch baseline chỉ định nghĩa bản vá nào được duyệt — không tự động vá. **Scan** chỉ báo cáo trạng thái. Để vá thật, chọn **Scan and install** (instance có thể reboot — chỉ làm trên máy lab).
{{% /notice %}}

![Báo cáo tuân thủ Patch](/images/5-Workshop/5.5-ssm/patch-compliance.png)
