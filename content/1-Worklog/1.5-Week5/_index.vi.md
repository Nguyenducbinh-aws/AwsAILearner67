---
title: "Worklog Tuần 5"
date: 2026-10-05
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---


### Mục tiêu tuần 5:

* Cùng nhóm lên ý tưởng cho proposal và workshop.
* Hiểu cơ bản về Amazon RDS và dựng một database.
* Tìm hiểu cách vào Windows instance bằng Fleet Manager và cách theo dõi AI agent bằng CloudWatch Omni.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Lên ý tưởng cho proposal và workshop cùng với nhóm | 05/10/2026 | 05/10/2026 | |
| 3 | - Đọc tài liệu về Amazon RDS <br>&emsp; + Database engine được hỗ trợ <br>&emsp; + Multi-AZ, Read Replica <br>&emsp; + Backup và snapshot <br>&emsp; + Bảo mật: mã hóa, VPC, IAM | 06/10/2026 | 06/10/2026 | <https://000005.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Tạo VPC, Security Group và DB subnet group <br>&emsp; + Tạo RDS MySQL <br>&emsp; + Tạo EC2, kết nối RDS bằng MySQL client <br>&emsp; + Tạo database, tạo bảng và thêm dữ liệu <br>&emsp; + Triển khai ứng dụng kết nối RDS <br>&emsp; + Tạo snapshot và restore RDS <br>&emsp; + Dọn tài nguyên sau lab | 07/10/2026 | 07/10/2026 | <https://000005.awsfcaj.com/> |
| 5 | - Đọc blog về AWS Systems Manager Fleet Manager | 08/10/2026 | 08/10/2026 | <https://aws.amazon.com/vi/blogs/mt/console-based-access-to-windows-instances-using-aws-systems-manager-fleet-manager/> |
| 6 | - Đọc blog về Amazon CloudWatch Omni | 09/10/2026 | 09/10/2026 | <https://aws.amazon.com/vi/blogs/aws/introducing-amazon-cloudwatch-omni-ai-powered-observability-for-generative-ai-and-agentic-workloads/> |


### Kết quả đạt được tuần 5:

* Cùng nhóm có những ý tưởng đầu tiên cho proposal và workshop.

* Học Amazon RDS: các database engine, Multi-AZ, backup và snapshot, bảo mật.

* Triển khai database trên Amazon RDS:

  * Tạo RDS MySQL trong VPC, đặt ở private subnet qua DB subnet group
  * Kết nối từ EC2 bằng MySQL client để tạo database, bảng và thêm dữ liệu
  * Triển khai một ứng dụng kết nối RDS
  * Tạo snapshot và restore ra instance mới

* Học AWS Systems Manager Fleet Manager qua blog về truy cập Windows instance bằng console:

  * Mở RDP vào Windows instance ngay trên trình duyệt qua Session Manager
  * Không cần mở cổng RDP, dùng public IP hay chạy bastion host, nên bề mặt tấn công nhỏ hơn
  * Dùng được key pair EC2 hoặc credential Windows, xem tối đa 4 instance cùng lúc
  * Cần SSM Agent, instance profile có quyền Systems Manager và đường tới SSM endpoint

* Học Amazon CloudWatch Omni qua blog về theo dõi AI agent:

  * Agent không phải lần nào cũng chạy giống nhau, nên cần trace ghi từng lần gọi LLM và tool để tìm chỗ nó đi sai
  * 17 evaluator dựng sẵn (coherence, helpfulness, faithfulness và các loại khác) chấm chất lượng câu trả lời, vì latency và tỷ lệ lỗi không cho biết câu trả lời có tốt không
  * Playground, Experiments và Prompt Management dùng để so sánh prompt, cấu hình và quay lại bản cũ
  * Dùng được trong VS Code, Kiro hoặc trên web, theo chuẩn mở OpenInference và ADOT, hỗ trợ các framework như LangChain, CrewAI và Strands
