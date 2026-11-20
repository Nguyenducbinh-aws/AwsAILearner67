---
title: "Worklog Tuần 10"
date: 2026-11-09
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---


### Mục tiêu tuần 10:

* Tìm hiểu AWS WAF và dùng nó để bảo vệ ứng dụng web.
* Tìm hiểu Amazon GuardDuty và thử phát hiện, xử lý mối đe dọa.
* Xem cách dựng phản ứng tự động bằng Amazon EventBridge và AWS Lambda.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Đọc phần giới thiệu AWS WAF <br>- **Thực hành:** <br>&emsp; + Tạo S3 bucket và triển khai ứng dụng web mẫu <br>&emsp; + Tạo web ACL với managed rule <br>&emsp; + Tạo custom rule và advanced custom rule <br>&emsp; + Kiểm tra các rule mới và bật logging <br>&emsp; + Dọn tài nguyên sau lab | 09/11/2026 | 13/11/2026 | <https://000026.awsfcaj.com/> |
| 3 | - Đọc tổng quan Amazon GuardDuty: nguồn dữ liệu và finding <br>- Xem và làm thử lab: <br>&emsp; + Chuẩn bị môi trường bằng CloudFormation template | 10/11/2026 | 10/11/2026 | <https://000098.awsfcaj.com/> |
| 4 | - Xem và làm thử lab: <br>&emsp; + Kịch bản 1: EC2 bị xâm nhập, phát hiện bằng GuardDuty và xử lý bằng EventBridge và Lambda | 11/11/2026 | 11/11/2026 | <https://000098.awsfcaj.com/> |
| 5 | - Xem và làm thử lab: <br>&emsp; + Kịch bản 2: IAM credential bị lộ, xử lý thủ công | 12/11/2026 | 12/11/2026 | <https://000098.awsfcaj.com/> |
| 6 | - Xem và làm thử lab: <br>&emsp; + Kịch bản 3: IAM role credential bị đánh cắp, xử lý bằng Lambda <br>&emsp; + Xem lại các finding và dọn tài nguyên sau lab | 13/11/2026 | 13/11/2026 | <https://000098.awsfcaj.com/> |


### Kết quả đạt được tuần 10:

* Hiểu AWS WAF làm gì: đây là tường lửa ứng dụng web, bảo vệ ứng dụng web và API khỏi các tấn công như SQL injection và cross-site scripting, cho phép viết custom rule để chặn hoặc cho phép request HTTP trước khi đến ứng dụng.

* Thực hành AWS WAF:

  * Triển khai ứng dụng web mẫu và tạo web ACL với managed rule
  * Tạo custom rule chặn request có một header nhất định và một advanced custom rule
  * Gửi request thử và thấy các request bị chặn trong dashboard và sampled requests
  * Bật logging cho web ACL

* Hiểu cách Amazon GuardDuty hoạt động:

  * Dịch vụ được quản lý hoàn toàn, theo từng Region, phân tích VPC Flow Logs, CloudTrail logs và DNS logs mà không ảnh hưởng tài nguyên đang theo dõi
  * Kết quả hiện dưới dạng finding có tên loại dễ đọc, và GuardDuty gửi thông báo khi finding thay đổi
  * Chi phí tính theo số sự kiện CloudTrail và dung lượng flow log, DNS log, mỗi tài khoản có 30 ngày dùng thử ở mỗi Region

* Xem và làm thử ba kịch bản mối đe dọa trong lab:

  * EC2 bị xâm nhập (finding UnauthorizedAccess:EC2/MaliciousIPCaller.Custom), GuardDuty phát hiện và xử lý tự động bằng EventBridge rule kết hợp Lambda
  * IAM credential bị lộ (finding Recon:IAMUser/MaliciousIPCaller.Custom và UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom), tìm ra ai đang gọi API và xử lý thủ công
  * IAM role credential bị đánh cắp (finding UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration), xử lý bằng Lambda

* Thấy được toàn bộ luồng phát hiện, thông báo và phản ứng trong bảo mật cloud, và dọn tài nguyên sau mỗi lab.
