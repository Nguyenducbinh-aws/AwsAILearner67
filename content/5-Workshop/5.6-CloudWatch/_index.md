---
title: "Amazon CloudWatch"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Step 4 – Amazon CloudWatch

**Goal:** Collect Windows logs and custom metrics, create a metric filter counting HTTP 403 errors, trigger an SNS email alert, and build a dashboard.

---

### 6.1 Metrics

**1.** Go to **CloudWatch → Metrics → All metrics → EC2 → Per-Instance Metrics**  
A *dimension* is a key/value pair (e.g., `InstanceId`) used to filter metrics.

**2.** Select **CPUUtilization** and **EBSWriteBytes** for one instance → **Add to graph**  
Since the two metrics have different units, move EBSWriteBytes to the right Y-axis (Graphed metrics tab). Set the left axis: min 0, max 100, label `Utilization (%)`.

**3.** Search expression (auto-updates as instances change):

```
SEARCH('{AWS/EC2,InstanceId} MetricName="CPUUtilization"', 'Average', 300)
```

**4.** Math expression — top 3 instances by total network packets:

```
SEARCH('NetworkPackets', 'Average', 300)
SORT(e1, SUM, DESC, 3)
```

*(Replace `e1` with the actual expression ID shown in the console.)*

**5.** Dynamic labels for readable legends:

```
SEARCH('exe="cloudwatch" MetricName="procstat memory_rss"', 'Average', 300)
```

Label format: `${PROP('Dim.exe')} - ${PROP('Dim.InstanceId')} - ${PROP('MetricName')}`

![CloudWatch metrics graph](/images/5-Workshop/5.6-cloudwatch/metrics.png)

---

### 6.2 Logs

**1.** Go to **CloudWatch → Logs → Log groups**  
View `/windows/amazon-cloudwatch-agent.log` and `/windows/eventlog/*`  
Log streams are named by Instance ID.

**2.** Set retention: change **Never expire** to **7 days** to reduce cost.

**3.** Simulate HTTP 403 logs: run the following script on an instance via Session Manager to write fake log entries that the CloudWatch Agent will pick up:

```powershell
$logPath = "C:\inetpub\logs\LogFiles\HTTPERR\httperr1.log"
1..10 | ForEach-Object {
    Add-Content -Path $logPath -Value "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') 127.0.0.1 GET /admin HTTP/1.1 403 0 - -"
}
```

**4.** Logs Insights query:

```
fields @timestamp, @message
| filter @message like /403/
| stats count() as errors by bin(1m)
```

Save this query as `HTTP 403 Responses` in folder `operate-windows-workshop` to pin it to the dashboard later.

![Logs Insights query result](/images/5-Workshop/5.6-cloudwatch/logs-insights.png)

---

### 6.3 Metric Filter

Go to **Log groups → /windows/httperr → Actions → Create metric filter**

| Field | Value |
|---|---|
| Filter pattern | `HTTP 403` |
| Filter name | `http403` |
| Namespace | `ec2-logs` |
| Metric name | `/windows/webserver/http403` |
| Metric value | `1` |
| Default value | `0` |
| Unit | `Count` |

Use **Test pattern** with an actual log stream to confirm the filter matches.

{{% notice warning %}}
**Troubleshooting note:** When creating the alarm (next step), make sure to select the exact same metric name and namespace you set here (`ec2-logs` / `/windows/webserver/http403`). A mismatch between the metric filter name and the alarm metric is a common mistake.
{{% /notice %}}

![Metric filter creation](/images/5-Workshop/5.6-cloudwatch/metric-filter.png)

---

### 6.4 Alarm and SNS

**1.** Go to **Alarms → All alarms → Create alarm → Select metric**  
Navigate to: `ec2-logs → Metrics with no dimensions` → select the metric created above.

| Setting | Value |
|---|---|
| Statistic | Sum |
| Period | 1 minute |
| Condition | Greater than 3 |

**2.** Notification: create a new SNS topic with your email, name the alarm `LogonFailureAlarm`.  
Confirm the email subscription (SNS will not send alerts until you click Confirm in the email).

**3.** Also create an alarm for **CPU Utilization** (threshold: 80%) as a second example.

![Alarm in ALARM state](/images/5-Workshop/5.6-cloudwatch/alarm.png)

---

### 6.5 Dashboard

**1.** Go to **Dashboards → Create dashboard**, name: `operate-windows-workshop`

**2.** Add widgets in this order:
- The alarm (Alarms → Actions → Add to dashboard)
- Metric: `MyCustomNamespace → Memory Available Bytes`
- Metric: `ec2-logs → /windows/webserver/http403`
- Logs Insights saved query: `HTTP 403 Responses`

**3.** Try different widget types (line, number, gauge), resize them, then **Save dashboard**.

![Dashboard with all widgets](/images/5-Workshop/5.6-cloudwatch/dashboard.png)
