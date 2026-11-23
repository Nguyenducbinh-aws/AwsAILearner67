---
title: "Rà soát bảo mật và chi phí"
date: 2026-11-18
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Rà soát bảo mật và chi phí

Bước này xem lại cách Cost Guard xử lý bảo mật, chi phí và cách giữ chi phí ở mức thấp.

### 1. Bảo mật

**Mỗi hàm một IAM role.** Ba hàm Lambda có role riêng, nên một lỗi ở hàm này không dùng được quyền của hàm khác. Quyền dừng instance bị giới hạn theo **ARN của instance được bảo vệ**:

```json
{
  "Effect": "Allow",
  "Action": "ec2:StopInstances",
  "Resource": "arn:aws:ec2:us-east-1:<account-id>:instance/<instance-id>"
}
```

`ec2:DescribeInstances` được cho phép trên mọi tài nguyên, vì hành động này không hỗ trợ giới hạn theo tài nguyên. Các quyền còn lại chỉ áp dụng cho tham số, bảng và topic mà hàm đó dùng.

**Không có thông tin xác thực trong code.** Các hàm dùng execution role. Giá trị trong Parameter Store chỉ gồm đơn giá theo giờ, ngưỡng, hạn mức ngân sách và instance ID.

**Các dịch vụ gọi nhau bằng role riêng.** EventBridge Scheduler dùng một role chỉ gọi được hai hàm. Topic policy của `costguard-budgets` chỉ cho dịch vụ Budgets của chính tài khoản bạn gửi tin.

**Không mở cổng nào.** Máy lab trong workshop này dùng security group riêng `costguard-lab-no-inbound`, không có quy tắc inbound nào, và bắt buộc IMDSv2. Nếu để template tự tạo máy, máy sẽ dùng security group mặc định của default VPC, chỉ cho phép lưu lượng từ các máy cùng group.

**Cách kiểm tra:** mở **IAM → Roles**, chọn từng role `CostGuard` và đọc policy. Hành động duy nhất đụng tới trạng thái EC2 là `StopInstances` trên một instance.

### 2. Dự toán chi phí

Dự toán được lập bằng AWS Pricing Calculator cho US East (N. Virginia), giá on-demand và 120 giờ lab mỗi tháng. Giá tính không có Free Tier.

- Amazon EC2 Windows (t3.small): 120 giờ × $0.0392 = **$4.70**
- Amazon EBS (gp3, 30 GB): **$2.40**, tính cả tháng kể cả khi máy tắt
- Địa chỉ Public IPv4: 120 giờ × $0.005 = **$0.60**
- AWS Lambda: khoảng **$0.02**
- Amazon DynamoDB (on-demand): khoảng **$0.002**
- Amazon CloudWatch (3 alarm và log): khoảng **$0.31**
- EventBridge Scheduler và Amazon SNS: mỗi dịch vụ dưới **$0.01**

Các dịch vụ trong Calculator (EC2, EBS và Lambda) cộng lại **$7.12 mỗi tháng** ([bản ước tính](https://calculator.aws/#/estimate?id=641acc2f03b075415ec10d1ab0de1590ab659dd9)). Khi thêm các dịch vụ còn lại, tổng là khoảng **$8.05 mỗi tháng**, dưới ngân sách $20. Cũng máy đó chạy 24/7 sẽ tốn khoảng $31.04 mỗi tháng, nên Cost Guard giúp tránh khoảng $23.92 mỗi tháng.

### 3. Tối ưu chi phí

- Giới hạn giờ chạy. Lịch dừng và việc bật máy thủ công giữ máy ở gần 120 giờ thay vì 730.
- Dùng các dịch vụ trả theo lượt dùng và nằm trong hạn mức miễn phí: Lambda, DynamoDB on-demand, SNS và EventBridge Scheduler.
- Dùng tham số standard của Parameter Store, không tốn phí.
- Giữ ổ gp3 30 GB và không thêm IOPS hay throughput.
- Không gắn Elastic IP, vì bị tính phí khi đang được cấp phát.
- Xóa toàn bộ tài nguyên sau workshop (xem 5.8).
