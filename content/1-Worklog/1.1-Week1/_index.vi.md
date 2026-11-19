---
title: "Worklog Tuần 1"
date: 2026-09-07
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---


### Mục tiêu tuần 1:

* Làm quen với các thành viên và cách làm việc của chương trình First Cloud AI Journey (FCAJ), nắm nội quy thực tập.
* Hiểu tổng quan về AWS và các nhóm dịch vụ chính (Compute, Storage, Networking, Database, …).
* Tạo và bảo vệ tài khoản AWS ngay từ đầu: bật MFA, thiết lập cảnh báo chi phí bằng AWS Budgets.
* Biết cách tìm hỗ trợ qua AWS Support và quản lý truy cập bằng IAM.
* Làm quen với hai cách thao tác: AWS Management Console và AWS CLI.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Làm quen với các thành viên FCAJ <br> - Đọc và ghi chú nội quy, quy định thực tập <br> - Tìm hiểu lộ trình Cloud Journey <br>&emsp; + Cấu trúc 5 module: Explore, Migrate, Optimize, Modernize, Container <br>&emsp; + Cách mỗi module chia thành các workshop có số hiệu riêng <br> - Chuẩn bị môi trường: trình duyệt, trình soạn thảo, ghi chú cá nhân, khởi tạo repo Worklog từ fcj-workshop-template | 07/09/2026 | 07/09/2026 | <https://cloudjourney.awsfcaj.com/> |
| 3 | - Tìm hiểu AWS và các loại dịch vụ <br>&emsp; + Compute (EC2, Lightsail, container) <br>&emsp; + Storage (S3, EBS) <br>&emsp; + Networking (VPC, Route 53, CloudFront) <br>&emsp; + Database (RDS, DynamoDB, ElastiCache) <br>&emsp; + Quản trị & giám sát (IAM, CloudWatch, Budgets) <br> - Đọc danh sách workshop của Module 1 để lên kế hoạch học 3 tuần | 08/09/2026 | 08/09/2026 | <https://cloudjourney.awsfcaj.com/1-explore/> |
| 4 | - Tạo tài khoản AWS <br>&emsp; + Quy trình đăng ký, xác minh thanh toán/điện thoại, chọn gói Support <br>&emsp; + Bảo vệ root user: bật MFA, hạn chế dùng root cho công việc hằng ngày <br> - Tìm hiểu AWS Budgets <br>&emsp; + Các loại budget (cost, usage), ngưỡng cảnh báo, email nhận thông báo <br> - **Thực hành:** tạo tài khoản, bật MFA cho root, tạo một cost budget có cảnh báo qua email | 09/09/2026 | 09/09/2026 | <https://000001.awsfcaj.com> <br> <https://000007.awsfcaj.com> |
| 5 | - Tìm hiểu AWS Support <br>&emsp; + Các gói Support và phạm vi hỗ trợ <br>&emsp; + Cách tạo và theo dõi support case <br> - Tìm hiểu AWS IAM <br>&emsp; + User, Group, Policy, Role <br>&emsp; + Nguyên tắc least privilege <br> - **Thực hành:** xem trang Support Center và các loại case; tạo IAM group gắn policy, tạo IAM user admin để dùng thay root, đăng nhập lại bằng IAM user | 10/09/2026 | 10/09/2026 | <https://000009.awsfcaj.com> <br> <https://000002.awsfcaj.com> |
| 6 | - Tìm hiểu AWS CLI <br>&emsp; + So sánh thao tác qua Console và CLI <br>&emsp; + Access key, profile, region mặc định, định dạng output <br> - **Thực hành:** cài đặt và cấu hình AWS CLI (aws configure), chạy một số lệnh truy vấn cơ bản; ôn lại toàn bộ kiến thức tuần 1, kiểm tra Billing/Budgets và cập nhật Worklog | 11/09/2026 | 11/09/2026 | <https://000011.awsfcaj.com> |


### Kết quả đạt được tuần 1:

* Nắm được nội quy, cách làm việc và lộ trình 5 module của FCAJ.
* Phân biệt được các nhóm dịch vụ AWS chính và vai trò của từng nhóm.
* Đã tạo tài khoản AWS và áp dụng các bước bảo vệ cơ bản (MFA cho root, dùng IAM user thay root).
  * Ngày tạo tài khoản / region chính sử dụng: [TỰ ĐIỀN: ...]
* Đã thiết lập AWS Budgets để cảnh báo chi phí.
  * Ngưỡng budget đã đặt: [TỰ ĐIỀN: số tiền USD]
* Hiểu các khái niệm cốt lõi của IAM (user, group, policy, role) và nguyên tắc least privilege.
* Cài đặt và cấu hình được AWS CLI, thực hiện được các lệnh truy vấn cơ bản.
  * Phiên bản AWS CLI / hệ điều hành: [TỰ ĐIỀN: ...]
* Minh chứng (ảnh chụp màn hình): [TỰ ĐIỀN: đường dẫn ảnh trong /static/images/...]
