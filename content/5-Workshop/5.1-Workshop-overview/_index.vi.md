---
title: "Giới thiệu"
date: 2026-11-18
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu về Cost Guard

Máy Windows EC2 tính phí theo giờ. Một máy t3.small chạy 24/7 tốn khoảng $31.04 mỗi tháng, vượt quá ngân sách credit $20 của dự án. AWS Budgets có cảnh báo chi phí, nhưng dữ liệu billing chỉ cập nhật vài lần mỗi ngày, nên tự nó không kịp dừng một máy bị quên.

**Cost Guard** là hệ thống serverless tự động dừng máy lab. Hệ thống dùng ba lớp độc lập, mỗi lớp bù đắp điểm yếu của các lớp còn lại. Lớp nào cũng dừng được máy, không lớp nào bật được máy, và dừng một máy đã dừng thì không có tác dụng.

- **Lớp 1 – Schedule Guard:** dừng instance mỗi ngày lúc 22:00 (Asia/Ho_Chi_Minh), dù có ai nhớ hay không.
- **Lớp 2 – Cost Meter:** chạy mỗi 5 phút, ước tính chi phí tháng từ thời gian chạy và đơn giá theo giờ, rồi dừng instance khi ước tính chạm ngưỡng khoảng $14 (70% của $20).
- **Lớp 3 – Budget Backstop:** một AWS Budget theo dõi hóa đơn thực, và ở mức 80% và 100% của $20 sẽ kích hoạt một hàm Lambda dừng instance.

#### Tổng quan về workshop

Trong workshop này, bạn triển khai Cost Guard bằng một template CloudFormation, cố ý kiểm thử từng lớp, rà soát bảo mật và chi phí, rồi xóa toàn bộ ở cuối để không còn phát sinh phí.

![Sơ đồ kiến trúc](../../../images/2-Proposal/architecture.png)

Các số trong sơ đồ theo đúng thứ tự luồng:

1. Người dùng lab bật EC2 Windows thủ công.
2. Lớp 1: EventBridge Scheduler gọi ScheduleHandler khi hết giờ lab.
3. Lớp 2: một lịch EventBridge Scheduler gọi CostMeter mỗi 5 phút.
4. Lớp 3: cảnh báo của AWS Budgets đi qua SNS tới BudgetBackstop.
5. Các hàm Lambda đọc cấu hình từ Parameter Store, đọc hoặc ghi bản ghi phiên trong DynamoDB.
6. Một hàm Lambda dừng EC2 khi hết giờ lab, chi phí vượt ngưỡng hoặc budget chạm hạn mức.
7. SNS gửi email cảnh báo cho người dùng.
8. CloudWatch Alarms theo dõi lỗi Lambda và gửi qua SNS cho người dùng.

Bạn sẽ xây dựng các tài nguyên sau:

- Một **EC2 Windows** (t3.small) với ổ gp3 30 GB làm máy lab được bảo vệ
- Ba **hàm Lambda** cho ba lớp, với **EventBridge Scheduler** và **AWS Budgets** làm trigger
- **Parameter Store** cho đơn giá, ngưỡng và hạn mức ngân sách, và bảng **DynamoDB** cho bản ghi phiên
- Hai **SNS topic** (cảnh báo và lỗi) và một **IAM role** quyền tối thiểu cho từng hàm
- **CloudWatch Logs** và **CloudWatch Alarms** cho lỗi của cả ba hàm

Ngưỡng của Lớp 2 chỉ tính chi phí chạy EC2. Ổ EBS ($2.40 mỗi tháng) tính riêng. Sau khi Lớp 2 dừng máy, tổng chi phí trong tháng vẫn trên ngưỡng, nên máy bị khóa đến tháng dương lịch tiếp theo.
