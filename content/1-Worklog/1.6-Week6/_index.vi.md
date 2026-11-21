---
title: "Worklog Tuần 6"
date: 2026-10-12
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:

* Tìm hiểu Amazon Lightsail và triển khai hai ứng dụng trên Lightsail.
* Học cách bảo mật, sao lưu và nâng cấp ứng dụng trên Lightsail.
* Tìm hiểu Amazon EC2 Auto Scaling cùng Elastic Load Balancing và so sánh các giải pháp scaling.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Đọc tổng quan về Amazon Lightsail <br>- **Thực hành:** <br>&emsp; + Tạo database MySQL do Lightsail quản lý <br>&emsp; + Triển khai blog WordPress từ blueprint Ubuntu, gắn static IP và cấu hình WordPress <br>&emsp; + Triển khai cửa hàng PrestaShop từ application blueprint, gắn static IP và đặt tên miền | 12/10/2026 | 12/10/2026 | <https://000045.awsfcaj.com/> |
| 3 | - **Thực hành:** <br>&emsp; + Giới hạn rule SSH của firewall <br>&emsp; + Tạo snapshot <br>&emsp; + Chuyển WordPress sang gói lớn hơn từ snapshot <br>&emsp; + Tạo alarm CPU <br>&emsp; + Dọn tài nguyên sau lab | 13/10/2026 | 13/10/2026 | <https://000045.awsfcaj.com/> |
| 4 | - Đọc tổng quan về EC2 Auto Scaling Group và Elastic Load Balancing <br>- **Thực hành:** <br>&emsp; + Thiết lập mạng <br>&emsp; + Tạo EC2 và database instance, thêm dữ liệu vào database <br>&emsp; + Triển khai web server | 14/10/2026 | 14/10/2026 | <https://000006.awsfcaj.com/> |
| 5 | - **Thực hành:** <br>&emsp; + Tạo launch template <br>&emsp; + Tạo target group và Application Load Balancer <br>&emsp; + Kiểm tra ứng dụng qua load balancer <br>&emsp; + Tạo Auto Scaling group <br>&emsp; + Chuẩn bị metric cho predictive scaling | 15/10/2026 | 15/10/2026 | <https://000006.awsfcaj.com/> |
| 6 | - **Thực hành:** <br>&emsp; + Kiểm tra manual, scheduled, dynamic và predictive scaling <br>&emsp; + Dọn tài nguyên sau lab | 16/10/2026 | 16/10/2026 | <https://000006.awsfcaj.com/> |


### Kết quả đạt được tuần 6:

* Học Amazon Lightsail: một gói gồm máy chủ ảo, ổ SSD, băng thông và static IP với giá cố định theo tháng, nên không phải thiết kế VPC, security group hay EBS từ đầu.

* Triển khai hai ứng dụng trên Lightsail ở Region Singapore:

  * Blog WordPress từ blueprint Ubuntu 24.04 với database MySQL do Lightsail quản lý
  * Cửa hàng PrestaShop từ application blueprint dựng sẵn
  * Static IP cho từng instance để địa chỉ không đổi

* Bảo mật và vận hành các ứng dụng:

  * Giới hạn rule SSH để chỉ Lightsail browser SSH client kết nối được, thay vì mọi địa chỉ trên internet
  * Tạo snapshot và dùng nó để chuyển WordPress sang gói lớn hơn, vì instance Lightsail không đổi gói tại chỗ được
  * Chuyển static IP sang instance mới, blog giữ nguyên nội dung vì dữ liệu nằm trong managed database
  * Tạo alarm CPU

* Hiểu EC2 Auto Scaling group mang lại gì: tự thay instance lỗi, dung lượng đi theo nhu cầu, Elastic Load Balancing chia tải, và chi phí giảm khi ít truy cập.

* Triển khai ứng dụng FCAJ Management trên EC2 với database và web server, rồi dựng phần scaling xung quanh:

  * Launch template chứa AMI, loại instance, security group và user data
  * Target group và Application Load Balancer đặt trước các instance
  * Auto Scaling group trải trên 3 public subnet với desired capacity 1, tối thiểu 1, tối đa 3, health check từ load balancer, CloudWatch group metrics và thông báo SNS

* So sánh bốn giải pháp scaling:

  * Manual scaling: đổi desired capacity bằng tay khi cần nhất thời
  * Scheduled scaling: đổi dung lượng theo lịch khi lưu lượng đoán trước được
  * Dynamic scaling: phản ứng theo metric thời gian thực như CPU hoặc số request
  * Predictive scaling: dùng dữ liệu lịch sử để lên kế hoạch dung lượng cho 48 đến 72 giờ tới, cần chuẩn bị metric trước
