---
title: "Tổng quan Workshop"
date: 2026-11-18
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Tổng quan Workshop

**"Cost Guard: Bảo vệ credit máy lab Windows trên AWS bằng ba lớp serverless"**

### Vấn đề cần giải quyết

Máy Windows EC2 tính phí theo giờ. Một máy t3.medium chạy 24/7 tốn khoảng $46.22 mỗi tháng, vượt quá ngân sách credit $20 của dự án. AWS Budgets có cảnh báo chi phí, nhưng dữ liệu billing chỉ cập nhật vài lần mỗi ngày, nên không kịp dừng một máy bị quên.

Workshop này xây dựng Cost Guard, hệ thống tự động dừng máy chỉ bằng các dịch vụ serverless thuần AWS.

### Những gì bạn sẽ xây dựng

Sau khi hoàn thành workshop, bạn sẽ có:

- Một **EC2 Windows** (t3.medium) với ổ gp3 30 GB làm máy lab được bảo vệ
- **Lớp 1 – Schedule Guard:** EventBridge Scheduler và một hàm Lambda dừng máy mỗi ngày lúc 22:00
- **Lớp 2 – Cost Meter:** một hàm Lambda, được EventBridge gọi mỗi 5 phút, ước tính chi phí và dừng máy khi đạt ngưỡng
- **Lớp 3 – Budget Backstop:** một AWS Budget thông báo qua SNS ở mức 80% và 100% của $20 và kích hoạt Lambda để dừng máy
- **Nền tảng dùng chung:** Parameter Store cho cấu hình, bảng DynamoDB cho bản ghi phiên, hai SNS topic (cảnh báo và lỗi) và IAM role quyền tối thiểu cho từng hàm
- **Giám sát:** CloudWatch Logs và CloudWatch Alarms cho lỗi của cả ba hàm

### Kiến trúc

![Sơ đồ kiến trúc](../../../images/2-Proposal/architecture.png)

Các số trong sơ đồ theo đúng thứ tự luồng:

1. Người dùng lab bật EC2 Windows thủ công.
2. Lớp 1: EventBridge Scheduler gọi ScheduleHandler khi hết giờ lab.
3. Lớp 2: một lịch EventBridge Scheduler gọi CostMeter mỗi 5 phút.
4. Lớp 3: cảnh báo của AWS Budgets đi qua SNS tới BudgetBackstop.
5. Các hàm Lambda đọc đơn giá và ngưỡng, đọc hoặc ghi bản ghi phiên.
6. Một hàm Lambda dừng EC2 (hết giờ lab, chi phí vượt ngưỡng, hoặc budget chạm hạn mức).
7. SNS gửi email cảnh báo cho người dùng.
8. CloudWatch Alarms theo dõi lỗi Lambda và gửi qua SNS cho người dùng.

### Business logic

- **Bật máy:** Người dùng bật instance thủ công. Không lớp nào tự bật máy.
- **Lớp 1:** Mỗi ngày lúc 22:00 (Asia/Ho_Chi_Minh), nếu instance đang chạy thì dừng. Nếu đã dừng thì không làm gì.
- **Lớp 2:** Cứ 5 phút một lần, nếu instance đang chạy:
  - ước tính chi phí phiên hiện tại bằng số giờ chạy × đơn giá theo giờ
  - cộng với chi phí ước tính của các phiên trước trong cùng tháng dương lịch
  - nếu tổng đạt ngưỡng (khoảng $14, tức 70% của $20) thì dừng instance, ghi sự kiện và gửi cảnh báo
  - nếu chưa đạt thì chỉ ghi lại ước tính
- **Bật lại sau khi Lớp 2 dừng máy:** Tổng chi phí trong tháng vẫn trên ngưỡng, nên lần chạy kế tiếp lại dừng máy. Máy bị khóa đến tháng dương lịch tiếp theo.
- **Lớp 3:** AWS Budgets theo dõi chi tiêu thực của tài khoản và thông báo ở mức 80% và 100% của $20. Mỗi thông báo kích hoạt BudgetBackstop, hàm này dừng instance và gửi email xác nhận. Dữ liệu billing bị trễ nên lớp này chỉ là lưới an toàn cuối cùng.
- **Các lớp không xung đột:** Lớp nào cũng dừng được máy, không lớp nào bật được máy, và dừng một máy đã dừng thì không có tác dụng.
- **Khi một lớp lỗi:** Hai lớp còn lại vẫn hoạt động. Lỗi của hàm tạo CloudWatch alarm và gửi email.
- **Phạm vi chi phí:** Ngưỡng của Lớp 2 chỉ tính chi phí chạy EC2. Ổ EBS ($2.40 mỗi tháng) tính riêng.
- **Cấu hình:** Đơn giá, ngưỡng và hạn mức ngân sách là các giá trị trong Parameter Store. Thay đổi có hiệu lực từ lần chạy kế tiếp, không cần deploy lại.

### Các quyết định thiết kế chính

- **Chỉ dừng máy, không bao giờ tự bật:** Bật một máy không ai dùng là lãng phí credit.
- **Ba lớp thay vì một:** Lịch theo giờ thì đáng tin cậy, bộ đo chi phí thì nhanh, còn Budgets dựa trên billing thực. Kết hợp lại, một lỗi đơn lẻ không làm máy mất bảo vệ.
- **Bộ đo chi phí dựa trên thời gian chạy:** Phản ứng trong vài phút, không chờ dữ liệu billing.
- **Cấu hình trong Parameter Store:** Hệ thống dùng lại được cho loại instance hoặc tài khoản khác mà không sửa code.
- **Mỗi hàm một IAM role:** Mỗi role chỉ có quyền dừng đúng instance đích, nên lỗi của hàm không thể dừng máy khác.
- **Khóa đến tháng sau khi dừng vì vượt ngưỡng:** Đơn giản và an toàn cho credit. Cái giá phải trả là người dùng không thể tiếp tục làm việc trong tháng đó.

### Dịch vụ và các Pillar (Well-Architected)

- **Cost Optimization:** EventBridge Scheduler, Cost Meter và AWS Budgets chặn khoản chi không ai cần.
- **Reliability:** Ba lớp độc lập; nếu một lớp lỗi, các lớp còn lại vẫn bảo vệ máy.
- **Security:** Mỗi hàm một IAM role quyền tối thiểu, quyền dừng máy chỉ giới hạn cho instance đích.
- **Operational Excellence:** CloudWatch Logs, CloudWatch Alarms, thông báo SNS và bản ghi phiên trong DynamoDB giúp nhìn thấy hành vi của hệ thống.
