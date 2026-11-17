---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Bước 6 – Dọn dẹp tài nguyên

**Mục tiêu:** Xóa toàn bộ tài nguyên đã tạo trong workshop để tránh phát sinh chi phí AWS không mong muốn.

{{% notice warning %}}
Luôn kiểm tra **Billing → Cost Explorer** sau 1–2 ngày dọn dẹp để xác nhận không còn tài nguyên nào đang chạy.
{{% /notice %}}

---

### Thứ tự dọn dẹp

Làm theo thứ tự sau — một số tài nguyên phụ thuộc vào tài nguyên khác phải xóa trước.

**1. Amazon Inspector**

Tắt resource scanning trước khi tắt dịch vụ. Nếu bị lỗi, đợi vài phút rồi chạy lại:

```bash
aws inspector2 disable --resource-types EC2 ECR LAMBDA
aws inspector2 disable
```

**2. AWS Config**

Vào **AWS Config → Rules** → xóa cả hai rule (`infosec-approved-amis`, `required-tags`)  
Sau đó vào **Settings** → tắt recorder.

**3. EventBridge Rules**

Xóa EventBridge rule đã tạo cho thông báo Config.

**4. Amazon CloudWatch**

```bash
# Xóa alarm
aws cloudwatch delete-alarms --alarm-names LogonFailureAlarm

# Xóa log group (không tự xóa cùng stack — phải xóa thủ công)
aws logs delete-log-group --log-group-name /ssm/runcommand
aws logs delete-log-group --log-group-name /windows/httperr
aws logs delete-log-group --log-group-name /windows/eventlog/application
aws logs delete-log-group --log-group-name /windows/eventlog/system
```

Xóa dashboard: **CloudWatch → Dashboards → `operate-windows-workshop` → Delete**

**5. Amazon SNS**

```bash
# Thay bằng ARN topic thực tế của bạn
aws sns delete-topic --topic-arn <arn-topic-LogonFailureAlarm>
aws sns delete-topic --topic-arn <arn-topic-SecureWindowsNotifications>
```

**6. AWS Systems Manager**

```bash
# Xóa parameter
aws ssm delete-parameter --name module2
aws ssm delete-parameter --name AmazonCloudWatch-windows
```

Xóa State Manager associations: **Systems Manager → State Manager** → chọn từng association → **Delete**

**7. EC2 Instances**

Terminate tất cả instance lab:  
**EC2 → Instances** → chọn tất cả instance lab → **Instance state → Terminate**

**8. EC2 Image Builder**

Theo thứ tự:
- Xóa **Lifecycle policies**: `my-ib-lifecycle-policy`
- Xóa **Pipelines**: `my-first-pipeline`, `Windows-STIG-Pipeline`
- Xóa **Recipes**: `my-first-recipe`, `Secure Image`
- Xóa **Infrastructure configurations**: `workshop-infra-config`
- **Deregister AMIs**: EC2 → AMIs → chọn các AMI đã build → **Actions → Deregister AMI**
- **Xóa EBS Snapshots**: EC2 → Snapshots → xóa snapshot liên quan đến các AMI đã deregister

**9. IAM Roles**

Xóa role: `lab-windows-ssm-role`, `lab-imagebuilder-role`

**10. Mạng**

```bash
# Xóa VPC endpoint, NAT Gateway (nếu có), giải phóng Elastic IP
# Sau đó xóa subnet, route table, internet gateway, và VPC
```

Vào **VPC → Endpoints** → xóa tất cả endpoint lab  
Vào **VPC → Your VPCs** → xóa VPC lab (sẽ xóa cả subnet, route table, và security group)

---

### Xác nhận đã dọn dẹp

Sau khi hoàn thành các bước:

- [ ] Không còn EC2 instance nào đang chạy từ lab
- [ ] Không còn AMI nào từ Image Builder
- [ ] Không còn EBS snapshot nào từ lab
- [ ] Không còn NAT Gateway hoặc chi phí Elastic IP
- [ ] Inspector và Config đã tắt
- [ ] Log group đã xóa
- [ ] Kiểm tra **Billing → Cost Explorer** sau 1–2 ngày

![Cost Explorer sau khi dọn dẹp](/images/5-Workshop/5.8-cleanup/cost-explorer.png)
