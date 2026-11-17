---
title: "Bảo mật – Config & Inspector"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Bước 5 – Bảo mật (AWS Config & Amazon Inspector)

**Mục tiêu:** Kiểm soát AMI được duyệt và tag bắt buộc bằng AWS Config, phát hiện lỗ hổng CVE bằng Amazon Inspector, và xác nhận truy cập instance an toàn qua Session Manager và IMDSv2.

---

### 7.1 Hardened Image với DISA STIG

Tạo recipe thứ hai để build AMI được hardened theo STIG:

**EC2 Image Builder → Image recipes → Create image recipe**

| Trường | Giá trị |
|---|---|
| Tên | `Secure Image` |
| Version | `1.0.0` |
| Base image | Windows Server 2019 English Full Base (hoặc 2022) |
| Build component | Amazon managed → `stig-build-windows` |

Tạo pipeline tên `Windows-STIG-Pipeline`, chạy pipeline, rồi copy AMI ID từ version mới nhất (ví dụ `1.0.0/1`).

Khởi chạy instance mới từ AMI STIG này, gắn tag:
- `Workshop=SecureWindows`
- `Image=STIGImage`

Gắn SSM IAM profile. Không mở RDP.

{{% notice warning %}}
Dùng STIG component không đảm bảo qua được kiểm toán chính thức. Hãy ghi rõ điều này trong báo cáo.
{{% /notice %}}

![Kết quả STIG pipeline](/images/5-Workshop/5.7-security/stig-pipeline.png)

---

### 7.2 AWS Config – Giới hạn AMI được duyệt

**1.** Vào **AWS Config → 1-click setup → Confirm**

**2.** Vào **Rules → Add rule → AWS managed rule → `approved-amis-by-id`**  
Tên: `infosec-approved-amis`  
Trigger: giữ mặc định (EC2 Instances)  
Parameter: nhập AMI ID STIG từ bước trước.

**3.** Cài đặt thông báo qua EventBridge:
- Tạo SNS topic: `SecureWindowsNotifications` + email subscription
- Tạo EventBridge rule bắt sự kiện **Config rule compliance change** cho `infosec-approved-amis`
- Target: SNS topic đó

(EventBridge cung cấp thông báo chi tiết hơn với Instance ID, so với thông báo mặc định của Config.)

Khởi chạy một instance **không được duyệt** để kích hoạt NON_COMPLIANT và xác nhận nhận được email.

![Config NON_COMPLIANT finding](/images/5-Workshop/5.7-security/config-noncompliant.png)

---

### 7.3 AWS Config – Kiểm soát Tag bắt buộc

Thêm rule managed: **`required-tags`**

Cài đặt để kiểm tra tag `Workshop = SecureWindows`.  
Dùng lại SNS topic `SecureWindowsNotifications`.

---

### 7.4 Amazon Inspector

**1.** Vào **Amazon Inspector → Get started → Activate Amazon Inspector**  
(Tài khoản đơn; không cần delegate admin cho lab này.)

{{% notice tip %}}
Inspector quét EC2 instance thông qua SSM Agent. Hãy chắc chắn agent đang chạy trên các instance của bạn.
{{% /notice %}}

**2.** Xem **Findings** để thấy các lỗ hổng CVE được phát hiện trên instance.

**3.** Dùng **Patch Manager** (từ Bước 3.5) để áp dụng patch cấp OS.  
Lưu ý: Patch Manager chỉ cập nhật patch OS — không nâng cấp phiên bản OS lớn.

![Inspector findings](/images/5-Workshop/5.7-security/inspector-findings.png)

---

### 7.5 Truy cập Instance An toàn

**Session Manager – kiểm tra IMDSv2:**

Kết nối vào instance qua Session Manager và chạy:

```powershell
[string]$token = Invoke-RestMethod `
  -Headers @{"X-aws-ec2-metadata-token-ttl-seconds"="21600"} `
  -Method PUT `
  -Uri http://169.254.169.254/latest/api/token

Invoke-RestMethod `
  -Headers @{"X-aws-ec2-metadata-token"=$token} `
  -Method GET `
  -Uri http://169.254.169.254/latest/meta-data/instance-id
```

Điều này xác nhận IMDSv2 đang hoạt động (dùng token, an toàn hơn IMDSv1).

**Fleet Manager RDP:** Kết nối qua **EC2 → Connect → RDP client → Connect using Fleet Manager** — không cần mở port 3389.

![Terminal Session Manager](/images/5-Workshop/5.7-security/session-manager.png)

![Fleet Manager RDP](/images/5-Workshop/5.7-security/fleet-manager-rdp.png)

---

### Tổng kết – Các biện pháp bảo mật đã áp dụng

| Biện pháp | Cài đặt |
|---|---|
| Không mở cổng inbound | Security group không có inbound rule |
| Không expose RDP | Chỉ truy cập qua Session Manager / Fleet Manager |
| Bắt buộc IMDSv2 | Yêu cầu token để lấy metadata |
| Chính sách AMI được duyệt | AWS Config rule `approved-amis-by-id` |
| Tag bắt buộc | AWS Config rule `required-tags` |
| Lỗ hổng OS | Amazon Inspector (quét CVE) + Patch Manager |
| Hardened image | STIG build component trong Image Builder |
| Least privilege | Role riêng biệt chỉ gắn policy cần thiết |
