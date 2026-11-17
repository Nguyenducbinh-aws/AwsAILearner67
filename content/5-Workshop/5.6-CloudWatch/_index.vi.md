---
title: "Amazon CloudWatch"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Bước 4 – Amazon CloudWatch

**Mục tiêu:** Thu thập log Windows và custom metric, tạo metric filter đếm lỗi HTTP 403, kích hoạt cảnh báo email SNS, và xây dựng dashboard.

---

### 6.1 Metrics

**1.** Vào **CloudWatch → Metrics → All metrics → EC2 → Per-Instance Metrics**  
*Dimension* là cặp key/value (ví dụ `InstanceId`) dùng để lọc metric.

**2.** Chọn **CPUUtilization** và **EBSWriteBytes** cho một instance → **Add to graph**  
Vì hai metric có đơn vị khác nhau, chuyển EBSWriteBytes sang trục Y phải (tab Graphed metrics). Đặt trục trái: min 0, max 100, nhãn `Utilization (%)`.

**3.** Search expression (tự cập nhật khi instance thay đổi):

```
SEARCH('{AWS/EC2,InstanceId} MetricName="CPUUtilization"', 'Average', 300)
```

**4.** Math expression — top 3 instance theo tổng network packet:

```
SEARCH('NetworkPackets', 'Average', 300)
SORT(e1, SUM, DESC, 3)
```

*(Thay `e1` bằng ID expression thực tế hiển thị trong console.)*

**5.** Dynamic label cho legend dễ đọc:

```
SEARCH('exe="cloudwatch" MetricName="procstat memory_rss"', 'Average', 300)
```

Label format: `${PROP('Dim.exe')} - ${PROP('Dim.InstanceId')} - ${PROP('MetricName')}`

![Đồ thị CloudWatch metrics](/images/5-Workshop/5.6-cloudwatch/metrics.png)

---

### 6.2 Logs

**1.** Vào **CloudWatch → Logs → Log groups**  
Xem `/windows/amazon-cloudwatch-agent.log` và `/windows/eventlog/*`  
Log stream được đặt tên theo Instance ID.

**2.** Đặt retention: đổi **Never expire** thành **7 ngày** để tiết kiệm chi phí.

**3.** Mô phỏng log HTTP 403: chạy script sau trên instance qua Session Manager để ghi log giả mà CloudWatch Agent sẽ thu thập:

```powershell
$logPath = "C:\inetpub\logs\LogFiles\HTTPERR\httperr1.log"
1..10 | ForEach-Object {
    Add-Content -Path $logPath -Value "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') 127.0.0.1 GET /admin HTTP/1.1 403 0 - -"
}
```

**4.** Truy vấn Logs Insights:

```
fields @timestamp, @message
| filter @message like /403/
| stats count() as errors by bin(1m)
```

Lưu truy vấn này tên `HTTP 403 Responses` trong folder `operate-windows-workshop` để ghim lên dashboard sau.

![Kết quả truy vấn Logs Insights](/images/5-Workshop/5.6-cloudwatch/logs-insights.png)

---

### 6.3 Metric Filter

Vào **Log groups → /windows/httperr → Actions → Create metric filter**

| Trường | Giá trị |
|---|---|
| Filter pattern | `HTTP 403` |
| Filter name | `http403` |
| Namespace | `ec2-logs` |
| Metric name | `/windows/webserver/http403` |
| Metric value | `1` |
| Default value | `0` |
| Unit | `Count` |

Dùng **Test pattern** với log stream thực tế để xác nhận filter khớp đúng.

{{% notice warning %}}
**Ghi chú troubleshooting:** Khi tạo alarm (bước tiếp theo), hãy chắc chắn chọn đúng metric name và namespace bạn vừa đặt (`ec2-logs` / `/windows/webserver/http403`). Nhầm tên giữa metric filter và alarm metric là lỗi phổ biến nhất.
{{% /notice %}}

![Tạo metric filter](/images/5-Workshop/5.6-cloudwatch/metric-filter.png)

---

### 6.4 Alarm và SNS

**1.** Vào **Alarms → All alarms → Create alarm → Select metric**  
Điều hướng đến: `ec2-logs → Metrics with no dimensions` → chọn metric vừa tạo.

| Cài đặt | Giá trị |
|---|---|
| Statistic | Sum |
| Period | 1 phút |
| Condition | Greater than 3 |

**2.** Notification: tạo SNS topic mới với email của bạn, đặt tên alarm `LogonFailureAlarm`.  
Xác nhận email subscription (SNS không gửi cảnh báo cho đến khi bạn nhấn Confirm trong email).

**3.** Cũng tạo thêm alarm cho **CPU Utilization** (ngưỡng 80%) làm ví dụ thứ hai.

![Alarm ở trạng thái ALARM](/images/5-Workshop/5.6-cloudwatch/alarm.png)

---

### 6.5 Dashboard

**1.** Vào **Dashboards → Create dashboard**, tên: `operate-windows-workshop`

**2.** Thêm widget theo thứ tự:
- Alarm (Alarms → Actions → Add to dashboard)
- Metric: `MyCustomNamespace → Memory Available Bytes`
- Metric: `ec2-logs → /windows/webserver/http403`
- Logs Insights query đã lưu: `HTTP 403 Responses`

**3.** Thử các loại widget khác nhau (line, number, gauge), chỉnh kích thước, rồi **Save dashboard**.

![Dashboard với tất cả widget](/images/5-Workshop/5.6-cloudwatch/dashboard.png)
