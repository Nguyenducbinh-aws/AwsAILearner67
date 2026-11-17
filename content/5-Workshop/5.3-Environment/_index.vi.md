---
title: "Dựng môi trường nền"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Bước 1 – Dựng môi trường nền

**Mục tiêu:** Có 2 EC2 instance Windows Server trong private subnet, đã đăng ký là SSM managed node, gắn tag `workshop=module2`.

---

### 3.1 IAM Role cho EC2 Instance (SSM + CloudWatch Agent)

Vào **IAM → Roles → Create role → AWS service → EC2**, gắn 2 policy sau:

- `AmazonSSMManagedInstanceCore`
- `CloudWatchAgentServerPolicy`

**Tên role:** `lab-windows-ssm-role`

(Console sẽ tự động tạo instance profile cùng tên.)

![Tạo IAM role](/images/5-Workshop/5.3-environment/iam-role.png)

---

### 3.2 IAM Role cho Image Builder

Tạo role thứ hai với trust policy (`ec2.amazonaws.com`) và gắn:

- `EC2InstanceProfileForImageBuilder`
- `AmazonSSMManagedInstanceCore`

**Tên role:** `lab-imagebuilder-role`

---

### 3.3 Mạng – VPC và Endpoints

**1.** Tạo VPC có 1 private subnet. Dùng **VPC Endpoints** thay NAT Gateway (rẻ hơn và giữ traffic không đi qua Internet công cộng).

Tạo các interface endpoint sau, trỏ vào security group của private subnet trên port 443:

| Endpoint | Dịch vụ |
|---|---|
| `com.amazonaws.<region>.ssm` | Systems Manager |
| `com.amazonaws.<region>.ssmmessages` | Session Manager |
| `com.amazonaws.<region>.ec2messages` | EC2 messages |
| `com.amazonaws.<region>.logs` | CloudWatch Logs |
| `com.amazonaws.<region>.monitoring` | CloudWatch Metrics |

**2.** Security group cho instance: **không có inbound rule**.  
Security group cho endpoint: cho phép HTTPS 443 inbound từ security group của instance.

![VPC và endpoints](/images/5-Workshop/5.3-environment/vpc-endpoints.png)

{{% notice tip %}}
Nếu tạm thời cần truy cập Internet (ví dụ: tải CloudWatch Agent), bạn có thể dùng public subnet cho lab ngắn, nhưng vẫn không mở cổng RDP. Chỉ truy cập qua Session Manager.
{{% /notice %}}

---

### 3.4 Khởi chạy 2 EC2 Instance Windows

Lệnh dưới tự động lấy AMI Windows Server 2022 mới nhất từ Parameter Store:

```bash
aws ec2 run-instances \
  --image-id $(aws ssm get-parameter \
     --name /aws/service/ami-windows-latest/Windows_Server-2022-English-Full-Base \
     --query Parameter.Value --output text) \
  --instance-type t3.medium --count 2 \
  --iam-instance-profile Name=lab-windows-ssm-role \
  --subnet-id <subnet-id-của-bạn> \
  --security-group-ids <sg-id-của-bạn> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=workshop,Value=module2},{Key=Name,Value=lab-win}]'
```

**Kiểm tra:** Sau vài phút, vào **Systems Manager → Fleet Manager**. Cả 2 instance phải hiện là managed node trạng thái **Online**.

Nếu không hiện: kiểm tra IAM role đã gắn chưa, endpoint tồn tại chưa, và thử reboot instance (SSM Agent đăng ký khi khởi động).

![Fleet Manager hiển thị managed node](/images/5-Workshop/5.3-environment/fleet-manager.png)

---

### 3.5 Đặt mật khẩu Administrator (để RDP qua Fleet Manager)

Kết nối qua **Session Manager**, sau đó chạy:

```powershell
$pw = -join ((48..57)+(65..90)+(97..122)+(33,35,36,37,38,42) | `
  Get-Random -Count 20 | ForEach-Object {[char]$_})
net user administrator $pw
Write-Host "Password set successfully"
```

Sau đó kết nối qua: **EC2 → Connect → RDP client → Connect using Fleet Manager**

{{% notice warning %}}
Không đưa mật khẩu vào báo cáo hay ảnh chụp màn hình.
{{% /notice %}}
