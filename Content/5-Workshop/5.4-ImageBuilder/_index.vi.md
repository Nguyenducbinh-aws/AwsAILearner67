---
title: "EC2 Image Builder – Golden AMI"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Bước 2 – EC2 Image Builder (Golden AMI)

**Mục tiêu:** Tạo golden AMI từ Windows Server 2025 với CloudWatch Agent được cài sẵn, có phiên bản và được quản lý bởi lifecycle policy.

---

### Image Builder hoạt động như thế nào

Image Builder chạy pipeline qua 3 giai đoạn:

1. **Build** — khởi chạy EC2 tạm thời, cài component qua SSM Run Command
2. **Test** — khởi chạy instance thứ hai từ AMI mới để chạy test component
3. **Distribute** — đưa AMI vào tài khoản (và tùy chọn các region khác)

![Luồng pipeline Image Builder](/images/5-Workshop/5.4-imagebuilder/pipeline-flow.png)

---

### 4.1 Tạo Recipe

Vào **EC2 Image Builder → Image recipes → Create image recipe**

| Trường | Giá trị |
|---|---|
| Image type | Amazon Machine Image (AMI) |
| Name | `my-first-recipe` |
| Version | `1.0.0` |
| Base image | Managed images → Windows → Windows Server 2025 English Full Base x86 → **Use latest available OS version** |
| Build components | Amazon managed → `amazon-cloudwatch-agent-windows` (latest) |
| Test components | Amazon managed → `simple-boot-test-windows` |

{{% notice tip %}}
Recipe không thể sửa sau khi tạo. Muốn thay đổi component phải tạo version mới (1.0.1, 1.0.2…).
{{% /notice %}}

![Tạo recipe](/images/5-Workshop/5.4-imagebuilder/recipe.png)

---

### 4.2 Tạo Pipeline

**1.** Vào **Image pipelines → Create image pipeline**  
Tên: `my-first-pipeline`  
Bật: **Enhanced metadata collection** và **Enable EC2 image scanning**

**2.** Build schedule: giữ mặc định. Với lab, chọn **manual** để tránh chi phí phát sinh.

**3.** Chọn recipe vừa tạo. Image creation process: **Default workflows**.

**4.** Infrastructure configuration:
- Tên: `workshop-infra-config`
- IAM role: `lab-imagebuilder-role`
- Instance type: `m7i-flex.large` (hoặc `m5.large` nếu không có)
- VPC, subnet, security group: dùng từ Bước 1

**5.** Distribution: **Create distribution settings using service defaults** (AMI riêng tư trong tài khoản).

**6.** Review → **Create pipeline**

**7.** Image pipelines → chọn pipeline → **Actions → Run pipeline**

Theo dõi trạng thái: `Pending → Building → Testing → Available`  
Mất khoảng **15–30 phút**. Instance build tạm thời sẽ xuất hiện trong EC2.

![Pipeline đang chạy](/images/5-Workshop/5.4-imagebuilder/pipeline-running.png)

![AMI trạng thái Available](/images/5-Workshop/5.4-imagebuilder/ami-available.png)

---

### 4.3 Lifecycle Policy

**EC2 Image Builder → Lifecycle policies → Create lifecycle policy**

| Trường | Giá trị |
|---|---|
| Resource type | AMI |
| Tên | `my-ib-lifecycle-policy` |
| IAM role | `lab-imagebuilder-role` |
| Rule scope | Recipes → `my-first-recipe` |
| Lifecycle rule | Disable sau 6 tháng |

Điều này ngăn AMI cũ và EBS snapshot tích lũy không cần thiết.

---

### 4.4 Khám phá bên trong pipeline (rất tốt cho phần học thuật của báo cáo)

**Component document:**  
EC2 Image Builder → Components → `amazon-cloudwatch-agent-windows`  
Xem cấu trúc: `phases → steps → action (ExecutePowerShell…)`

**Log build:**  
CloudWatch → Log groups → `/aws/imagebuilder/my-first-recipe*`  
Lọc từ khóa: `Step`, `Std`, `CmdExecution`

**SSM Run Command history:**  
Systems Manager → Run Command → Command history  
Image Builder dùng Run Command nội bộ để cài component.

![Cấu trúc component document](/images/5-Workshop/5.4-imagebuilder/component-doc.png)

![Build log trong CloudWatch](/images/5-Workshop/5.4-imagebuilder/build-logs.png)

---

### 4.5 Tự viết Component (Nâng cao)

Trên instance Windows (qua Fleet Manager), cài VS Code và tải `awstoe.exe` từ tài liệu AWS. Tạo `my-first-doc.yml`:

```yaml
name: Hello World
description: Hello world testing document for Windows.
schemaVersion: 1.0
phases:
  - name: 'build'
    steps:
      - name: SamplePowerShellStep
        action: ExecutePowerShell
        timeoutSeconds: 120
        onFailure: Abort
        maxAttempts: 3
        inputs:
          commands:
            - 'Write-Host "Hello world!"'
```

Validate và chạy thử:

```powershell
.\awstoe.exe validate -d .\my-first-doc.yml
.\awstoe.exe run -d .\my-first-doc.yml -t
```

**Test IIS bằng Pester** — chạy trước khi cài IIS (phải fail), rồi sau khi cài (phải pass):

```powershell
Describe "IIS validation test" {
    BeforeAll { $httpResponse = Invoke-WebRequest localhost }
    It "HTTP Headers should have key: server, value: Microsoft-IIS" {
        $httpResponse.Headers['server'] | Should -Match 'Microsoft-IIS'
    }
    It "IIS should return HTTP 200 OK" {
        $httpResponse.StatusCode | Should -BeExactly 200
    }
}
```

Cài IIS:

```powershell
Install-WindowsFeature -Name web-server
```

Rồi chạy lại test Pester — cả hai test phải pass.
