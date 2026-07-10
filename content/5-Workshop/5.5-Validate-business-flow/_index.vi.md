---
title: "Kết quả triển khai và xác thực hệ thống"
date: 2026-07-04
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---


Sau khi triển khai hạ tầng bằng Terraform và xác thực bằng các chính sách OPA, hệ thống đã tạo thành công các tài nguyên sau:

- 1 VPC riêng tư
- 1 subnet ứng dụng riêng tư
- 2 subnet cơ sở dữ liệu riêng tư
- 1 EC2 Worker không có Public IP
- 1 PostgreSQL RDS chạy trong private subnet
- Các KMS Key do khách hàng quản lý cho dữ liệu ứng dụng và log
- S3 Bucket lưu trữ log được mã hóa bằng KMS
- AWS CloudTrail được kích hoạt
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

## Xác thực yêu cầu bảo mật

Môi trường triển khai đáp ứng đầy đủ các yêu cầu cốt lõi của mô hình Private-by-Default:

- Không triển khai Internet Gateway.
- Không triển khai NAT Gateway.
- Không sử dụng Bastion Host.
- Không có tài nguyên workload nào được gán Public IP.
- Việc quản trị hệ thống được thực hiện thông qua AWS Systems Manager Session Manager.
- Dữ liệu lưu trữ được mã hóa bằng các AWS KMS Key do khách hàng quản lý.
- Hoạt động ghi nhận và kiểm toán được thực hiện thông qua CloudTrail, AWS Config và VPC Flow Logs.
- Chính sách OPA được sử dụng để kiểm tra tính tuân thủ trước khi triển khai.

## Kết quả đạt được

Kết quả triển khai cho thấy một hệ thống xử lý workload trên AWS có thể vận hành hoàn toàn trong môi trường mạng riêng tư, đồng thời vẫn đảm bảo khả năng quản trị, giám sát, ghi log, cảnh báo và kiểm soát bảo mật bằng các dịch vụ quản lý của AWS.