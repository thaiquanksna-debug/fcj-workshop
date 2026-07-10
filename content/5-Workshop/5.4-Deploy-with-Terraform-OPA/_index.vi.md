---
title: "Deploy bằng Terraform và OPA"
date: 2026-07-04
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Mục tiêu

Phần này deploy toàn bộ infrastructure. Chỉ apply khi OPA trả về deny list rỗng: `[]`.

# 1. Login AWS SSO

Chạy trong Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
~~~

Kết quả đúng: output có AWS account ID.

# 2. Initialize và validate Terraform

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform init
terraform fmt -recursive ..\..\modules
terraform fmt -recursive .
terraform validate
~~~

Expected result:

~~~text
Success! The configuration is valid.
~~~

# 3. Tạo Terraform plan

~~~powershell
terraform plan -out tfplan.binary
~~~

Terraform phải hiển thị các resource sẽ tạo, gồm VPC, private subnets, security groups, VPC endpoints, KMS, S3, CloudTrail, AWS Config, VPC Flow Logs, RDS, EC2 Worker, SQS, DLQ, EventBridge Scheduler, CloudWatch Alarms, and SNS.

# 4. Export plan sang JSON

~~~powershell
cd D:\mvp_private_by_default_architecture
New-Item -ItemType Directory -Force evidence\m9 | Out-Null
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform show -json tfplan.binary > ..\..\..\evidence\m9\final-plan.json
~~~

# 5. Chạy OPA/Rego policy gate

~~~powershell
cd D:\mvp_private_by_default_architecture
opa eval --format pretty --data policy\terraform --input evidence\m9\final-plan.json "data.terraform.deny" > evidence\m9\final-opa.txt
Get-Content evidence\m9\final-opa.txt
~~~

Kết quả đúng:

~~~text
[]
~~~

# 6. Apply Terraform

Chỉ chạy nếu OPA output là `[]`:

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform apply tfplan.binary
~~~

Khi Terraform hỏi confirm, gõ:

~~~text
yes
~~~

RDS có thể mất 7–20 phút. Đợi tới khi Terraform in ra:

~~~text
Apply complete!
~~~

# 7. Confirm SNS email subscription

Mở email đã cấu hình trong `terraform.tfvars`. Tìm email có tiêu đề gần giống:

~~~text
AWS Notification - Subscription Confirmation
~~~

Bấm **Confirm subscription**. Nếu chưa confirm, CloudWatch Alarm sẽ chưa gửi được SNS Email.

# 8. Lưu Terraform evidence

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform output > ..\..\..\evidence\m9\terraform-output.txt
terraform output -json > ..\..\..\evidence\m9\terraform-output.json
terraform state list > ..\..\..\evidence\m9\terraform-state.txt
~~~

# 9. Kiểm tra Session Manager đã thấy EC2 chưa

~~~powershell
aws ssm describe-instance-information --profile mvp --region us-east-1
~~~

Kết quả đúng: output có EC2 instance ID từ `terraform output`.

Nếu list rỗng, đợi 3–5 phút và chạy lại. Nếu vẫn rỗng, reboot worker:

~~~powershell
$InstanceId = terraform output -raw worker_instance_id
aws ec2 reboot-instances --profile mvp --region us-east-1 --instance-ids $InstanceId
Start-Sleep -Seconds 180
aws ssm describe-instance-information --profile mvp --region us-east-1
~~~

# Lỗi thường gặp và cách sửa đúng

## SSO token hết hạn

Dấu hiệu:

```text
No valid credential sources found
InvalidGrantException
```

Cách sửa:

```powershell
cd D:\mvp_private_by_default_architecture
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
```

## Saved plan is stale

Dấu hiệu:

```text
Error: Saved plan is stale
```

Cách sửa: tạo plan mới, export JSON lại, chạy OPA lại, rồi apply plan mới. Không dùng lại plan cũ sau khi Terraform state đã thay đổi.

## AccessDenied

Dấu hiệu: Terraform báo `not authorized to perform ...`.

Cách sửa: quay lại phần 5.2 và kiểm tra đã paste full inline policy vào permission set `PrivateWorkloadTerraformOperator`, sau đó chạy `aws sso logout` and `aws sso login --profile mvp --no-browser` again.

# Tóm tắt kết quả triển khai

Sau khi triển khai thành công, hệ thống sẽ bao gồm:

- 1 VPC riêng tư
- 1 subnet ứng dụng riêng tư
- 2 subnet cơ sở dữ liệu riêng tư
- 1 EC2 Worker không có Public IP
- 1 PostgreSQL RDS chạy trong private subnet
- Các KMS Key do khách hàng quản lý cho dữ liệu ứng dụng và log
- S3 Bucket lưu trữ log được mã hóa bằng KMS
- CloudTrail được kích hoạt
- AWS Config được kích hoạt
- VPC Flow Logs được kích hoạt
- Các Interface VPC Endpoint:
  - KMS
  - STS
  - CloudWatch Logs
  - SSM
  - SSMMessages
  - EC2Messages
  - CloudWatch Monitoring
  - SQS
- Gateway VPC Endpoint:
  - S3
- Hàng đợi xử lý SQS
- Dead Letter Queue (DLQ)
- SNS Alert Topic
- EventBridge Scheduler
- CloudWatch Alarms
- Kết nối Session Manager

Việc triển khai thành công chứng minh rằng nền tảng có thể vận hành hoàn toàn trong môi trường private-by-default mà không cần Internet Gateway, NAT Gateway, bastion host hoặc Public IP, đồng thời vẫn duy trì khả năng quản trị và vận hành thông qua AWS Systems Manager và các AWS Private Endpoint.