---
title: "Rà soát bảo mật và chi phí"
date: 2026-11-18
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Rà soát bảo mật và chi phí

Bước này xem lại cách Cost Guard xử lý bảo mật, chi phí và mức độ phù hợp với các trụ cột AWS Well-Architected.

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

**Không mở cổng nào.** Template không thêm quy tắc inbound, và máy lab dùng security group mặc định của default VPC.

**Cách kiểm tra:** mở **IAM → Roles**, chọn từng role `CostGuard` và đọc policy. Hành động duy nhất đụng tới trạng thái EC2 là `StopInstances` trên một instance.

### 2. Dự toán chi phí

Dự toán được lập bằng AWS Pricing Calculator cho US East (N. Virginia), giá on-demand và 120 giờ lab mỗi tháng. Giá tính không có Free Tier.

- Amazon EC2 Windows (t3.medium): 120 giờ × $0.06 = **$7.20**
- Amazon EBS (gp3, 30 GB): **$2.40**, tính cả tháng kể cả khi máy tắt
- Địa chỉ Public IPv4: 120 giờ × $0.005 = **$0.60**
- AWS Lambda: khoảng **$0.02**
- Amazon DynamoDB (on-demand): khoảng **$0.002**
- Amazon CloudWatch (3 alarm và log): khoảng **$0.31**
- EventBridge Scheduler và Amazon SNS: mỗi dịch vụ dưới **$0.01**

Các dịch vụ trong Calculator (EC2, EBS, Lambda và DynamoDB) cộng lại khoảng **$9.62 mỗi tháng**. Khi thêm các dịch vụ còn lại, tổng là khoảng **$10.54 mỗi tháng**, dưới ngân sách $20. Cũng máy đó chạy 24/7 sẽ tốn khoảng $46.22 mỗi tháng, nên Cost Guard giúp tránh khoảng $36.60 mỗi tháng.

### 3. Tối ưu chi phí

- Giới hạn giờ chạy. Lịch dừng và việc bật máy thủ công giữ máy ở gần 120 giờ thay vì 730.
- Dùng các dịch vụ trả theo lượt dùng và nằm trong hạn mức miễn phí: Lambda, DynamoDB on-demand, SNS và EventBridge Scheduler.
- Dùng tham số standard của Parameter Store, không tốn phí.
- Giữ ổ gp3 30 GB và không thêm IOPS hay throughput.
- Không gắn Elastic IP, vì bị tính phí khi đang được cấp phát.
- Xóa toàn bộ tài nguyên sau workshop (xem 5.8).

### 4. Các trụ cột Well-Architected

- **Operational Excellence:** log Lambda, CloudWatch alarm và bản ghi phiên trong DynamoDB cho thấy hệ thống đã làm gì và vì sao.
- **Security:** mỗi hàm một role, quyền dừng giới hạn cho một instance, không có thông tin xác thực trong code.
- **Reliability:** ba lớp độc lập, và mọi hành động đều an toàn khi lặp lại.
- **Performance Efficiency:** t3.medium cho một phiên lab, và hàm Lambda 128 MB chạy khoảng một giây.
- **Cost Optimization:** giới hạn giờ chạy, dịch vụ trả theo lượt dùng và dọn dẹp ở cuối.
