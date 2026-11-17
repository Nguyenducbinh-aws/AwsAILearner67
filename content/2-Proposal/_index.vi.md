---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cost Guard for Windows Labs on AWS
## Giải Pháp Bảo Vệ Credit Ba Lớp Dùng Serverless

### 1. Executive Summary

Cost Guard là hệ thống tự động hóa serverless dành cho sinh viên và đội nhỏ chạy máy lab Windows trên tài khoản AWS Free Plan. Vì máy Windows EC2 tính phí theo giờ, chỉ cần quên tắt vài ngày là có thể hết toàn bộ credit. Dự án giải quyết vấn đề này bằng ba lớp bảo vệ tự động: lịch dừng theo giờ, bộ đo chi phí gần thời gian thực, và một lớp dự phòng dùng AWS Budgets — tất cả đều dùng dịch vụ nằm trong AWS Free Tier.

### 2. Vấn đề cần giải quyết

#### Vấn đề là gì?

Credit AWS Free Plan có giới hạn. Một máy Windows EC2 chạy không có người giám sát có thể tiêu tốn phần lớn ngân sách chỉ trong vài ngày. AWS Budgets có cảnh báo chi phí nhưng dữ liệu billing chỉ cập nhật vài lần mỗi ngày, nên không đủ nhanh để làm cơ chế bảo vệ thời gian thực.

#### Giải pháp

Cost Guard dùng ba lớp bảo vệ bổ sung cho nhau:

- **Lớp 1 — Lịch dừng/bật:** EventBridge Scheduler tự dừng máy ngoài giờ lab. Lớp này không phụ thuộc vào dữ liệu billing và đáng tin cậy nhất.
- **Lớp 2 — Tự đo chi phí (đóng góp riêng):** Lambda chạy định kỳ để ước tính chi phí phiên hiện tại dựa trên thời gian chạy và đơn giá đã cấu hình. Nếu vượt ngưỡng nội bộ, máy bị dừng và cảnh báo được gửi qua SNS.
- **Lớp 3 — Budget dự phòng:** AWS Budgets theo dõi chi tiêu thực của tài khoản. Khi tiến gần giới hạn, nó kích hoạt Lambda qua SNS để dừng máy như lưới an toàn cuối cùng.

### Benefits and Return on Investment

- Ngăn mất credit ngoài ý muốn do quên tắt máy hoặc máy chạy không cần thiết.
- Lớp 2 cung cấp thông tin chi phí trong vài phút sau khi máy khởi động, không cần chờ billing cập nhật.
- Các dịch vụ hỗ trợ (Lambda, EventBridge, DynamoDB, SNS, Parameter Store) đều nằm trong Free Tier trong điều kiện sử dụng bình thường.
- Đơn giá và ngưỡng được lưu trong Parameter Store, không hardcode — tái sử dụng được cho các loại instance hoặc tài khoản khác.

### 3. Kiến trúc giải pháp

Hệ thống chạy trong một AWS Region, dùng hoàn toàn serverless hoặc dịch vụ managed. EventBridge điều phối lịch và gọi hàm định kỳ. Lambda xử lý logic dừng/bật và tính chi phí. DynamoDB ghi lại hoạt động từng phiên. SNS gửi thông báo. Parameter Store lưu cấu hình. IAM role giới hạn quyền tối thiểu cho từng hàm.

![Sơ đồ kiến trúc](/images/2-Proposal/architecture.png)

### Các dịch vụ AWS sử dụng

- **Amazon EC2 (Windows)**: Máy lab được bảo vệ; bị dừng tự động bởi cả ba lớp.
- **Amazon EventBridge**: Kích hoạt lịch dừng/bật và gọi bộ đo chi phí định kỳ.
- **AWS Lambda**: Thực thi logic bảo vệ của từng lớp.
- **Amazon DynamoDB**: Lưu log hoạt động và ước tính chi phí từng phiên.
- **Amazon SNS**: Gửi email cảnh báo khi vượt ngưỡng hoặc khi có lỗi.
- **AWS Budgets**: Theo dõi chi tiêu thực và kích hoạt lớp bảo vệ cuối cùng.
- **AWS Systems Manager Parameter Store**: Lưu đơn giá/giờ và ngưỡng chi phí, không hardcode trong Lambda.
- **Amazon CloudWatch**: Thu thập log Lambda và cảnh báo khi hàm lỗi.
- **AWS IAM**: Cung cấp role quyền tối thiểu cho từng Lambda.
- **AWS CloudFormation**: Đóng gói toàn bộ stack để triển khai lại và dọn dẹp sạch sẽ (tùy chọn).

### Component Design

- **Scheduler Layer**: EventBridge Scheduler → Lambda `ScheduleHandler` → dừng hoặc bật EC2 theo lịch cố định.
- **Cost Meter**: EventBridge rule định kỳ → Lambda `CostMeter` → đọc đơn giá và ngưỡng từ Parameter Store → tính chi phí ước tính từ dữ liệu phiên trong DynamoDB → nếu vượt ngưỡng: dừng máy + gửi SNS.
- **Budget Backstop**: AWS Budgets alarm → SNS topic → Lambda `BudgetBackstop` → dừng máy → gửi email xác nhận.
- **Thông báo**: Một SNS topic cho cảnh báo chi phí đến người dùng; một SNS topic cho lỗi vận hành.
- **Dữ liệu**: Bảng DynamoDB ghi sự kiện bật/tắt máy và ước tính chi phí định kỳ từng phiên.
- **Bảo mật**: Mỗi Lambda có IAM role riêng, quyền giới hạn đúng tài nguyên nó cần dùng.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

- **Giai đoạn 1 — Nền tảng:** Dựng hạ tầng cơ sở — IAM role, SNS topic, bảng DynamoDB, tham số Parameter Store và AWS Budget. Chọn loại instance và region; dùng AWS Pricing Calculator lấy đơn giá/giờ thực tế.
- **Giai đoạn 2 — Lớp 1:** Khởi chạy Windows EC2, deploy `ScheduleHandler`, cấu hình EventBridge Scheduler, kiểm tra dừng/bật.
- **Giai đoạn 3 — Lớp 2:** Deploy `CostMeter`, cấu hình EventBridge rule định kỳ, thử nghiệm logic ngưỡng với dữ liệu giả lập.
- **Giai đoạn 4 — Lớp 3 + Quan sát:** Cấu hình Budgets alarm, deploy `BudgetBackstop`, thêm CloudWatch Alarm, kiểm tra end-to-end cả ba lớp.
- **Giai đoạn 5 — Demo và tài liệu:** Quay video demo khi tài nguyên còn sống, dọn dẹp, chụp Cost Explorer, viết nội dung workshop.

**Yêu cầu kỹ thuật**

- Tài khoản AWS còn credit hoặc đang trong Free Plan.
- AWS CLI cấu hình với IAM user quyền tối thiểu, không dùng root key.
- Lambda runtime Python với `boto3` có sẵn.
- Hiểu cơ bản về Lambda, EventBridge, SNS, DynamoDB, Parameter Store và IAM.
- draw.io để vẽ sơ đồ kiến trúc.
- Hugo để xem trước site trước khi push.

### 5. Lịch trình & Cột mốc

**Kế hoạch 7 ngày**

| Ngày | Việc cần làm | Sản phẩm |
|------|-------------|----------|
| 1 | Chốt instance và region; chạy Pricing Calculator; tạo IAM, SNS, DynamoDB, Parameter Store, Budget | Hạ tầng nền sẵn sàng |
| 2 | Khởi chạy Windows EC2; deploy Lớp 1; test lịch dừng/bật | Lớp 1 hoạt động |
| 3 | Deploy CostMeter; test ngưỡng và cảnh báo SNS | Lớp 2 hoạt động |
| 4 | Deploy BudgetBackstop; thêm CloudWatch Alarm; test end-to-end | Cả 3 lớp hoạt động |
| 5 | Quay video demo; dọn tài nguyên; chụp Cost Explorer | Video + ảnh dọn dẹp |
| 6 | Vẽ sơ đồ kiến trúc; viết workshop 5.1–5.8 | Bản nháp workshop |
| 7 | Viết Proposal; dịch sang tiếng Anh; xem trước Hugo; push và kiểm tra link | Site hoàn chỉnh |

### 6. Dự toán chi phí

Chi phí ước tính dựa trên AWS Pricing Calculator cho region đã chọn, giá on-demand. Máy Windows là phần chi phí chủ yếu; các dịch vụ còn lại nằm trong Free Tier trong điều kiện sử dụng bình thường.

[Ước tính AWS Pricing Calculator](#) *(cập nhật link sau khi export)*

File PDF ước tính đặt tại: [`/attachments/budget_estimation.pdf`](/attachments/budget_estimation.pdf)

### Infrastructure Costs

| Dịch vụ | Giả định | Chi phí ước tính |
|---------|---------|-----------------|
| EC2 Windows (instance nhỏ) | Giới hạn trong giờ lab | Xác nhận qua Pricing Calculator |
| EBS root volume | Kích thước tiêu chuẩn, gp3 | Xác nhận qua Pricing Calculator |
| DynamoDB | Trong giới hạn Free Tier | $0.00 |
| Lambda | Trong giới hạn Free Tier | $0.00 |
| SNS | Trong giới hạn Free Tier | $0.00 |
| EventBridge | Trong giới hạn Free Tier | $0.00 |
| CloudWatch Logs | Trong giới hạn Free Tier | $0.00 |
| AWS Budgets | 2 budget đầu miễn phí | $0.00 |
| Systems Manager Parameter Store | Standard parameter | $0.00 |
| **Tổng** | | **Mục tiêu dưới $20** |

> Ngưỡng nội bộ của Lớp 2 được đặt thấp hơn tổng ngân sách (khoảng 70%) để bù cho độ trễ billing của Lớp 3.

### 7. Đánh giá rủi ro

#### Risk Matrix

| Rủi ro | Mức ảnh hưởng | Xác suất |
|--------|--------------|---------|
| Quên tắt máy thủ công | Cao | Trung bình |
| Cảnh báo AWS Budgets đến quá trễ | Cao | Chắc chắn (theo thiết kế) |
| Chi phí ước tính Lớp 2 lệch billing thực | Trung bình | Trung bình |
| Lambda dừng nhầm instance do cấu hình IAM sai | Cao | Thấp |
| Loại instance Windows không khả dụng trên Free Plan | Trung bình | Thấp |
| Lộ thông tin tài khoản trong ảnh chụp màn hình | Trung bình | Thấp |
| Trễ tiến độ trong 1 tuần | Trung bình | Trung bình |

#### Mitigation Strategies

- **Quên tắt máy:** Lớp 1 dừng máy theo lịch cố định, không phụ thuộc dữ liệu billing.
- **Budgets trễ:** Lớp 2 hành động trong vài phút dựa trên thời gian chạy và đơn giá đã cấu hình, độc lập với pipeline billing.
- **Chi phí ước tính lệch:** Ngưỡng nội bộ đặt thấp hơn tổng ngân sách. Đối chiếu chi phí thực với ước tính qua Cost Explorer là phần bắt buộc trong báo cáo cuối.
- **Dừng nhầm instance:** Quyền stop trong IAM role của mỗi Lambda giới hạn theo ARN instance cụ thể.
- **Instance không khả dụng:** Kiểm tra ngay ngày 1; chuyển sang loại nhỏ hơn nếu Console báo lỗi.

#### Contingency Plans

- Nếu Lambda hỏng: dừng máy thủ công từ Console và tắt các EventBridge rule.
- Nếu hết credit giữa dự án: terminate máy Windows ngay lập tức, tiếp tục viết tài liệu bằng ảnh đã có.
- Nếu ngân sách $20 không đủ: giảm giờ lab hoặc đổi loại instance nhỏ hơn, cập nhật lại Pricing Calculator.

### 8. Kết quả kỳ vọng

#### Technical Improvements

- Máy Windows EC2 tự dừng khi hết giờ lab hoặc khi chi phí ước tính vượt ngưỡng — không cần can thiệp thủ công.
- Tổng chi tiêu dự án dưới $20, xác nhận qua Cost Explorer sau khi dọn dẹp.
- Cả ba lớp bảo vệ đều có thể quan sát qua CloudWatch Logs, SNS email và log DynamoDB.

#### Long-term Value

Vì instance ID, đơn giá và ngưỡng đều được lưu trong Parameter Store, hệ thống có thể tái sử dụng cho bất kỳ tài khoản lab nào mà không cần thay đổi code. Mô hình ba lớp — lịch cố định, đo gần thời gian thực, billing backstop — áp dụng được cho bất kỳ môi trường AWS nào cần kiểm soát chi phí khi dữ liệu billing có độ trễ.

> **Phần đóng góp riêng:** Đóng góp chính của dự án là cơ chế tự đo chi phí gần thời gian thực ở Lớp 2. Thay vì chờ dữ liệu billing của AWS, hệ thống tính chi phí từ thời gian chạy thực tế và đơn giá đã cấu hình, cho phép hành động trong vài phút thay vì vài giờ. Thiết kế ba lớp bảo vệ được phát triển độc lập cho dự án này.
