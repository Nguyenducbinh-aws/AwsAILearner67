---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Quản lý, Giám sát và Vận hành Microsoft Workload An toàn trên AWS

#### Tổng quan

Workshop này hướng dẫn cách xây dựng, quản lý, giám sát và bảo mật Windows Server workload trên AWS bằng các dịch vụ AWS thuần túy — không mở cổng RDP ra Internet.

Bạn sẽ dùng **EC2 Image Builder** để tạo golden AMI đã được hardened, **AWS Systems Manager** để quản lý fleet theo tag, **Amazon CloudWatch** để thu thập log, metric và cảnh báo, và **AWS Config + Amazon Inspector** để kiểm soát tuân thủ bảo mật.

#### Các dịch vụ AWS sử dụng

| Dịch vụ | Mục đích |
|---|---|
| EC2 Image Builder | Tạo golden AMI với CloudWatch Agent được cài sẵn |
| AWS Systems Manager | Fleet Manager, Run Command, State Manager, Patch Manager, Parameter Store, Session Manager |
| Amazon CloudWatch | Metrics, Logs, Metric Filters, Alarms, Dashboards |
| Amazon SNS | Cảnh báo email từ alarm và sự kiện Config |
| AWS Config | Kiểm soát AMI được duyệt và tag bắt buộc |
| Amazon Inspector | Quét lỗ hổng CVE |
| Amazon VPC + Endpoints | Mạng riêng, không expose RDP ra ngoài |

#### Kiến trúc

![Sơ đồ kiến trúc Workshop](/images/5-Workshop/5.1-overview/architecture.png)

#### Nội dung

1. [Tổng quan Workshop](5.1-Workshop-overview/)
2. [Yêu cầu tiên quyết](5.2-Prerequiste/)
3. [Dựng môi trường nền](5.3-Environment/)
4. [EC2 Image Builder – Golden AMI](5.4-ImageBuilder/)
5. [AWS Systems Manager](5.5-SystemsManager/)
6. [Amazon CloudWatch](5.6-CloudWatch/)
7. [Bảo mật – Config & Inspector](5.7-Security/)
8. [Dọn dẹp tài nguyên](5.8-Cleanup/)
