---
title: "Tổng quan Workshop"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Tổng quan Workshop

**"Quản lý, Giám sát và Vận hành Microsoft Workload An toàn trên AWS"**

### Vấn đề cần giải quyết

Quản lý fleet Windows Server trên AWS theo cách thủ công rất dễ xảy ra lỗi: patch không đồng nhất, không có khả năng quan sát tập trung, và RDP mở ra Internet làm tăng rủi ro bảo mật cũng như chi phí vận hành.

Workshop này giải quyết các vấn đề đó bằng các dịch vụ AWS thuần túy — không phần mềm bên thứ ba, không mở cổng inbound.

### Những gì bạn sẽ xây dựng

Sau khi hoàn thành workshop, bạn sẽ có:

- Một **golden AMI** (Windows Server 2025 + CloudWatch Agent) được xây dựng và quản lý phiên bản bởi EC2 Image Builder
- **Hai EC2 instance Windows** trong private subnet, quản lý hoàn toàn qua SSM — không mở RDP ra Internet
- **Systems Manager** quản lý fleet theo tag: Run Command, State Manager associations, Patch Baseline, Parameter Store
- **CloudWatch** thu thập Windows Event Log và IIS log, metric filter đếm lỗi HTTP 403, alarm, thông báo email SNS, và dashboard
- **AWS Config** kiểm soát AMI được duyệt và tag bắt buộc
- **Amazon Inspector** quét lỗ hổng CVE

### Kiến trúc

![Sơ đồ kiến trúc](/images/5-Workshop/5.1-overview/architecture.png)

> Hãy tự vẽ sơ đồ kiến trúc bằng [draw.io](https://draw.io) với bộ [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/). Không dùng sơ đồ do AI tạo ra.

### Các quyết định thiết kế chính

| Quyết định | Lý do |
|---|---|
| VPC Endpoints thay vì NAT Gateway | Rẻ hơn và giữ traffic không đi qua Internet công cộng |
| Không mở RDP/SSH inbound | Chỉ truy cập qua Session Manager và Fleet Manager |
| Targeting theo tag trong SSM | Mở rộng được cho bất kỳ số lượng instance nào mà không cần liệt kê ID |
| State Manager associations | Đảm bảo cấu hình CloudWatch Agent và cập nhật SSM Agent được áp dụng theo lịch, không chỉ một lần |
| Lifecycle policy trên Image Builder | Tránh tích lũy AMI cũ và EBS snapshot không dùng |

### Dịch vụ và các Pillar (Well-Architected)

| Dịch vụ | Well-Architected Pillar |
|---|---|
| EC2 Image Builder | Operational Excellence – image nhất quán, có version |
| Systems Manager | Operational Excellence – quản lý fleet tập trung |
| CloudWatch | Reliability – quan sát, cảnh báo |
| AWS Config | Security – kiểm soát tuân thủ |
| Amazon Inspector | Security – phát hiện lỗ hổng |
| VPC Endpoints | Security + Cost Optimization |
