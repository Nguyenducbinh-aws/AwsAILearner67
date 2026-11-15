---
title: "Yêu cầu tiên quyết"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Yêu cầu tiên quyết

### Chuẩn bị tài khoản AWS

1. Dùng **tài khoản AWS riêng** cho lab này. Không dùng tài khoản chung hoặc production.
2. **Không dùng root user.** Tạo IAM user hoặc IAM Identity Center user có quyền administrator cho lab, bật MFA cho cả root và user lab.
3. **Đặt Budget trước khi tạo bất kỳ tài nguyên nào.**

```
Billing → Budgets → Create budget → Cost budget (Monthly)
Số tiền: $20 USD
Cảnh báo: 50% và 80%
```

{{% notice warning %}}
Hạ tầng đầy đủ của workshop (NAT Gateway, Managed AD, nhiều instance) tốn khoảng **$3–5 USD/giờ**. Không để chạy qua đêm.
{{% /notice %}}

### Công cụ trên máy tính

Cài đặt các công cụ sau:

| Công cụ | Mục đích | Cài đặt |
|---|---|---|
| AWS CLI v2 | Chạy lệnh AWS | [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) |
| Git | Quản lý phiên bản | [git-scm.com](https://git-scm.com) |
| VS Code | Viết code, component | [code.visualstudio.com](https://code.visualstudio.com) |
| Hugo Extended | Xem trước báo cáo tại máy | [gohugo.io](https://gohugo.io/installation/) |
| Session Manager plugin | Kết nối SSM từ CLI | [docs.aws.amazon.com/systems-manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) |

Kiểm tra AWS CLI hoạt động:

```bash
aws --version
aws configure          # dùng key của IAM user lab — không commit lên GitHub
aws sts get-caller-identity
```

{{% notice warning %}}
Tuyệt đối không hard-code Access Key hoặc mật khẩu trong code hoặc trong repository GitHub. Thêm `.env` và file credentials vào `.gitignore`.
{{% /notice %}}

### Region

Chọn một region và không thay đổi trong suốt quá trình làm workshop. Khuyến nghị: **ap-southeast-1** hoặc **us-east-1**.

```bash
aws configure set region ap-southeast-1
```

### Ảnh minh chứng

![Cài đặt Budget](/images/5-Workshop/5.2-prerequisite/budget.png)

![IAM user với MFA](/images/5-Workshop/5.2-prerequisite/iam-mfa.png)
