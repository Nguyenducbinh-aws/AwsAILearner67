---
title: "Proposal"
date: 2026-11-18
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Cost Guard for Windows Labs on AWS
## Giải pháp bảo vệ credit ba lớp dùng Serverless

### 1. Tóm tắt điều hành

Cost Guard là hệ thống tự động hóa serverless dành cho sinh viên và nhóm nhỏ chạy máy lab Windows trên tài khoản AWS Free Plan. Máy Windows EC2 tính phí theo giờ, nên chỉ cần quên tắt vài ngày là có thể mất phần lớn credit. Dự án giải quyết vấn đề này bằng ba lớp bảo vệ tự động: lịch dừng theo giờ, bộ đo chi phí gần thời gian thực và một lớp dự phòng dùng AWS Budgets. Các dịch vụ hỗ trợ đều nằm trong AWS Free Tier; chi phí đáng kể duy nhất là máy Windows EC2.

### 2. Tuyên bố vấn đề

***Vấn đề hiện tại***

Credit AWS Free Plan có giới hạn. Một máy Windows EC2 (t3.medium) chạy liên tục 24/7 tốn khoảng $46.20 mỗi tháng, vượt quá ngân sách credit $20. AWS Budgets có cảnh báo chi phí, nhưng dữ liệu billing chỉ cập nhật vài lần mỗi ngày, nên không đủ nhanh để làm cơ chế bảo vệ thời gian thực.

***Giải pháp***

Cost Guard dùng ba lớp bảo vệ bổ sung cho nhau:

- **Lớp 1 — Lịch dừng (Schedule Guard):** EventBridge Scheduler tự dừng máy ngoài giờ lab. Lớp này không phụ thuộc dữ liệu billing nên đáng tin cậy nhất. Lịch chỉ dừng máy; việc bật máy vẫn làm thủ công để không tốn credit khi không có người dùng.
- **Lớp 2 — Bộ đo chi phí (Cost Meter, đóng góp riêng):** Lambda chạy mỗi 5 phút, ước tính chi phí của máy đang chạy dựa trên thời gian chạy và đơn giá đã cấu hình, rồi cộng với chi phí ước tính của các phiên trước trong tháng. Khi tổng vượt ngưỡng nội bộ, máy bị dừng và cảnh báo được gửi qua SNS.
- **Lớp 3 — Budget dự phòng (Budget Backstop):** AWS Budgets theo dõi chi tiêu thực của tài khoản. Khi chi tiêu tiến gần giới hạn, nó kích hoạt Lambda qua SNS để dừng máy như lưới an toàn cuối cùng.

***Lợi ích và ROI***

Với kế hoạch dùng 120 giờ lab mỗi tháng, chi phí ước tính khoảng $9.60 mỗi tháng. Cũng máy đó chạy 24/7 sẽ tốn $46.20 mỗi tháng. Cost Guard giúp tránh khoảng $36.60 mỗi tháng, tương đương khoảng 79%, và giữ dự án dưới ngân sách $20. Lớp 2 cho biết chi phí trong vài phút sau khi máy khởi động, không cần chờ billing cập nhật. Đơn giá và ngưỡng chi phí lưu trong Parameter Store thay vì viết cứng trong code, nên hệ thống dùng lại được cho loại instance hoặc tài khoản khác.

### 3. Kiến trúc giải pháp

Hệ thống chạy trong một AWS Region (US East, N. Virginia) và chỉ dùng dịch vụ serverless hoặc managed. EventBridge điều phối lịch dừng và việc kiểm tra chi phí định kỳ. Các hàm Lambda xử lý logic dừng máy và ước tính chi phí. DynamoDB ghi lại hoạt động từng phiên. SNS gửi thông báo. Parameter Store lưu cấu hình. Mỗi hàm Lambda có IAM role riêng với quyền tối thiểu.

![Sơ đồ kiến trúc](../../images/2-Proposal/architecture.png)

**Dịch vụ AWS sử dụng**

- **Amazon EC2 (Windows):** Máy lab được bảo vệ (t3.medium, Windows Server).
- **Amazon EBS (gp3, 30 GB):** Ổ đĩa gốc của máy lab. Được tính phí cả tháng, kể cả khi máy đang tắt.
- **Amazon EventBridge:** EventBridge Scheduler kích hoạt lịch dừng; EventBridge rule chạy bộ đo chi phí mỗi 5 phút.
- **AWS Lambda:** Thực thi logic của từng lớp bảo vệ.
- **Amazon DynamoDB:** Lưu hoạt động từng phiên và ước tính chi phí.
- **Amazon SNS:** Gửi email cảnh báo khi vượt ngưỡng hoặc khi có lỗi.
- **AWS Budgets:** Theo dõi chi tiêu thực và kích hoạt lớp bảo vệ cuối cùng.
- **AWS Systems Manager Parameter Store:** Lưu đơn giá theo giờ, ngưỡng chi phí và hạn mức ngân sách.
- **Amazon CloudWatch:** Thu thập log Lambda và cảnh báo khi hàm lỗi.
- **AWS IAM:** Cấp role quyền tối thiểu cho từng hàm Lambda.
- **AWS CloudFormation:** Đóng gói stack để triển khai lặp lại và dọn dẹp sạch sẽ.
- **AWS Cost Explorer:** Đối chiếu chi phí ước tính với chi phí thực tế cuối dự án.

**Thiết kế thành phần**

- **Lớp lịch dừng:** EventBridge Scheduler → Lambda `ScheduleHandler` → dừng EC2 theo lịch cố định và ghi sự kiện vào DynamoDB.
- **Bộ đo chi phí:** EventBridge rule (mỗi 5 phút) → Lambda `CostMeter` → đọc đơn giá và ngưỡng từ Parameter Store → tính chi phí ước tính từ bản ghi phiên trong DynamoDB → nếu vượt ngưỡng thì dừng máy và gửi cảnh báo SNS. Ngưỡng chỉ áp dụng cho chi phí chạy EC2; ổ EBS được tính riêng trong ngân sách.
- **Budget dự phòng:** AWS Budgets alarm → SNS topic → Lambda `BudgetBackstop` → dừng máy → gửi email xác nhận.
- **Thông báo:** Một SNS topic cho cảnh báo chi phí gửi người dùng; một SNS topic cho lỗi vận hành.
- **Dữ liệu:** Bảng DynamoDB ghi sự kiện bật/tắt máy và ước tính chi phí định kỳ của từng phiên.
- **Bảo mật:** Mỗi hàm Lambda có IAM role riêng, quyền dừng máy chỉ giới hạn cho instance đích.

**Business logic**

- **Bật máy:** Người dùng lab bật instance thủ công. Không lớp nào tự động bật máy, nên không tốn credit khi không có ai làm việc.
- **Quy tắc Lớp 1:** Mỗi ngày lúc 22:00 (Asia/Ho_Chi_Minh), nếu instance đang chạy thì dừng. Nếu đã dừng thì không làm gì.
- **Quy tắc Lớp 2:** Cứ 5 phút một lần, nếu instance đang chạy thì ước tính chi phí của phiên hiện tại (số giờ chạy × đơn giá theo giờ) rồi cộng với chi phí ước tính của các phiên trước trong cùng tháng dương lịch. Nếu tổng đạt ngưỡng (khoảng $14, tức 70% của ngân sách $20) thì dừng instance, ghi sự kiện và gửi cảnh báo. Nếu chưa đạt thì chỉ ghi lại ước tính. Nếu instance đã dừng thì không làm gì.
- **Bật lại sau khi Lớp 2 dừng máy:** Tổng chi phí trong tháng vẫn trên ngưỡng, nên lần chạy kế tiếp (trong vòng 5 phút) sẽ dừng máy lại. Máy bị khóa cho đến tháng dương lịch tiếp theo, khi bộ đếm bắt đầu lại.
- **Quy tắc Lớp 3:** AWS Budgets theo dõi chi tiêu thực của tài khoản và gửi thông báo ở mức 80% và 100% hạn mức $20. Mỗi thông báo kích hoạt `BudgetBackstop`, hàm này dừng instance và gửi email xác nhận. Dữ liệu billing chỉ cập nhật vài lần mỗi ngày nên lớp này chậm theo thiết kế và chỉ làm lưới an toàn cuối cùng.
- **Thứ tự bảo vệ:** Lớp 1 cố định theo giờ, Lớp 2 phản ứng trong vài phút, Lớp 3 theo billing thực. Lớp nào cũng dừng được máy, không lớp nào bật được máy, và dừng một máy đã dừng thì không có tác dụng, nên các lớp không xung đột với nhau.
- **Thông báo:** Mỗi lần dừng tự động gửi một email qua topic cảnh báo. Lỗi của hàm tạo CloudWatch alarm và gửi email qua topic lỗi.
- **Khi một lớp lỗi:** Hai lớp còn lại vẫn hoạt động, nên một lỗi đơn lẻ không để máy mất bảo vệ.
- **Phạm vi chi phí:** Ngưỡng của Lớp 2 chỉ tính chi phí chạy EC2. Ổ EBS ($2.40 mỗi tháng) là chi phí cố định, tính riêng trong ngân sách.
- **Cấu hình:** Đơn giá theo giờ, ngưỡng và hạn mức ngân sách là các giá trị trong Parameter Store. Đổi giá trị có hiệu lực từ lần chạy kế tiếp, không cần deploy lại.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

1. **Nền tảng:** Tạo IAM role, SNS topic, bảng DynamoDB, tham số Parameter Store và AWS Budget. Chốt loại instance và region, lấy đơn giá theo giờ từ AWS Pricing Calculator.
2. **Lớp 1:** Khởi chạy Windows EC2, deploy `ScheduleHandler`, cấu hình EventBridge Scheduler và kiểm tra lịch dừng hoạt động đúng.
3. **Lớp 2:** Deploy `CostMeter`, cấu hình EventBridge rule định kỳ và thử logic ngưỡng bằng dữ liệu chi phí giả lập.
4. **Lớp 3 và giám sát:** Cấu hình Budgets alarm, deploy `BudgetBackstop`, thêm CloudWatch alarm cho lỗi Lambda và chạy thử end-to-end cả ba lớp.
5. **Demo và tài liệu:** Quay demo khi tài nguyên còn chạy, dọn dẹp, đối chiếu chi phí bằng Cost Explorer và viết workshop.

### 5. Lộ trình & Mốc triển khai

**Lộ trình dự án**

- **Tháng 0 (trước thực tập):** Tham gia chương trình FCAJ, đọc nội quy thực tập và chuẩn bị môi trường làm việc.
- **Tháng 1 (11/09 – 10/10/2026):** Học nền tảng AWS: tạo tài khoản, MFA, IAM, Budgets, Support, Console và CLI.
- **Tháng 2 (11/10 – 10/11/2026):** Tìm hiểu các dịch vụ dùng trong dự án (Lambda, EventBridge, SNS, DynamoDB, Parameter Store, CloudWatch) và thiết kế ba lớp bảo vệ.
- **Tháng 3 (11/11 – 14/12/2026):** Xây dựng và viết tài liệu cho giải pháp.
  - Tuần 1: Nền tảng và Lớp 1 (chốt loại instance và đơn giá, tạo tài nguyên nền, lịch dừng).
  - Tuần 2: Lớp 2 (bộ đo chi phí và thử ngưỡng) và sơ đồ kiến trúc.
  - Tuần 3: Lớp 3, CloudWatch alarm và kiểm thử end-to-end.
  - Tuần 4: Quay demo, dọn dẹp, đối chiếu Cost Explorer, viết workshop và xuất bản site.
- **Sau triển khai:** Giữ giải pháp làm mẫu dùng lại; mở rộng cho loại instance hoặc tài khoản khác bằng cách đổi giá trị trong Parameter Store.

### 6. Ước tính ngân sách

Ước tính được lập bằng AWS Pricing Calculator cho US East (N. Virginia), giá on-demand, với kế hoạch dùng 120 giờ lab mỗi tháng (4 giờ mỗi ngày trong 30 ngày).

<!-- TODO: thêm link chia sẻ AWS Pricing Calculator và file PDF xuất ra (static/attachments/budget_estimation.pdf), rồi tham chiếu ở đây. -->

**Chi phí hạ tầng**

- Amazon EC2 Windows (t3.medium): 120 giờ × $0.06 mỗi giờ = $7.20 mỗi tháng.
- Amazon EBS (gp3, 30 GB): $2.40 mỗi tháng, tính cả tháng kể cả khi máy tắt.
- Địa chỉ Public IPv4 (nếu instance dùng IP công cộng): khoảng $0.60 mỗi tháng cho 120 giờ. Khoản này chưa có trong estimate hiện tại của Calculator và cần bổ sung.
- AWS Lambda, Amazon DynamoDB, Amazon SNS, Amazon EventBridge, Amazon CloudWatch, Parameter Store (tham số standard), AWS Budgets: $0.00 mỗi tháng trong Free Tier.

**Tổng:** khoảng $9.60 mỗi tháng (khoảng $10.20 nếu tính địa chỉ Public IPv4), dưới ngân sách $20.

Để so sánh, cùng máy đó chạy 24/7 (730 giờ) tốn $46.20 mỗi tháng: $43.80 cho EC2 và $2.40 cho EBS.

Ngưỡng nội bộ của Lớp 2 đặt ở khoảng 70% ngân sách ($14). Với đơn giá $0.06 mỗi giờ, con số này tương đương khoảng 233 giờ chạy, cao hơn nhiều so với kế hoạch 120 giờ. Khoảng chênh này chừa chỗ cho độ trễ billing của Lớp 3.

### 7. Đánh giá rủi ro

**Ma trận rủi ro**

- Quên dừng máy thủ công: tác động cao, xác suất trung bình.
- Cảnh báo AWS Budgets đến quá trễ: tác động cao, xác suất cao (theo thiết kế).
- Ước tính của Lớp 2 lệch so với billing thực: tác động trung bình, xác suất trung bình.
- Lambda dừng nhầm máy do cấu hình IAM sai: tác động cao, xác suất thấp.
- Loại instance Windows đã chọn không dùng được trên Free Plan: tác động trung bình, xác suất thấp.
- Lộ credentials hoặc account ID trong ảnh chụp màn hình: tác động trung bình, xác suất thấp.
- Trễ tiến độ trong tháng cuối của đợt thực tập: tác động trung bình, xác suất trung bình.

**Chiến lược giảm thiểu**

- Quên tắt máy: Lớp 1 dừng máy theo lịch cố định, không phụ thuộc dữ liệu chi phí.
- Budgets trễ: Lớp 2 phản ứng trong vài phút dựa trên thời gian chạy và đơn giá đã cấu hình.
- Ước tính lệch: đặt ngưỡng thấp hơn ngân sách và đối chiếu với Cost Explorer vào cuối dự án.
- Dừng nhầm máy: mỗi role Lambda chỉ có quyền dừng instance đích.
- Instance không dùng được: kiểm tra trong tuần đầu, đổi sang loại nhỏ hơn nếu cần.
- Lộ thông tin: che account ID và mọi khóa trong từng ảnh chụp trước khi đăng.
- Trễ tiến độ: viết workshop song song khi xây từng lớp thay vì để dồn cuối, và làm các lớp theo thứ tự quan trọng.

**Kế hoạch dự phòng**

- Nếu một lớp Lambda lỗi: dừng máy thủ công trên AWS Console và tắt các rule EventBridge.
- Nếu hết credit giữa chừng: terminate máy Windows ngay và hoàn thành tài liệu bằng các ảnh đã chụp.
- Nếu ngân sách $20 không đủ: giảm số giờ lab mỗi ngày hoặc đổi sang loại instance nhỏ hơn rồi cập nhật ước tính.

### 8. Kết quả kỳ vọng

**Cải tiến kỹ thuật**

- Máy Windows EC2 tự dừng khi hết giờ lab hoặc khi chi phí ước tính vượt ngưỡng, không cần thao tác thủ công.
- Tổng chi tiêu nằm trong ngân sách $20, được xác nhận bằng Cost Explorer sau khi dọn dẹp.
- Có thể quan sát cả ba lớp qua CloudWatch Logs, email cảnh báo SNS và nhật ký hoạt động trong DynamoDB.

**Giá trị dài hạn**

Vì instance ID, đơn giá theo giờ và ngưỡng đều lưu trong Parameter Store, hệ thống dùng lại được cho bất kỳ tài khoản lab Windows nào mà không cần sửa code. Mẫu ba lớp (lịch cố định, bộ đo chi phí gần thời gian thực và lớp dự phòng billing) áp dụng được cho mọi môi trường AWS cần kiểm soát chi phí mà bị hạn chế bởi độ trễ của dữ liệu billing.
