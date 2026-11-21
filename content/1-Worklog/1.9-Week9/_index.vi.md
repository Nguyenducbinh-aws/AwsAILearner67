---
title: "Worklog Tuần 9"
date: 2026-11-02
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---


### Mục tiêu tuần 9:

* Hiểu cách tự động hóa việc nâng cấp Windows Server bằng AWS Systems Manager Automation.
* Triển khai WordPress trên AWS với EC2 ở public subnet, database ở private subnet và Multi-AZ.
* Thử Auto Scaling, Load Balancer, backup và restore, CloudFront trên cùng một ứng dụng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Đọc blog về tự động hóa nâng cấp Windows Server bằng AWS Systems Manager | 02/11/2026 | 06/11/2026 | <https://aws.amazon.com/vi/blogs/modernizing-with-aws/how-to-automate-your-microsoft-windows-server-upgrades-using-aws-systems-manager-update/> |
| 3 | - Đọc tổng quan lab WordPress on AWS <br>- **Thực hành:** <br>&emsp; + Tạo VPC và subnet <br>&emsp; + Tạo security group cho EC2 và cho database <br>&emsp; + Tạo EC2 và RDS database | 03/11/2026 | 03/11/2026 | <https://000021.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Cài WordPress trên EC2 và kết nối với database | 04/11/2026 | 04/11/2026 | <https://000021.awsfcaj.com/> |
| 5 | - **Thực hành:** <br>&emsp; + Tạo AMI và launch template <br>&emsp; + Tạo target group và load balancer <br>&emsp; + Tạo Auto Scaling group | 05/11/2026 | 05/11/2026 | <https://000021.awsfcaj.com/> |
| 6 | - **Thực hành:** <br>&emsp; + Tạo snapshot database và restore từ snapshot <br>&emsp; + Tạo CloudFront distribution <br>&emsp; + Dọn tài nguyên sau lab | 06/11/2026 | 06/11/2026 | <https://000021.awsfcaj.com/> |


### Kết quả đạt được tuần 9:

* Học từ blog cách tự động hóa nâng cấp tại chỗ Windows Server bằng AWS Systems Manager Automation:

  * Windows Server 2012 và 2012 R2 hết hỗ trợ từ 10/10/2023, Windows Server 2016 hết hỗ trợ mở rộng vào 12/01/2027, nên server không còn được hỗ trợ sẽ không nhận bản vá bảo mật
  * Runbook AWSEC2-CloneInstanceAndUpgradeWindows sao chép instance rồi nâng cấp bản sao, kết quả là một AMI mới để tạo instance mới
  * Điều kiện: SSM Agent chạy được, bật TLS 1.2, trống ít nhất 20 GB ở ổ root, có IAM instance profile cho SSM và truy cập ra được SSM endpoint
  * Giới hạn: không hỗ trợ domain controller, cluster, hệ điều hành desktop và server có role Remote Desktop Session Host
  * Nâng cấp xong cần kiểm tra ứng dụng trước khi đưa lại vào production

* Hiểu các thành phần của kiến trúc WordPress on AWS: CloudFront để phân phối nội dung, Auto Scaling group và load balancer để mở rộng và chia tải, Amazon RDS Multi-AZ để database luôn sẵn sàng.

* Triển khai WordPress trên AWS:

  * Tạo mạng, security group và EC2 ở public subnet, database ở private subnet
  * Cài WordPress trên EC2 và kết nối với RDS
  * Tạo AMI, launch template, target group, load balancer và Auto Scaling group để ứng dụng mở rộng và cân bằng tải
  * Tạo snapshot database và restore từ snapshot
  * Đặt CloudFront phía trước ứng dụng
