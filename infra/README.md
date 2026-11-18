# Cost Guard – triển khai nhanh bằng CloudFormation

Template [cost-guard.yaml](cost-guard.yaml) tạo toàn bộ hệ thống trong **một stack** ở region **us-east-1**:
máy Windows t3.medium (ổ gp3 30 GB), Parameter Store, bảng DynamoDB `CostGuardSessions`, 3 SNS topic,
3 hàm Lambda với 3 IAM role riêng, 2 lịch EventBridge Scheduler, AWS Budget và 3 CloudWatch Alarm.
Mã Lambda nằm ngay trong template nên không cần upload S3.

> **Chưa kiểm chứng:** template chưa được chạy `cfn-lint` hay deploy thử. Hãy chạy
> `aws cloudformation validate-template` (hoặc để Console báo lỗi) ở lần đầu, và sửa nếu cần.

## 1. Chuẩn bị (5 phút)
- Đăng nhập AWS Console bằng IAM user (không dùng root), chọn region **US East (N. Virginia)**.
- Chuẩn bị một email nhận cảnh báo.
- Kiểm tra tài khoản có **default VPC** (nếu không, tạo máy sẽ lỗi; khi đó truyền `ExistingInstanceId`).

## 2. Deploy
**Cách A – Console:** CloudFormation → Create stack → Upload `cost-guard.yaml` → đặt tên `CostGuard`
→ nhập `AlertEmail` → các tham số khác giữ mặc định → tick "I acknowledge that AWS CloudFormation might create IAM resources" → Create.

**Cách B – CLI:**
```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name CostGuard \
  --template-file infra/cost-guard.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides AlertEmail=you@example.com
```

Sau khi stack xong:
1. Mở email, bấm **Confirm subscription** cho 2 thư SNS (alerts và errors).
2. Máy EC2 **đang chạy ngay khi tạo xong** và tính tiền $0.06 mỗi giờ. Đừng để quá lâu trước khi test.

Tham số hay chỉnh:
- `StopSchedule`, `StopScheduleTimezone`: giờ dừng hằng ngày (mặc định 22:00, Asia/Ho_Chi_Minh).
- `CostThreshold` (mặc định 14), `BudgetLimit` (mặc định 20), `PricePerHour` (mặc định 0.06).
- `ExistingInstanceId`: để trống để tạo máy mới, hoặc nhập ID máy có sẵn.

## 3. Kiểm thử từng lớp (dùng khi quay demo)
Ghi lại ảnh chụp mỗi bước (che account ID).

**Lớp 1 – lịch dừng:**
1. Lambda → `CostGuard-ScheduleHandler` → Test → event `{}` → Test.
2. Kỳ vọng: EC2 chuyển `running` → `stopping` → `stopped`; có bản ghi `schedule_stop` trong DynamoDB; xem log trong CloudWatch.
3. Bật lại máy (EC2 → Start) để test lớp tiếp theo.

**Lớp 2 – bộ đo chi phí:**
1. Parameter Store → `/costguard/cost-threshold` → đổi giá trị thành `0.001`.
2. Chờ tối đa 5 phút. Kỳ vọng: máy bị dừng, có email "instance stopped (cost threshold)", có bản ghi `sample` và `auto_stop` trong DynamoDB.
3. **Đặt lại** `/costguard/cost-threshold` về `14`. Lưu ý: các bản ghi `sample` của lúc test vẫn cộng vào tổng tháng (rất nhỏ).
4. Ghi lại thời gian từ lúc đổi ngưỡng tới lúc máy dừng (mục tiêu dưới 5 phút).

**Lớp 3 – Budget Backstop:**
1. Bật lại máy nếu đang tắt.
2. SNS → topic `costguard-budgets` → Publish message → nội dung bất kỳ → Publish.
3. Kỳ vọng: máy bị dừng, có email xác nhận, có bản ghi `budget_stop`.

**Kịch bản lỗi (cảnh báo lỗi):**
1. Parameter Store → `/costguard/instance-id` → đổi tạm thành `i-00000000000000000`.
2. Chờ tối đa 5–10 phút. Kỳ vọng: hàm `CostGuard-CostMeter` lỗi, có email từ topic `costguard-errors` và alarm `CostGuard-CostMeter-Errors` chuyển sang ALARM.
3. **Đặt lại** `/costguard/instance-id` về đúng ID máy (xem tab Outputs của stack, mục `ProtectedInstanceId`).

## 4. Cách tính chi phí của Lớp 2 (để ghi vào báo cáo)
- Mỗi lần chạy (5 phút) khi máy đang `running`, hàm ghi một bản ghi `sample` có chi phí = đơn giá × 5/60.
- Tổng tháng = cộng các bản ghi `sample` của tháng dương lịch hiện tại (theo UTC).
- Nếu tổng ≥ ngưỡng thì dừng máy, ghi `auto_stop` và gửi email. Máy bật lại sẽ bị dừng ở lần chạy kế tiếp cho đến tháng sau (đúng quy tắc "khóa đến đầu tháng sau" trong Proposal).
- Hạn chế: một phiên chạy ngắn hơn 5 phút có thể không được tính; chu kỳ tháng theo UTC lệch 7 giờ so với giờ Việt Nam.

## 5. Dọn dẹp (bắt buộc sau khi demo)
1. CloudFormation → stack `CostGuard` → **Delete**. Stack xóa cả máy EC2 và ổ EBS, bảng DynamoDB, tham số, Lambda, log group, Budget, SNS và alarm.
2. Kiểm tra lại: EC2 (instance và Volumes), Elastic IPs, Lambda, DynamoDB, SNS, CloudWatch Alarms và Log groups, Budgets, Parameter Store đều đã hết.
3. Sau 1–2 ngày mở **Cost Explorer**, chụp ảnh chi phí và đối chiếu với ước tính trong Proposal.
