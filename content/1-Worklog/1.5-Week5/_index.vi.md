---
title: "Worklog Tuần 5"
date: 2026-10-05
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---


### Mục tiêu tuần 5:

* Lên ý tưởng cho proposal và workshop cùng nhóm.
* Hiểu cơ bản về Amazon RDS và thực hành triển khai database.
* Tìm hiểu cách truy cập Windows instance bằng Fleet Manager và cách quan sát AI agent bằng CloudWatch Omni.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Lên ý tưởng cho proposal và workshop cùng với nhóm | 05/10/2026 | 05/10/2026 | |
| 3 | - Đọc tài liệu về Amazon RDS <br>&emsp; + Database engine được hỗ trợ <br>&emsp; + Multi-AZ, Read Replica <br>&emsp; + Backup và snapshot <br>&emsp; + Bảo mật: mã hóa, VPC, IAM | 06/10/2026 | 06/10/2026 | <https://000005.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Tạo VPC, Security Group và DB subnet group <br>&emsp; + Tạo RDS MySQL <br>&emsp; + Tạo EC2, kết nối RDS bằng MySQL client <br>&emsp; + Tạo database, tạo bảng và thêm dữ liệu <br>&emsp; + Triển khai ứng dụng kết nối RDS <br>&emsp; + Tạo snapshot và restore RDS <br>&emsp; + Dọn tài nguyên sau lab | 07/10/2026 | 07/10/2026 | <https://000005.awsfcaj.com/> |
| 5 | - Đọc blog về AWS Systems Manager Fleet Manager <br>&emsp; + Kết nối RDP vào Windows instance ngay trên trình duyệt qua Session Manager <br>&emsp; + Không cần mở cổng RDP, không cần public IP và bastion host <br>&emsp; + Dùng EC2 key pair hoặc Windows credentials, xem tối đa 4 instance cùng lúc <br>&emsp; + Cần SSM Agent, instance profile có quyền Systems Manager | 08/10/2026 | 08/10/2026 | <https://aws.amazon.com/vi/blogs/mt/console-based-access-to-windows-instances-using-aws-systems-manager-fleet-manager/> |
| 6 | - Đọc blog về Amazon CloudWatch Omni <br>&emsp; + Quan sát AI agent bằng trace ghi lại từng bước gọi LLM và tool <br>&emsp; + 17 evaluator có sẵn để chấm chất lượng phản hồi <br>&emsp; + Playground, Experiments, Prompt Management để so sánh prompt và cấu hình <br>&emsp; + Dùng trong VS Code, Kiro hoặc web, theo chuẩn mở OpenInference và ADOT | 09/10/2026 | 09/10/2026 | <https://aws.amazon.com/vi/blogs/aws/introducing-amazon-cloudwatch-omni-ai-powered-observability-for-generative-ai-and-agentic-workloads/> |


### Kết quả đạt được tuần 5:

* Có ý tưởng ban đầu cho proposal và workshop cùng nhóm.

* Hiểu Amazon RDS: các database engine, Multi-AZ, backup và snapshot, bảo mật.

* Triển khai được database trên Amazon RDS:

  * Tạo RDS MySQL trong VPC, đặt ở private subnet qua DB subnet group
  * Kết nối từ EC2 bằng MySQL client để tạo database, bảng và thêm dữ liệu
  * Triển khai ứng dụng kết nối RDS
  * Tạo snapshot và restore ra instance mới

* Hiểu Fleet Manager cho phép kết nối RDP vào Windows instance từ console mà không cần mở cổng RDP.

* Hiểu CloudWatch Omni dùng để theo dõi trace và đánh giá chất lượng AI agent.
