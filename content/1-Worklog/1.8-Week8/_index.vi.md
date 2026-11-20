---
title: "Worklog Tuần 8"
date: 2026-10-26
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---


### Mục tiêu tuần 8:

* Tìm hiểu AWS CLI và dùng nó để quản lý các dịch vụ AWS bằng dòng lệnh.
* Tạo tài nguyên bằng CLI: S3, SNS, IAM, mạng và EC2.
* Tiếp tục làm workshop và kiểm tra kiến trúc cũng như thông tin trong proposal.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Đọc tổng quan về AWS CLI: profile, cấu hình, định dạng output <br>- **Thực hành:** <br>&emsp; + Cài AWS CLI và cấu hình profile bằng aws configure <br>&emsp; + Kiểm tra tài nguyên qua CLI | 26/10/2026 | 26/10/2026 | <https://000011.awsfcaj.com/> |
| 3 | - **Thực hành:** <br>&emsp; + Dùng CLI với Amazon S3 <br>&emsp; + Dùng CLI với Amazon SNS | 27/10/2026 | 27/10/2026 | <https://000011.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Tạo IAM group, user và access key bằng CLI <br>&emsp; + Tạo VPC, subnet, internet gateway và route table bằng CLI | 28/10/2026 | 28/10/2026 | <https://000011.awsfcaj.com/> |
| 5 | - **Thực hành:** <br>&emsp; + Tạo key pair và security group, rồi tạo EC2 và kết nối SSH bằng CLI <br>&emsp; + Terminate instance và dọn tài nguyên sau lab | 29/10/2026 | 29/10/2026 | <https://000011.awsfcaj.com/> |
| 6 | - Tiếp tục làm workshop <br>- Kiểm tra kiến trúc và các thông tin trong proposal | 30/10/2026 | 30/10/2026 | |


### Kết quả đạt được tuần 8:

* Hiểu AWS CLI là gì: công cụ mã nguồn mở truy cập trực tiếp API công khai của các dịch vụ AWS, nên việc làm được trên console cũng làm được bằng dòng lệnh hoặc script.

* Cài đặt và dùng CLI:

  * Cấu hình profile bằng aws configure (access key, secret access key, Region và định dạng output), dùng named profile với tùy chọn --profile
  * Chọn giữa các định dạng output json, yaml, text, table tùy người hay công cụ đọc kết quả
  * Kiểm tra EC2 và S3 bucket từ CLI, thử CLI auto-prompt để hỗ trợ cú pháp

* Tạo và quản lý tài nguyên bằng CLI:

  * Làm việc với Amazon S3 và Amazon SNS từ dòng lệnh
  * Tạo IAM group, user và access key
  * Tạo VPC, subnet, internet gateway và route table
  * Tạo key pair và security group, chạy một EC2 trong mạng vừa dựng, kết nối SSH và terminate

* Biết cách bảo vệ thông tin xác thực: không chia sẻ access key, ưu tiên IAM role khi có thể, xóa key pair và access key không còn dùng.

* Tiếp tục xây dựng workshop và kiểm tra kiến trúc cũng như thông tin trong proposal.
