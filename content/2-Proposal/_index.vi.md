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

Credit AWS Free Plan có giới hạn. Một máy Windows EC2 (t3.medium) chạy liên tục 24/7 tốn khoảng $46.22 mỗi tháng, vượt quá ngân sách credit $20. AWS Budgets có cảnh báo chi phí, nhưng dữ liệu billing chỉ cập nhật vài lần mỗi ngày, nên không đủ nhanh để làm cơ chế bảo vệ thời gian thực.

***Giải pháp***

Cost Guard dùng ba lớp bảo vệ bổ sung cho nhau:

- **Lớp 1 — Lịch dừng (Schedule Guard):** EventBridge Scheduler tự dừng máy ngoài giờ lab. Lớp này không phụ thuộc dữ liệu billing nên đáng tin cậy nhất. Lịch chỉ dừng máy; việc bật máy vẫn làm thủ công để không tốn credit khi không có người dùng.
- **Lớp 2 — Bộ đo chi phí (Cost Meter, đóng góp riêng):** Lambda chạy mỗi 5 phút, ước tính chi phí của máy đang chạy dựa trên thời gian chạy và đơn giá đã cấu hình, rồi cộng với chi phí ước tính của các phiên trước trong tháng. Khi tổng vượt ngưỡng nội bộ, máy bị dừng và cảnh báo được gửi qua SNS.
- **Lớp 3 — Budget dự phòng (Budget Backstop):** AWS Budgets theo dõi chi tiêu thực của tài khoản. Khi chi tiêu tiến gần giới hạn, nó kích hoạt Lambda qua SNS để dừng máy như lưới an toàn cuối cùng.

***Lợi ích và ROI***

Với kế hoạch dùng 120 giờ lab mỗi tháng, chi phí ước tính khoảng $9.62 mỗi tháng. Cũng máy đó chạy 24/7 sẽ tốn $46.22 mỗi tháng. Cost Guard giúp tránh khoảng $36.60 mỗi tháng, tương đương khoảng 79%, và giữ dự án dưới ngân sách $20. Lớp 2 cho biết chi phí trong vài phút sau khi máy khởi động, không cần chờ billing cập nhật. Đơn giá và ngưỡng chi phí lưu trong Parameter Store thay vì viết cứng trong code, nên hệ thống dùng lại được cho loại instance hoặc tài khoản khác.

***Mục tiêu và phạm vi***

Cost Guard dành cho sinh viên và nhóm nhỏ chạy một máy lab Windows trên tài khoản AWS có credit hạn chế.

Mục tiêu:

- Tự động dừng máy khi hết giờ lab.
- Dừng máy khi chi phí ước tính trong tháng đạt ngưỡng nội bộ (khoảng $14).
- Dừng máy như phương án cuối cùng khi chi tiêu thực đạt 80% và 100% ngân sách $20.
- Giữ tổng chi tiêu của dự án dưới $20.

Tiêu chí hoàn thành:

- Lớp 1: máy được dừng lúc 22:00, có thể thấy trong log Lambda và bản ghi DynamoDB.
- Lớp 2: với dữ liệu chi phí giả lập, máy được dừng trong vòng 5 phút sau khi đạt ngưỡng và có email cảnh báo.
- Lớp 3: một thông báo thử trên budget topic làm máy dừng và gửi email xác nhận.
- Cost Explorer sau khi dọn dẹp cho thấy tổng chi tiêu dưới $20, không còn instance hay ổ đĩa nào sót lại.

Ngoài phạm vi: quản lý nhiều instance hoặc nhiều tài khoản, tự động bật máy, dashboard web, và kiểm soát chi phí cho các dịch vụ khác ngoài máy lab.

### 3. Kiến trúc giải pháp

Hệ thống chạy trong một AWS Region (US East, N. Virginia) và chỉ dùng dịch vụ serverless hoặc managed. EventBridge điều phối lịch dừng và việc kiểm tra chi phí định kỳ. Các hàm Lambda xử lý logic dừng máy và ước tính chi phí. DynamoDB ghi lại hoạt động từng phiên. SNS gửi thông báo. Parameter Store lưu cấu hình. Mỗi hàm Lambda có IAM role riêng với quyền tối thiểu.

![Sơ đồ kiến trúc](../../images/2-Proposal/architecture.png)

**Dịch vụ AWS sử dụng**

- **Amazon EC2 (Windows):** Máy lab được bảo vệ (t3.medium, Windows Server).
- **Amazon EBS (gp3, 30 GB):** Ổ đĩa gốc của máy lab. Được tính phí cả tháng, kể cả khi máy đang tắt.
- **Amazon EventBridge:** EventBridge Scheduler kích hoạt lịch dừng và chạy bộ đo chi phí mỗi 5 phút. Cả hai đều dùng lịch (schedule).
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

- **Lớp lịch dừng:** EventBridge Scheduler → Lambda ScheduleHandler → dừng EC2 theo lịch cố định và ghi sự kiện vào DynamoDB.
- **Bộ đo chi phí:** EventBridge Scheduler (mỗi 5 phút) → Lambda CostMeter → đọc đơn giá và ngưỡng từ Parameter Store → tính chi phí ước tính từ bản ghi phiên trong DynamoDB → nếu vượt ngưỡng thì dừng máy và gửi cảnh báo SNS. Ngưỡng chỉ áp dụng cho chi phí chạy EC2; ổ EBS được tính riêng trong ngân sách.
- **Budget dự phòng:** AWS Budgets alarm → SNS topic → Lambda BudgetBackstop → dừng máy → gửi email xác nhận.
- **Thông báo:** Một SNS topic cho cảnh báo chi phí gửi người dùng; một SNS topic cho lỗi vận hành.
- **Dữ liệu:** Bảng DynamoDB ghi sự kiện bật/tắt máy và ước tính chi phí định kỳ của từng phiên.
- **Bảo mật:** Mỗi hàm Lambda có IAM role riêng, quyền dừng máy chỉ giới hạn cho instance đích.

**Các cân nhắc theo Well-Architected**

- **Operational Excellence:** Log Lambda, CloudWatch alarm cho lỗi hàm, email SNS và bản ghi phiên trong DynamoDB cho biết hệ thống đã làm gì và vì sao.
- **Security:** Mỗi hàm có IAM role riêng, quyền dừng máy chỉ giới hạn cho instance đích. Không có access key hay mật khẩu nào viết trong code; Parameter Store chỉ chứa đơn giá, ngưỡng và hạn mức.
- **Reliability:** Ba lớp độc lập cùng bảo vệ máy, nên một lớp lỗi không làm mất bảo vệ. Dừng một máy đã dừng thì không có tác dụng, nên chạy lặp lại vẫn an toàn.
- **Performance Efficiency:** t3.medium (2 vCPU, 4 GiB) có kích thước phù hợp cho một phiên lab. Lambda không có máy chủ nào cần chọn cỡ, và chu kỳ 5 phút (khoảng 8,640 lần chạy mỗi tháng) cân bằng giữa tốc độ phản ứng và số lần chạy.
- **Cost Optimization:** Xem danh sách Tối ưu chi phí ở mục 6.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

1. **Nền tảng:** Tạo IAM role, SNS topic, bảng DynamoDB, tham số Parameter Store và AWS Budget. Chốt loại instance và region, lấy đơn giá theo giờ từ AWS Pricing Calculator.
2. **Lớp 1:** Khởi chạy Windows EC2, deploy ScheduleHandler, cấu hình EventBridge Scheduler và kiểm tra lịch dừng hoạt động đúng.
3. **Lớp 2:** Deploy CostMeter, cấu hình lịch EventBridge Scheduler 5 phút và thử logic ngưỡng bằng dữ liệu chi phí giả lập.
4. **Lớp 3 và giám sát:** Cấu hình Budgets alarm, deploy BudgetBackstop, thêm CloudWatch alarm cho lỗi Lambda và chạy thử end-to-end cả ba lớp.
5. **Demo và tài liệu:** Quay demo khi tài nguyên còn chạy, dọn dẹp, đối chiếu chi phí bằng Cost Explorer và viết workshop.

### 5. Lộ trình & Mốc triển khai

**Lộ trình dự án**

- **Tháng 1 (07/09 – 06/10/2026):** Tham gia chương trình FCAJ, đọc nội quy thực tập và chuẩn bị môi trường làm việc. Sau đó học nền tảng AWS: tạo tài khoản, MFA, IAM, Budgets, Support, Console và CLI.
- **Tháng 2 (07/10 – 06/11/2026):** Tìm hiểu các dịch vụ dùng trong dự án (Lambda, EventBridge, SNS, DynamoDB, Parameter Store, CloudWatch) và thiết kế ba lớp bảo vệ.
- **Tháng 3 (07/11 – 14/12/2026):** Xây dựng và viết tài liệu cho giải pháp.
  - Tuần 1: Nền tảng và Lớp 1 (chốt loại instance và đơn giá, tạo tài nguyên nền, lịch dừng).
  - Tuần 2: Lớp 2 (bộ đo chi phí và thử ngưỡng) và sơ đồ kiến trúc.
  - Tuần 3: Lớp 3, CloudWatch alarm và kiểm thử end-to-end.
  - Tuần 4: Quay demo, dọn dẹp, đối chiếu Cost Explorer, viết workshop và xuất bản site.
- **Sau triển khai:** Giữ giải pháp làm mẫu dùng lại; mở rộng cho loại instance hoặc tài khoản khác bằng cách đổi giá trị trong Parameter Store.

### 6. Ước tính ngân sách

Ước tính được lập bằng AWS Pricing Calculator cho US East (N. Virginia), giá on-demand, với kế hoạch dùng 120 giờ lab mỗi tháng (4 giờ mỗi ngày trong 30 ngày).

<!-- TODO: thêm link chia sẻ AWS Pricing Calculator và file PDF xuất ra (static/attachments/budget_estimation.pdf), rồi tham chiếu ở đây. -->

**Chi phí hạ tầng**

- Amazon EC2 Windows (t3.medium): 120 giờ × $0.06 mỗi giờ = $7.20 mỗi tháng. Chế độ Guided của Calculator cố định mức dùng ở 730 giờ cho mỗi instance, nên 120 giờ được nhập bằng 0.1644 instance (120 ÷ 730), cho ra 120.012 giờ.
- Amazon EBS (gp3, 30 GB): $2.40 mỗi tháng, tính cả tháng kể cả khi máy tắt.
- Địa chỉ Public IPv4: $0.005 mỗi giờ cho địa chỉ đang dùng, nên 120 giờ × $0.005 = $0.60 mỗi tháng (cần thêm vào estimate của Calculator).
- AWS Lambda: khoảng $0.02 mỗi tháng (8,700 yêu cầu và 1,087.5 GB-giây với 128 MB và 1 giây mỗi lần chạy), tính không có Free Tier.
- Amazon DynamoDB (on-demand): khoảng $0.002 mỗi tháng (2,000 lần ghi, 7,500 đơn vị đọc và 0.001 GB lưu trữ).
- Amazon CloudWatch: 3 alarm tiêu chuẩn × $0.10 = $0.30 mỗi tháng, cộng khoảng 0.01 GB log Lambda với $0.50 mỗi GB nạp vào (khoảng $0.005), tính không có Free Tier.
- Parameter Store (tham số standard) và AWS Budgets (budget theo dõi): $0.00 mỗi tháng.
- EventBridge Scheduler tính phí theo số lần gọi, và 14 triệu lần gọi đầu tiên mỗi tháng là miễn phí. Dự án dùng khoảng 8,700 lần mỗi tháng (lịch 5 phút cộng một lần dừng mỗi ngày), tương đương khoảng $0.009 mỗi tháng nếu không tính Free Tier.
- Amazon SNS (US East, topic tiêu chuẩn): 1 triệu yêu cầu API đầu tiên và 1,000 thông báo email đầu tiên mỗi tháng là miễn phí. Dự án dùng khoảng 100 mỗi loại, nên chi phí là $0, và dưới $0.01 mỗi tháng nếu không tính Free Tier. Việc gửi tới Lambda không tính phí theo từng tin.

**Tổng:** khoảng $9.62 mỗi tháng cho các dịch vụ trong Calculator (EC2, EBS, Lambda và DynamoDB), và khoảng $10.54 khi thêm địa chỉ Public IPv4, CloudWatch, EventBridge Scheduler và SNS, tất cả tính không có Free Tier. Cả hai đều dưới ngân sách $20.

Để so sánh, cùng máy đó chạy 24/7 (730 giờ) tốn $46.22 mỗi tháng: $43.80 cho EC2, $2.40 cho EBS và khoảng $0.02 cho Lambda và DynamoDB.

Ngưỡng nội bộ của Lớp 2 đặt ở khoảng 70% ngân sách ($14). Với đơn giá $0.06 mỗi giờ, con số này tương đương khoảng 233 giờ chạy, cao hơn nhiều so với kế hoạch 120 giờ. Khoảng chênh này chừa chỗ cho độ trễ billing của Lớp 3.

**Tối ưu chi phí**

- Giới hạn giờ chạy: lịch dừng và việc bật máy thủ công giữ máy ở khoảng 120 giờ mỗi tháng thay vì 730.
- Dùng các dịch vụ tính phí theo lượt dùng và nằm trong Free Tier: Lambda, DynamoDB on-demand, SNS và EventBridge.
- Dùng tham số standard của Parameter Store, không tốn phí.
- Dùng ổ gp3 30 GB (kích thước mặc định của image Windows), không thêm IOPS hay throughput.
- So sánh các loại instance trong Pricing Calculator trước khi tạo máy và giữ loại nhỏ nhất vẫn chạy lab thoải mái.
- Tránh dùng Elastic IP, vì bị tính phí khi đang được cấp phát.
- Xóa toàn bộ tài nguyên sau khi demo và xác nhận bằng Cost Explorer.

### 7. Đánh giá rủi ro

**Ma trận rủi ro**

- Hết credit giữa chừng vì một máy bị quên chạy hoặc một lớp bảo vệ lỗi: tác động cao, xác suất trung bình.
- Ba người dùng chung một tài khoản AWS, nên một người có thể ghi đè cấu hình của người khác hoặc dừng máy đang được đồng đội dùng (ví dụ hạ ngưỡng để thử): tác động trung bình, xác suất trung bình.
- Lịch đặt sai múi giờ, nên lệnh dừng chạy vào sai giờ. Biểu thức cron của một EventBridge rule được đọc theo UTC, nên rất dễ lệch vài giờ: tác động trung bình, xác suất trung bình.
- Lịch không bao giờ kích hoạt được hàm Lambda, vì biểu thức cron viết sai hoặc hàm không cấp quyền cho EventBridge gọi nó: tác động trung bình, xác suất trung bình.
- Dữ liệu Budgets đến trễ hoặc bị hiểu sai, vì dữ liệu billing chỉ cập nhật vài lần mỗi ngày: tác động cao, xác suất cao (theo thiết kế).
- Ước tính của Lớp 2 lệch so với hóa đơn thật: tác động trung bình, xác suất trung bình.
- Một hàm Lambda dừng nhầm instance do cấu hình IAM role sai: tác động cao, xác suất thấp.

**Chiến lược giảm thiểu**

- Hết credit: Lớp 1 dừng máy mỗi ngày vào giờ cố định, Lớp 2 và Lớp 3 dừng máy khi chi phí tăng. Một lớp lỗi không làm mất hai lớp còn lại.
- Dùng chung tài khoản: mỗi người dùng IAM user riêng, chỉ một người khởi chạy instance, và mọi thay đổi ngưỡng, đơn giá hay lịch phải báo cho cả nhóm và đặt lại sau khi thử.
- Múi giờ: dùng EventBridge Scheduler cho lệnh dừng hằng ngày, vì nó cho đặt múi giờ rõ ràng (Asia/Ho_Chi_Minh) thay vì phải dựa vào UTC. Thử một lần với giờ dừng sau vài phút trước khi dựa vào nó.
- Lịch không kích hoạt: kiểm tra biểu thức cron ở phần xem trước của lịch, và xác nhận hàm Lambda cho phép EventBridge gọi nó (resource-based policy). Xem số lần gọi và log sau lần thử đầu tiên.
- Budgets trễ: chỉ coi Lớp 3 là lưới an toàn cuối, và dựa vào Lớp 2 để phản ứng nhanh.
- Ước tính lệch: đặt ngưỡng thấp hơn ngân sách và đối chiếu ước tính với Cost Explorer cuối dự án.
- Dừng nhầm máy: mỗi role Lambda chỉ có quyền dừng instance đích.

**Kế hoạch dự phòng**

- Nếu một lớp Lambda lỗi: dừng máy thủ công trên AWS Console và tắt các lịch EventBridge Scheduler.
- Nếu hết credit giữa chừng: terminate máy Windows ngay.
- Nếu ngân sách $20 không đủ: giảm số giờ lab mỗi ngày hoặc đổi sang loại instance nhỏ hơn rồi cập nhật ước tính.

### 8. Kết quả kỳ vọng

**Cải tiến kỹ thuật**

- **Thời gian phản ứng:** Từ lúc chi phí ước tính đạt ngưỡng đến lúc máy bị dừng là dưới 5 phút, tức một chu kỳ của Cost Meter.
- **Số lần dừng đúng:** Trong các bài kiểm thử, mỗi lớp dừng máy đúng mọi lần cần dừng, và không bao giờ dừng khi chi phí còn dưới ngưỡng. Workshop ghi lại số lần chạy thử và kết quả.
- **Chi tiêu:** Tổng chi tiêu dự án dưới $20, được xác nhận bằng Cost Explorer sau khi dọn dẹp.
- **Khả năng quan sát:** Có thể theo dõi cả ba lớp qua log Lambda, email SNS và bản ghi phiên trong DynamoDB.

**Giá trị dài hạn**

Vì instance ID, đơn giá theo giờ và ngưỡng đều lưu trong Parameter Store, cùng một hệ thống có thể bảo vệ một tài khoản lab Windows khác chỉ bằng cách đổi các giá trị đó, không cần sửa code.
