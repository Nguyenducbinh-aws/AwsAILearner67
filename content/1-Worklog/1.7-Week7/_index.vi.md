---
title: "Worklog Tuần 7"
date: 2026-10-19
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---


### Mục tiêu tuần 7:

* Tìm hiểu Amazon CloudWatch: metric, log, alarm và dashboard.
* Tìm hiểu cách hybrid DNS giữa hệ thống DNS on-premises và Amazon Route 53 hoạt động.
* Thực hành với CloudFormation template và dọn tài nguyên sau mỗi lab.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Đọc tổng quan về Amazon CloudWatch <br>- **Thực hành:** <br>&emsp; + Triển khai ba EC2 có CloudWatch agent từ CloudFormation template <br>&emsp; + Xem metric <br>&emsp; + Dùng search expression, math expression và dynamic label | 19/10/2026 | 19/10/2026 | <https://000008.awsfcaj.com/> |
| 3 | - **Thực hành:** <br>&emsp; + Đọc log và truy vấn bằng CloudWatch Logs Insights <br>&emsp; + Tạo metric filter đếm dòng lỗi <br>&emsp; + Tạo alarm gửi email qua Amazon SNS | 20/10/2026 | 20/10/2026 | <https://000008.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Tạo CloudWatch dashboard <br>&emsp; + Dọn tài nguyên sau lab | 21/10/2026 | 21/10/2026 | <https://000008.awsfcaj.com/> |
| 5 | - Đọc tổng quan về hybrid DNS và Route 53 Resolver <br>- **Thực hành:** <br>&emsp; + Tạo key pair và dựng mạng bằng CloudFormation template <br>&emsp; + Cấu hình security group và kết nối Remote Desktop Gateway <br>&emsp; + Triển khai AWS Managed Microsoft AD để giả lập DNS on-premises | 22/10/2026 | 22/10/2026 | <https://000010.awsfcaj.com/> |
| 6 | - **Thực hành:** <br>&emsp; + Tạo outbound endpoint và resolver rule của Route 53 Resolver <br>&emsp; + Tạo inbound endpoint của Route 53 Resolver <br>&emsp; + Kiểm tra phân giải tên bằng nslookup và Resolve-DnsName <br>&emsp; + Dọn tài nguyên sau lab | 23/10/2026 | 23/10/2026 | <https://000010.awsfcaj.com/> |


### Kết quả đạt được tuần 7:

* Hiểu Amazon CloudWatch làm gì: thu thập metric và log, cho phép truy vấn và vẽ biểu đồ, và tạo alarm khi có sự cố.

* Triển khai lab CloudWatch từ CloudFormation template:

  * Ba EC2 chạy Amazon Linux 2023 với một workload mẫu và CloudWatch agent
  * IAM role chỉ có đúng quyền agent cần, thay cho AdministratorAccess

* Làm việc với metric và log:

  * Kết hợp metric bằng search expression, math expression và dynamic label
  * Truy vấn log bằng CloudWatch Logs Insights
  * Tạo metric filter đếm các dòng ERROR của ứng dụng và đưa số đếm thành metric (filter chỉ đếm các sự kiện đến sau khi nó được tạo)

* Tạo alarm và dashboard:

  * Alarm trên số lỗi, gửi email qua Amazon SNS khi lỗi vượt ngưỡng
  * Dashboard gom các metric vào một chỗ

* Hiểu hybrid DNS với Route 53 Resolver:

  * Outbound endpoint gửi truy vấn từ AWS sang DNS on-premises
  * Inbound endpoint nhận truy vấn từ on-premises cho các domain trên AWS
  * Resolver rule quyết định domain nào được chuyển tiếp

* Dựng lab hybrid DNS:

  * Dựng mạng (VPC, public và private subnet, Remote Desktop Gateway) bằng CloudFormation template
  * Triển khai AWS Managed Microsoft AD với domain onprem.example.com để thay cho DNS on-premises
  * Tạo outbound endpoint, inbound endpoint và resolver rule
  * Kiểm tra từ Remote Desktop Gateway rằng tên miền phân giải được bằng nslookup và Resolve-DnsName

* Dọn tài nguyên sau mỗi lab, vì mạng và instance của lab vẫn tốn tiền khi còn chạy.
