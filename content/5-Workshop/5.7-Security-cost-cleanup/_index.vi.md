---
title: "Bảo mật, tối ưu chi phí và cleanup"
date: 2026-07-04
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Security controls đã triển khai

Phần workshop triển khai nhiều lớp kiểm soát bảo mật để đảm bảo hệ thống backend nội bộ không bị expose trực tiếp ra Internet, đồng thời vẫn có khả năng quản trị, giám sát và audit sau triển khai.

Các security controls chính bao gồm:

~~~text
No public EC2 IP
No SSH key pair for EC2 administration
No public SSH/RDP ingress
AWS Systems Manager Session Manager for EC2 access
Private RDS PostgreSQL
RDS public access disabled
RDS access restricted by Security Group
RDS storage encrypted with AWS KMS
EC2 root EBS encryption
S3 evidence/log bucket encryption
VPC Endpoints for private AWS service access
SQS Processing Queue and Dead Letter Queue
CloudWatch Alarm integrated with SNS Email
CloudTrail for API activity logging
AWS Config for resource configuration tracking
VPC Flow Logs for network traffic metadata
Terraform for Infrastructure as Code
OPA/Rego policy gate before Terraform apply
~~~

Các kiểm soát này phục vụ ba mục tiêu chính:

1. **Giảm public exposure:** EC2 Worker không có public IP, không dùng SSH public, RDS không public ra Internet.
2. **Tăng khả năng vận hành an toàn:** quản trị EC2 thông qua Session Manager, workload xử lý job thông qua SQS, failed jobs được cô lập trong DLQ.
3. **Tạo audit evidence:** CloudTrail, AWS Config, VPC Flow Logs, CloudWatch và S3 hỗ trợ kiểm tra lại hoạt động và cấu hình sau triển khai.

# Cost controls

Phiên bản lab được thiết kế theo hướng kiểm soát chi phí, không triển khai quá mức so với phạm vi cần chứng minh.

Các cost controls chính bao gồm:

~~~text
One EC2 Worker for lab deployment
Single-AZ RDS PostgreSQL for cost control
No NAT Gateway
No public load balancer
VPC Endpoints used only for required AWS services
SQS and EventBridge used at low demo traffic volume
CloudWatch log retention kept short for lab usage
Terraform destroy after evidence collection
~~~

Các tài nguyên dễ phát sinh chi phí nếu để quên gồm:

- Amazon EC2;
- Amazon RDS PostgreSQL;
- VPC Interface Endpoints;
- AWS KMS;
- CloudWatch Logs, Metrics và Alarms;
- AWS Config;
- VPC Flow Logs;
- S3 storage.

Vì project sử dụng các tài nguyên tính phí theo giờ như EC2, RDS và VPC Interface Endpoints, cleanup là bước bắt buộc sau khi hoàn tất evidence.

Ước tính chi phí cho cấu hình lab nếu chạy liên tục 24/7 tại `us-east-1` là khoảng **85–95 USD/tháng**. Trong thực tế demo, chi phí có thể thấp hơn vì hạ tầng chỉ được triển khai trong thời gian ngắn để kiểm thử, chụp evidence và cleanup sau đó.

# Cleanup

Sau khi đã thu thập đủ evidence cho báo cáo, cần xóa toàn bộ hạ tầng bằng Terraform để tránh phát sinh chi phí không cần thiết.

Chạy trong Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
aws sso login --profile mvp --no-browser
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform plan -destroy -out destroy.binary
terraform apply destroy.binary
~~~

Khi Terraform hỏi xác nhận, nhập:

~~~text
yes
~~~

Kết quả đúng cần thấy:

~~~text
Destroy complete!
~~~

# Kiểm tra sau cleanup

Sau khi Terraform destroy hoàn tất, cần mở AWS Console và kiểm tra lại các nhóm tài nguyên chính đã được xóa.

Các tài nguyên cần kiểm tra gồm:

- EC2 Worker;
- RDS PostgreSQL database;
- SQS Processing Queue;
- SQS Dead Letter Queue;
- EventBridge schedule;
- SNS topic và email subscription;
- CloudWatch Alarms;
- VPC Endpoints;
- S3 evidence/log bucket;
- CloudTrail;
- AWS Config;
- VPC Flow Logs.

Một số tài nguyên như KMS keys có thể không bị xóa ngay lập tức mà chuyển sang trạng thái scheduled deletion theo deletion window đã cấu hình. Đây là hành vi bình thường của AWS KMS.

Nếu sau cleanup vẫn thấy chi phí phát sinh trong Billing Dashboard, cần kiểm tra lại các dịch vụ thường bị sót như RDS snapshots, NAT Gateway, VPC Endpoints, CloudWatch Logs, AWS Config Recorder, S3 bucket hoặc KMS keys.

# Kết luận chương Workshop

Phần Workshop đã trình bày toàn bộ vòng đời triển khai của **Secure Internal Job Processing Platform on AWS**.

Quá trình triển khai bao gồm:

1. chuẩn bị môi trường local và quyền AWS;
2. xây dựng hạ tầng bằng Terraform;
3. kiểm tra Terraform plan bằng OPA/Rego;
4. triển khai workload trong môi trường AWS private-first;
5. kiểm chứng luồng xử lý job thông qua EventBridge, Amazon SQS và EC2 Worker;
6. giám sát vận hành bằng CloudWatch và SNS;
7. thu thập evidence về bảo mật, vận hành và audit;
8. cleanup tài nguyên AWS để kiểm soát chi phí.

Kết quả triển khai chứng minh rằng project không chỉ dừng ở sơ đồ kiến trúc. Hệ thống đã được triển khai thực tế trên AWS, có worker và database không public ra Internet, có queue để xử lý job bất đồng bộ, có DLQ để cô lập failed jobs, có CloudWatch/SNS để cảnh báo, có CloudTrail/AWS Config/VPC Flow Logs để hỗ trợ audit và có Terraform/OPA để kiểm soát hạ tầng bằng code.

Nói cách khác, workshop chứng minh rằng một workload backend nội bộ trên AWS có thể được triển khai theo hướng an toàn, kiểm chứng được và có khả năng cleanup rõ ràng sau khi hoàn tất demo.