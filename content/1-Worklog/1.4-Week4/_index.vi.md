---
title: "Worklog Tuần 4"
date: 2026-09-28
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---


### Mục tiêu tuần 4:

* Làm quen với AWS CloudShell và AWS CLI.
* Hiểu cơ bản về Amazon S3 và Amazon CloudFront.
* Thực hành lưu trữ, host static website và quản lý object trên S3.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu AWS CloudShell <br> - **Thực hành:** <br>&emsp; + Tạo môi trường CloudShell <br>&emsp; + Dùng lệnh cơ bản: pwd, ls -la, mkdir, touch, vi, cat <br>&emsp; + Dùng AWS CLI: kiểm tra phiên bản, liệt kê S3 bucket và EC2 instance <br>&emsp; + Tải file lên và tải file xuống CloudShell <br>&emsp; + Xóa môi trường CloudShell | 28/09/2026 | 28/09/2026 | <https://000049.awsfcaj.com/> |
| 3 | - Tìm hiểu Amazon S3 <br>&emsp; + Bucket và object <br>&emsp; + Storage class <br>&emsp; + Versioning, replication <br> - **Thực hành:** <br>&emsp; + Tạo S3 bucket <br>&emsp; + Tải source code website lên bucket | 29/09/2026 | 02/10/2026 | <https://000057.awsfcaj.com/> |
| 4 | - **Thực hành:** <br>&emsp; + Bật static website hosting, đặt index.html <br>&emsp; + Tắt Block public access, public object bằng ACL <br>&emsp; + Mở website bằng website endpoint và xem thời gian tải trang | 30/09/2026 | 30/09/2026 | <https://000057.awsfcaj.com/> |
| 5 | - Tìm hiểu Amazon CloudFront <br> - **Thực hành:** <br>&emsp; + Bật lại Block all public access cho bucket <br>&emsp; + Tạo CloudFront distribution với S3 origin, thêm bucket policy <br>&emsp; + Mở website bằng CloudFront và so sánh thời gian tải trang | 01/10/2026 | 01/10/2026 | <https://000057.awsfcaj.com/> |
| 6 | - Tìm hiểu quản lý object trong S3 <br>&emsp; + Versioning <br>&emsp; + Move object <br>&emsp; + Replication giữa các Region <br> - **Thực hành:** <br>&emsp; + Bật versioning, sửa index.html và tải lại để xem các phiên bản <br>&emsp; + Tạo bucket mới và move object sang bucket đó <br>&emsp; + Tạo replication rule sang bucket ở Region N. Virginia <br>&emsp; + Dọn tài nguyên sau lab | 02/10/2026 | 02/10/2026 | <https://000057.awsfcaj.com/> |


### Kết quả đạt được tuần 4:

* Dùng AWS CloudShell để chạy lệnh ngay trên trình duyệt mà không cần cài đặt:

  * Dùng lệnh Linux cơ bản để tạo thư mục, tạo và sửa file, xem quyền của file
  * Dùng AWS CLI để kiểm tra phiên bản, liệt kê S3 bucket và EC2 instance trong tài khoản
  * Tải file lên và tải file xuống giữa máy tính và CloudShell

* Hiểu Amazon S3 (bucket, object, storage class) và tạo được bucket để lưu source code website.

* Host được static website trên S3:

  * Bật static website hosting và mở website bằng website endpoint (HTTP)
  * Tắt Block public access và public object bằng ACL để website truy cập được

* Dùng CloudFront để phân phối website mà không cần public bucket:

  * Bật lại Block all public access, cho CloudFront đọc bucket bằng bucket policy
  * Thấy được thời gian tải trang giảm và biết được PoP phục vụ nội dung

* Quản lý object trong S3:

  * Dùng versioning để giữ nhiều phiên bản của object và khôi phục nội dung cũ
  * Move object sang bucket khác
  * Dùng replication để tự động sao chép object sang bucket ở Region khác, phục vụ dự phòng
