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

Cost Guard là hệ thống bảo vệ serverless dành cho sinh viên và đội nhỏ chạy máy lab Windows trên tài khoản AWS Free Plan. Máy Windows EC2 tính phí theo giờ, chỉ cần quên tắt vài ngày là hết toàn bộ credit $20. Cost Guard giải quyết vấn đề này bằng ba lớp tự động: lịch dừng/bật theo giờ, bộ đo chi phí gần thời gian thực xây dựng trên Lambda và DynamoDB, và một lớp dự phòng dùng AWS Budgets. Tất cả dịch vụ điều phối đều nằm trong Free Tier của AWS.

### 2. Vấn đề cần giải quyết

#### Vấn đề là gì?

Credit AWS Free Plan có giới hạn. Một máy Windows EC2 chạy liên tục có thể tiêu tốn $1–$3 chỉ trong một cuối tuần — chiếm phần đáng kể trong ngân sách $20. AWS Budgets có cảnh báo chi phí nhưng **dữ liệu billing chỉ cập nhật tối đa 3 lần/ngày**, cách nhau 8–12 giờ, nên không thể dùng làm cơ chế ngắt thời gian thực.

#### Giải pháp

Cost Guard dùng ba lớp bảo vệ bổ sung cho nhau:

- **Lớp 1 — Lịch dừng/bật:** EventBridge Scheduler tự dừng máy ngoài giờ lab và bật lại khi cần. Lớp này không phụ thuộc vào dữ liệu billing, chắc chắn nhất.
- **Lớp 2 — Tự đo chi phí (đóng góp riêng):** Lambda `CostMeter` chạy mỗi 15 phút, đọc đơn giá/giờ từ Parameter Store, nhân với số giờ máy đã chạy (ghi trong DynamoDB), so với ngưỡng nội bộ (~70% ngân sách $20). Nếu vượt ngưỡng: dừng máy + gửi SNS. Lớp này gần thời gian thực, không chờ billing.
- **Lớp 3 — Budget dự phòng:** AWS Budgets theo dõi chi tiêu thực của toàn tài khoản ở mức $20. Khi kích hoạt, Budgets gửi đến SNS topic → Lambda `BudgetBackstop` dừng máy và gửi email. Lớp này dùng số liệu billing thật nhưng có độ trễ, chỉ là lưới an toàn cuối cùng.

### Benefits and Return on Investment

- Ngăn mất credit ngoài ý muốn do quên tắt máy.
- Lớp 2 cung cấp thông tin chi phí trong vòng 15 phút kể từ khi máy khởi động — thay vì chờ 8–12 giờ nếu chỉ dùng Budgets.
- Chi phí hạ tầng ước tính: **~$3.29/tháng** (máy Windows là phần lớn); các dịch vụ serverless nằm trong Free Tier.
- Đơn giá và ngưỡng được lưu trong Parameter Store, không hardcode — tái sử dụng được cho bất kỳ lab nào.

### 3. Kiến trúc giải pháp

Kiến trúc nằm trong một AWS Region, chỉ dùng dịch vụ serverless hoặc managed chi phí thấp. EventBridge Scheduler điều khiển Lớp 1 theo cron. Một EventBridge rule riêng kích hoạt `CostMeter` mỗi 15 phút. AWS Budgets giám sát billing thực và kích hoạt Lớp 3 qua SNS khi tổng chi phí tài khoản tiến gần $20. Tất cả Lambda ghi log vào CloudWatch Logs; CloudWatch Alarm theo dõi lỗi Lambda và gửi SNS thông báo vận hành. IAM role theo nguyên tắc quyền tối thiểu.

![Sơ đồ kiến trúc](/images/2-Proposal/architecture.png)

*Sơ đồ được vẽ bằng draw.io với icon AWS chính thức. Xem chi tiết từng thành phần ở mục Component Design bên dưới.*

### Các dịch vụ AWS sử dụng

- **Amazon EC2 (Windows, t3.micro hoặc t2.micro)**: Máy lab được bảo vệ; bị dừng tự động bởi cả ba lớp.
- **Amazon EventBridge Scheduler**: Kích hoạt lịch dừng/bật (Lớp 1) và gọi CostMeter định kỳ (Lớp 2).
- **AWS Lambda**: Ba hàm — `ScheduleHandler` (Lớp 1), `CostMeter` (Lớp 2), `BudgetBackstop` (Lớp 3).
- **Amazon DynamoDB**: Lưu sự kiện bật/tắt máy và chi phí ước tính từng phiên.
- **Amazon SNS**: Gửi email cảnh báo khi vượt ngưỡng chi phí và khi Lambda lỗi.
- **AWS Budgets**: Giám sát chi tiêu thực toàn tài khoản và kích hoạt Lớp 3 khi đến ngưỡng $20.
- **AWS Systems Manager Parameter Store**: Lưu đơn giá/giờ và ngưỡng nội bộ — không có giá trị nào hardcode trong Lambda.
- **Amazon CloudWatch (Logs + Alarms)**: Thu thập log Lambda; cảnh báo khi hàm lỗi.
- **AWS IAM**: Role thực thi quyền tối thiểu cho từng Lambda.
- **AWS CloudFormation (tùy chọn)**: Đóng gói toàn bộ stack thành IaC để triển khai lại và dọn dẹp sạch sẽ.

### Component Design

- **Scheduler Layer**: EventBridge Scheduler (cron) → Lambda `ScheduleHandler` → `ec2:StopInstances` / `ec2:StartInstances` trên đúng instance ID.
- **Cost Meter**: EventBridge rule (rate 15 phút) → Lambda `CostMeter` → đọc `price_per_hour` và `cost_threshold` từ Parameter Store → đọc timestamp bật máy gần nhất từ DynamoDB → tính chi phí ước tính → ghi vào DynamoDB → nếu chi phí ≥ ngưỡng: dừng máy + gửi SNS.
- **Budget Backstop**: AWS Budgets alarm → SNS topic → Lambda `BudgetBackstop` → dừng máy → gửi email xác nhận qua SNS.
- **Notification**: Một SNS topic cho email cảnh báo đến người dùng; một SNS topic riêng cho cảnh báo vận hành (Lambda lỗi).
- **Data (DynamoDB)**: Bảng `CostGuardLog`, partition key `instanceId`, sort key `timestamp`; thuộc tính: `action` (start/stop/meter), `estimatedCostUSD`, `runningMinutes`.
- **Security (IAM)**: Ba role Lambda. Role `ScheduleHandler`: `ec2:StopInstances`, `ec2:StartInstances`, `ec2:DescribeInstances` giới hạn trên ARN instance cụ thể; `logs:CreateLogGroup`, `logs:PutLogEvents`. Role `CostMeter`: thêm `dynamodb:PutItem`, `dynamodb:GetItem` trên bảng `CostGuardLog` và `ssm:GetParameter` trên hai đường dẫn tham số. Role `BudgetBackstop`: `ec2:StopInstances` trên đúng instance và `sns:Publish` trên alert topic.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

- **Giai đoạn 1 — Tính giá và thiết kế (Ngày 1):** Xác định loại instance và region; dùng AWS Pricing Calculator lấy đơn giá on-demand theo giờ; lập bảng phân chia ngân sách $20; tạo IAM role, SNS topic, bảng DynamoDB, tham số Parameter Store.
- **Giai đoạn 2 — Lớp 1 (Ngày 2):** Khởi chạy Windows EC2; viết và deploy `ScheduleHandler`; cấu hình EventBridge Scheduler; kiểm tra dừng/bật bằng trigger thủ công.
- **Giai đoạn 3 — Lớp 2 (Ngày 3):** Viết và deploy `CostMeter`; cấu hình EventBridge rule 15 phút; thử nghiệm logic ngưỡng với giá trị chi phí giả lập cao trong DynamoDB; xác minh SNS và dừng máy.
- **Giai đoạn 4 — Lớp 3 + quan sát (Ngày 4):** Cấu hình AWS Budgets alarm; viết và deploy `BudgetBackstop`; thêm CloudWatch Alarm trên metric lỗi Lambda; kiểm tra end-to-end với cả ba lớp hoạt động.
- **Giai đoạn 5 — Demo, dọn dẹp, tài liệu (Ngày 5–7):** Quay video demo khi tài nguyên còn sống; xóa stack CloudFormation; chụp ảnh Cost Explorer; viết nội dung workshop và Proposal.

**Yêu cầu kỹ thuật**

- Tài khoản AWS còn ít nhất $5 credit hoặc đang trong Free Plan.
- AWS CLI v2 cấu hình với IAM user (không dùng root key).
- Lambda runtime Python 3.12; `boto3` có sẵn trong managed runtime.
- Kiến thức cần có: Lambda, EventBridge, SNS, DynamoDB, Parameter Store, cú pháp IAM policy.
- draw.io (desktop hoặc web) để vẽ sơ đồ kiến trúc.
- Hugo cài đặt cục bộ để xem trước workshop trước khi push.

### 5. Lịch trình & Cột mốc

**Kế hoạch 7 ngày**

| Ngày | Việc cần làm | Sản phẩm |
|------|-------------|----------|
| 1 | Chốt loại instance và region; chạy AWS Pricing Calculator; tạo IAM, SNS, DynamoDB, Parameter Store | Bảng Cost, hạ tầng nền |
| 2 | Khởi chạy Windows EC2; deploy `ScheduleHandler`; cấu hình EventBridge Scheduler; test | Lớp 1 hoạt động |
| 3 | Deploy `CostMeter`; cấu hình rule 15 phút; giả lập vượt ngưỡng; xác minh SNS + dừng | Lớp 2 hoạt động |
| 4 | Cấu hình Budgets alarm; deploy `BudgetBackstop`; CloudWatch Alarm; test end-to-end | Lớp 3 + quan sát đầy đủ |
| 5 | Quay video demo (máy còn sống); dọn tài nguyên; chụp Cost Explorer | Video, ảnh dọn dẹp |
| 6 | Vẽ sơ đồ kiến trúc bằng draw.io; viết workshop 5.1–5.8 (tiếng Việt) | Bản nháp workshop |
| 7 | Viết Proposal; dịch sang tiếng Anh; chạy `hugo server`; push; kiểm tra link | Site hoàn chỉnh |

### 6. Dự toán chi phí

Dựa trên AWS Pricing Calculator cho **ap-southeast-1 (Singapore)**, giá on-demand. Máy Windows được Lớp 1 giới hạn khoảng 2 giờ/ngày trong giờ lab.

[Ước tính AWS Pricing Calculator](https://calculator.aws/pricing/2/savedsettings) *(cập nhật link sau khi export từ Calculator)*

File PDF ước tính đặt tại: [`/attachments/budget_estimation.pdf`](/attachments/budget_estimation.pdf)

### Infrastructure Costs

| Dịch vụ | Giả định | Chi phí/tháng |
|---------|---------|--------------|
| EC2 Windows t3.micro (ap-southeast-1) | 2 giờ/ngày × 30 ngày = 60 giờ | ~$3.00 |
| EBS gp3 30 GB (root volume) | 30 GB × $0.096/GB | ~$0.29 |
| DynamoDB | < 25 GB, < 25 WCU/RCU (Free Tier) | $0.00 |
| Lambda | < 1M request/tháng (Free Tier) | $0.00 |
| SNS | < 1.000 email (Free Tier) | $0.00 |
| EventBridge Scheduler | < 14M lần gọi (Free Tier) | $0.00 |
| CloudWatch Logs | < 5 GB (Free Tier) | $0.00 |
| AWS Budgets | 2 budget đầu miễn phí | $0.00 |
| Systems Manager Parameter Store | Standard parameter, miễn phí | $0.00 |
| **Tổng** | | **~$3.29/tháng** |
| **Dự báo 12 tháng** | | **~$39.48** |

> **Lưu ý:** Ngân sách dự án $20 đủ cho khoảng 6–7 giờ chạy máy Windows (theo đơn giá trên). Lớp 1 và Lớp 2 cộng lại giữ tổng chi tiêu dưới mức này. Ngưỡng nội bộ của Lớp 2 đặt ở $14 (~70% ngân sách) để còn dư bù cho độ trễ billing của Lớp 3. Tất cả dịch vụ serverless nằm trong Free Tier trong điều kiện sử dụng bình thường.

### 7. Đánh giá rủi ro

#### Risk Matrix

| Rủi ro | Mức ảnh hưởng | Xác suất | Lớp liên quan |
|--------|--------------|---------|--------------|
| Quên tắt máy Windows thủ công | Cao | Trung bình | Cả 3 lớp đều giảm thiểu |
| AWS Budgets trễ 8–12 giờ | Cao | Chắc chắn (theo thiết kế) | Lớp 3 chỉ là dự phòng; Lớp 1–2 hành động trước |
| Chi phí ước tính của CostMeter lệch billing thực | Trung bình | Trung bình | Ghi nhận; đối chiếu với Cost Explorer trong báo cáo |
| Lambda lỗi quyền, dừng nhầm instance | Cao | Thấp | Giới hạn role theo ARN instance cụ thể |
| Loại instance Windows bị chặn trên Free Plan | Trung bình | Thấp | Thử ngay ngày 1; đổi sang loại nhỏ hơn nếu cần |
| Lộ email/Account ID trong ảnh chụp màn hình | Trung bình | Thấp | Che trước khi lưu và commit |
| Trễ tiến độ (deadline 1 tuần) | Trung bình | Trung bình | Ưu tiên Lớp 1 và 2 trước; Lớp 3 cắt bớt nếu cần |

#### Mitigation Strategies

- **Quên tắt máy:** EventBridge Scheduler dừng máy mỗi ngày vào cuối giờ lab, bất kể dữ liệu billing.
- **Budgets trễ:** `CostMeter` tính chi phí từ đầu (giờ chạy × đơn giá) và hành động trong 15 phút, độc lập với pipeline billing.
- **Chi phí ước tính lệch:** Ngưỡng nội bộ thấp hơn 30% so với ngân sách ($14 vs $20). Đối chiếu Cost Explorer là phần bắt buộc trong báo cáo cuối.
- **Dừng nhầm máy:** ARN `ec2:StopInstances` trong mỗi Lambda role giới hạn theo đúng instance ID. Thử nghiệm trên máy đã dừng trước khi bật lịch chạy.
- **Instance bị chặn:** Nhiệm vụ ngày 1 là thử khởi chạy loại instance đã chọn; nếu Console báo lỗi, chuyển sang `t2.micro` hoặc `t2.small`.

#### Contingency Plans

- Nếu tất cả lớp Lambda hỏng: dừng thủ công từ AWS Console và tắt các EventBridge rule.
- Nếu hết credit giữa dự án: xóa stack CloudFormation hoặc chạy `aws ec2 terminate-instances` để xóa ngay máy Windows, sau đó tiếp tục viết workshop bằng ảnh đã chụp.
- Nếu ngân sách $20 không đủ: giảm giờ lab hoặc chuyển sang instance nhỏ hơn; cập nhật lại Pricing Calculator.

### 8. Kết quả kỳ vọng

#### Technical Improvements

- Máy Windows EC2 tự dừng cuối mỗi phiên lab — không cần hành động thủ công.
- Tổng chi tiêu dự án dưới $20, xác nhận qua Cost Explorer sau khi dọn dẹp.
- `CostMeter` cung cấp ước tính chi phí cập nhật mỗi 15 phút, so với ngưỡng có thể điều chỉnh trong Parameter Store.
- CloudWatch Alarm và SNS thông báo ngay khi Lambda lỗi.
- Toàn bộ hạ tầng có thể xóa bằng một lệnh xóa CloudFormation stack, không để lại tài nguyên thừa.

#### Long-term Value

Hệ thống được tham số hóa (instance ID, đơn giá, ngưỡng đều trong Parameter Store) và có thể tái sử dụng cho bất kỳ tài khoản lab nào mà không cần thay đổi code. Mở rộng sang nhiều máy chỉ cần thêm bản ghi lịch và điều chỉnh DynamoDB schema. Mô hình ba lớp — lịch cố định, đo gần thời gian thực, billing backstop — áp dụng được cho bất kỳ tài khoản AWS nào cần kiểm soát chi phí khi dữ liệu billing có độ trễ.

> **Phần đóng góp riêng:** Cơ chế tự đo chi phí gần thời gian thực trong Lớp 2 (`CostMeter`) là đóng góp chính của dự án này. Thay vì chờ dữ liệu billing của AWS, hệ thống tính chi phí từ số giờ chạy thực tế và đơn giá cấu hình thủ công. Thiết kế ba lớp bảo vệ và tích hợp giữa EventBridge, Lambda, DynamoDB và Parameter Store được xây dựng độc lập và không sao chép từ bất kỳ workshop AWS nào.
