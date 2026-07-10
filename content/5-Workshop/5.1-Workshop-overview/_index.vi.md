---
title: "Tổng quan Workshop"
date: 2026-07-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục tiêu của workshop

Workshop này trình bày quá trình triển khai thực tế của **Secure Internal Job Processing Platform on AWS**, một hệ thống backend dùng để xử lý các business jobs nội bộ trong môi trường AWS private-first.

Mục tiêu chính của workshop không phải là xây dựng một website public, mà là chứng minh rằng một workload backend nội bộ có thể được triển khai trên AWS với các yêu cầu sau:

- worker và database không public ra Internet;
- job được xử lý thông qua queue thay vì gọi trực tiếp;
- failed jobs được cô lập bằng Dead Letter Queue;
- hệ thống có monitoring và alerting;
- có audit evidence phục vụ kiểm tra sau triển khai;
- toàn bộ hạ tầng được triển khai bằng Terraform;
- cấu hình hạ tầng được kiểm tra trước bằng OPA/Rego.

Nói cách khác, workshop này chứng minh quá trình chuyển một thiết kế bảo mật cloud từ proposal thành hạ tầng AWS thật, có thể kiểm chứng bằng bằng chứng triển khai và vận hành.

---

## Phạm vi triển khai thực tế

Phiên bản lab được triển khai trong workshop sử dụng cấu hình tối ưu chi phí gồm:

- 1 EC2 Worker trong private subnet;
- 1 Amazon RDS PostgreSQL Single-AZ;
- 1 Amazon SQS Processing Queue;
- 1 SQS Dead Letter Queue;
- Amazon EventBridge để tạo scheduled jobs;
- Amazon CloudWatch để ghi log, thu thập metrics và tạo alarm;
- Amazon SNS để gửi email cảnh báo;
- CloudTrail, AWS Config, VPC Flow Logs và S3 để hỗ trợ audit evidence;
- VPC Endpoints để hỗ trợ truy cập private tới các AWS services cần thiết;
- AWS Systems Manager Session Manager để quản trị EC2 thay cho SSH public.

Cấu hình này phản ánh đúng nguyên tắc của project: **triển khai thực tế đến đâu, mô tả và chứng minh đến đó**. Vì vậy, workshop không mô tả hệ thống như một kiến trúc Multi-AZ production đầy đủ, mà tập trung vào phiên bản lab đã được triển khai thật trên AWS.

Trong Terraform code và tên một số tài nguyên AWS có thể vẫn xuất hiện các tiền tố như `mvp`, `stage1` hoặc `private-by-default`. Đây là các định danh kỹ thuật được dùng trong quá trình triển khai. Tên chính thức của project trong báo cáo là:

**Secure Internal Job Processing Platform on AWS**

---

## Lý do sử dụng cấu hình 1 EC2 và 1 RDS Single-AZ

Workshop triển khai 1 EC2 Worker và 1 RDS PostgreSQL Single-AZ nhằm kiểm soát chi phí trong phạm vi thực tập, với ngân sách ước tính khoảng **85–95 USD/tháng nếu chạy 24/7**.

Việc sử dụng Single-AZ trong lab không có nghĩa là hệ thống không có khả năng mở rộng. Thiết kế vẫn giữ hai cơ chế quan trọng:

Thứ nhất, hệ thống sử dụng **Amazon SQS** để tách rời job producer và worker. EventBridge gửi job vào SQS, còn EC2 Worker chủ động poll message để xử lý. Nếu worker tạm thời dừng hoặc restart, các message chưa xử lý vẫn có thể được giữ trong queue theo cấu hình retention của SQS.

Thứ hai, toàn bộ hạ tầng được quản lý bằng **Terraform modules**. Khi cần nâng cấp lên môi trường production, có thể mở rộng thêm worker hoặc bật Multi-AZ cho RDS thông qua thay đổi cấu hình IaC thay vì thao tác thủ công trên AWS Console.

Do đó, workshop không cố chứng minh high availability bằng cách tạo nhiều tài nguyên đắt tiền trong lab. Trọng tâm của workshop là chứng minh một baseline backend an toàn, có queue bảo vệ workload, có monitoring/alerting, có audit evidence và có khả năng mở rộng bằng Infrastructure as Code.

---

## Luồng nghiệp vụ được kiểm chứng

Luồng runtime chính được kiểm chứng trong workshop là:

```text
EventBridge
→ Amazon SQS Processing Queue
→ EC2 Private Worker
→ Processing Log
→ CloudWatch / DLQ / SNS Alert