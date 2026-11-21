---
title: "Worklog Tuần 11"
date: 2026-11-16
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---


### Mục tiêu tuần 11:

* Triển khai workshop Cost Guard và kiểm thử.
* Cập nhật workshop, chạy demo và viết báo cáo.
* Đọc về cách đưa AWS Transform vào GitHub Copilot qua MCP.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Triển khai workshop Cost Guard | 16/11/2026 | 16/11/2026 | |
| 3 | - Chạy kiểm thử cho workshop  | 17/11/2026 | 17/11/2026 | |
| 4 | - Cập nhật workshop sau kiểm thử <br>- Chạy demo | 18/11/2026 | 18/11/2026 | |
| 5 | - Viết báo cáo <br>- Dọn tài nguyên sau khi demo xong | 19/11/2026 | 19/11/2026 | |
| 6 | - Đọc blog về đưa AWS Transform vào GitHub Copilot qua MCP | 20/11/2026 | 20/11/2026 | <https://aws.amazon.com/vi/blogs/dotnet/bring-aws-transform-into-github-copilot-with-mcp/> |


### Kết quả đạt được tuần 11:

* Triển khai workshop Cost Guard và chạy kiểm thử.

* Cập nhật workshop sau kiểm thử và chạy demo.

* Viết báo cáo.

* Học từ blog cách đưa AWS Transform vào GitHub Copilot qua MCP:

  * AWS Transform là dịch vụ AI dạng agent, phân tích và chuyển đổi cả ứng dụng, hạ tầng và mã nguồn, còn AWS Transform MCP server cho phép gọi nó bằng ngôn ngữ tự nhiên từ chế độ agent của Copilot mà không cần chuyển sang console
  * Ví dụ trong blog chuyển một ứng dụng .NET Azure Functions (trigger HTTP, Queue và Blob) sang AWS Lambda bằng Lambda Annotations Framework và sinh ra SAM template
  * AWS Transform dừng lại chờ duyệt trước các thao tác có tính phá hủy như push, merge, delete, và báo các thành phần nguồn không có tương đương trực tiếp trên AWS thay vì sinh ra code lỗi
  * Một chuyển đổi đã xong có thể lưu thành Agent Skill (file SKILL.md) và dùng lại trong các công cụ như GitHub Copilot, Kiro và Claude Code
  * Dùng quyền tối thiểu cho IAM role của việc chuyển đổi, và chạy sam delete để xóa tài nguyên khi xong
