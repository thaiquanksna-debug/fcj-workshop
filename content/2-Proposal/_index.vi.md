---
title: "Bản đề xuất"
date: 2026-07-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
## Nền tảng xử lý job nội bộ an toàn trên AWS

### 1. Tóm tắt điều hành

Secure Internal Job Processing Platform on AWS là một hệ thống backend được thiết kế cho các doanh nghiệp cần xử lý các tác vụ dữ liệu nội bộ trong môi trường cloud an toàn, không expose worker hoặc database trực tiếp ra Internet.

Project không phải là một website public có giao diện người dùng, mà là một hệ thống xử lý job nội bộ đã được triển khai trên AWS. Hệ thống mô phỏng các workload như đối soát giao dịch, validate dữ liệu khách hàng, xử lý batch logs, tạo bằng chứng audit hoặc chạy các tác vụ compliance định kỳ.

Kiến trúc sử dụng Amazon VPC để cô lập mạng, Amazon EC2 làm worker xử lý job, Amazon RDS PostgreSQL làm tầng database, Amazon SQS làm hàng đợi bất đồng bộ, Amazon EventBridge để tạo job định kỳ, Amazon CloudWatch và Amazon SNS để giám sát/cảnh báo, Amazon S3 để lưu trữ dữ liệu/log/evidence, cùng các dịch vụ bảo mật và governance như IAM, KMS, AWS Config, CloudTrail và Systems Manager.

Hệ thống được thiết kế và triển khai theo mô hình tối ưu hóa chi phí (FinOps) với một EC2 Worker và một Amazon RDS PostgreSQL cấu hình Single-AZ. Kiến trúc này không chỉ phản ánh chính xác hạ tầng thực tế được khởi tạo, mà vẫn đảm bảo toàn vẹn các mục tiêu chiến lược bao gồm: mạng nội bộ cô lập (private networking), xử lý bất đồng bộ qua hàng đợi (queue-based processing), giám sát và cảnh báo chủ động (monitoring & alerting), lưu vết kiểm toán (audit evidence), và quản lý tự động hóa hoàn toàn bằng mã nguồn (Infrastructure as Code).

---

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Nhiều doanh nghiệp có các tác vụ xử lý dữ liệu nội bộ không phù hợp để expose ra Internet, ví dụ:

- đối soát dữ liệu giao dịch;
- validate file CSV khách hàng;
- xử lý logs nghiệp vụ;
- tạo báo cáo kiểm toán;
- chạy tác vụ compliance định kỳ;
- đồng bộ hoặc kiểm tra dữ liệu giữa các hệ thống nội bộ.

Nếu các workload này được triển khai thủ công hoặc đặt trong môi trường public quá dễ dãi, hệ thống có thể gặp các rủi ro sau:

- EC2 hoặc database bị expose ra Internet;
- thiếu cơ chế queue để giữ job khi worker lỗi;
- khó retry hoặc điều tra failed jobs;
- thiếu alert khi hệ thống bất thường;
- thiếu audit evidence để chứng minh cấu hình bảo mật;
- thao tác hạ tầng thủ công gây drift và khó tái triển khai;
- chi phí cloud khó kiểm soát nếu dùng NAT Gateway hoặc tài nguyên dư thừa.

Vấn đề chính của project là: **làm thế nào để biến các yêu cầu bảo mật cloud thành hạ tầng AWS thực tế, có thể triển khai, kiểm chứng và audit được.**

#### Giải pháp đề xuất

Project đề xuất một nền tảng xử lý job nội bộ trên AWS theo hướng private-first. Thay vì nhận request từ public HTTP API, hệ thống nhận công việc thông qua Amazon SQS. Amazon EventBridge tạo job định kỳ và gửi vào SQS. EC2 Worker nằm trong private subnet sẽ chủ động poll message từ SQS để xử lý. Khi cần lưu dữ liệu hoặc metadata, worker kết nối tới Amazon RDS PostgreSQL thông qua Security Group nội bộ.

Nếu job xử lý thất bại nhiều lần, message được chuyển sang Dead Letter Queue để điều tra hoặc xử lý lại. CloudWatch theo dõi logs/metrics và kích hoạt alarm khi có bất thường. SNS gửi email cảnh báo cho Ops/Admin. CloudTrail, AWS Config, VPC Flow Logs và S3 hỗ trợ audit evidence sau triển khai.

Toàn bộ hạ tầng được định nghĩa bằng Terraform. Trước khi triển khai, Terraform plan được kiểm tra bằng OPA/Rego policy gate để hạn chế các cấu hình không an toàn.

#### Lợi ích và giá trị

Hệ thống mang lại các giá trị chính:

- giảm rủi ro public exposure cho worker và database;
- xử lý job bất đồng bộ bằng SQS;
- giảm nguy cơ mất job khi worker tạm dừng hoặc lỗi;
- hỗ trợ failure handling thông qua Dead Letter Queue;
- có monitoring và alerting qua CloudWatch/SNS;
- tạo audit evidence từ CloudTrail, AWS Config, VPC Flow Logs và S3;
- kiểm soát hạ tầng bằng Terraform thay vì thao tác thủ công;
- triển khai lab tiết kiệm chi phí nhưng vẫn có khả năng mở rộng bằng IaC.

Với mức chi phí ước tính khoảng 85–95 USD/tháng nếu chạy 24/7, nền tảng này không chỉ là một demo AWS service riêng lẻ, mà là một baseline có thể tái sử dụng cho nhiều loại internal jobs cần private networking, monitoring, alerting và audit evidence.

---

### 3. Kiến trúc giải pháp

![Secure Internal Job Processing Platform Architecture](/images/2-Proposal/secure-internal-job-processing-architecture.png)

**Hình:** Kiến trúc tổng quan của Secure Internal Job Processing Platform on AWS.

Kiến trúc triển khai thực tế trong lab gồm một EC2 Worker và một RDS PostgreSQL Single-AZ. Về mặt hạ tầng, hệ thống được triển khai theo mô hình tối giản (Lean Architecture) bao gồm một EC2 Worker và một Amazon RDS PostgreSQL cấu hình Single-AZ nhằm tối ưu hóa chi phí vận hành (FinOps) trong giai đoạn thử nghiệm (Pilot/Demo Deployment). Sơ đồ kiến trúc phản ánh chính xác 100% tài nguyên thực tế được khởi tạo.

Mặc dù vận hành trên cấu hình Single-AZ, hệ thống vẫn đảm bảo tính toàn vẹn của dữ liệu và khả năng mở rộng linh hoạt nhờ vào hai cơ chế cốt lõi sau:

- **Decoupling bằng SQS:** EventBridge gửi job vào SQS, còn EC2 Worker chủ động poll job. Nếu worker tạm dừng hoặc restart, job chưa xử lý vẫn có thể được giữ trong SQS theo thời gian retention.
- **Mở rộng bằng Terraform:** Hạ tầng được module hóa bằng Terraform. Khi cần nâng cấp, có thể bổ sung worker hoặc bật Multi-AZ cho RDS thông qua thay đổi cấu hình IaC thay vì thao tác thủ công trên AWS Console.

Nói cách khác, project không cố chứng minh high availability bằng cách tạo nhiều tài nguyên đắt tiền trong lab. Project chứng minh cách thiết kế một nền tảng xử lý job nội bộ có thể chạy tiết kiệm, có queue bảo vệ workload, có monitoring/alerting và có IaC để mở rộng khi cần.

---

### 4. Dịch vụ AWS sử dụng

| Dịch vụ / Thành phần | Vai trò trong project |
|---|---|
| Amazon VPC | Tạo mạng riêng cho hệ thống, đặt EC2 và RDS trong private subnets, hạn chế public exposure. |
| Amazon EC2 | Đóng vai trò Worker xử lý các internal batch jobs từ SQS. |
| Amazon RDS PostgreSQL | Lưu dữ liệu hoặc metadata xử lý nghiệp vụ trong tầng database private. |
| Amazon SQS | Hàng đợi bất đồng bộ giữa EventBridge và EC2 Worker; hỗ trợ retry và Dead Letter Queue. |
| Amazon EventBridge | Tạo business jobs định kỳ và gửi message vào SQS. |
| Amazon CloudWatch | Thu thập logs/metrics và kích hoạt alarm khi có bất thường. |
| Amazon SNS | Gửi email cảnh báo tới Ops/Admin khi CloudWatch Alarm được kích hoạt. |
| Amazon S3 | Lưu trữ dữ liệu, logs hoặc evidence phục vụ kiểm tra và audit. |
| AWS Systems Manager | Cho phép quản trị EC2 thông qua Session Manager thay vì SSH public. |
| AWS IAM | Quản lý quyền truy cập theo nguyên tắc least privilege. |
| AWS KMS | Mã hóa dữ liệu lưu trữ cho các tài nguyên cần bảo vệ, bao gồm RDS storage. |
| AWS Config | Theo dõi thay đổi cấu hình tài nguyên AWS theo thời gian. |
| AWS CloudTrail | Ghi nhận API activity để phục vụ audit và điều tra sau sự cố. |
| VPC Flow Logs | Ghi metadata lưu lượng mạng trong VPC vào CloudWatch để hỗ trợ troubleshooting và audit. |
| VPC Endpoints | Cho phép EC2 truy cập các AWS services cần thiết qua private AWS network path. |
| Terraform | Định nghĩa và triển khai hạ tầng dưới dạng Infrastructure as Code. |
| OPA/Rego | Kiểm tra policy trước khi triển khai Terraform plan. |

---

### 5. Thiết kế thành phần

#### Tầng Compute

EC2 Worker được đặt trong private subnet, không có public IP và không mở các cổng quản trị như SSH ra Internet. Worker đóng vai trò xử lý các batch jobs nội bộ, ví dụ đối soát dữ liệu giao dịch, validate dữ liệu định kỳ hoặc xử lý logs nghiệp vụ.

Kỹ sư quản trị truy cập EC2 thông qua AWS Systems Manager Session Manager, thay vì SSH public. Cách này giúp giảm bề mặt tấn công và phù hợp với mục tiêu private-first của hệ thống.

#### Tầng Database

Tầng database sử dụng Amazon RDS PostgreSQL cấu hình Single-AZ, không public, chỉ nhận kết nối nội bộ từ EC2 Worker thông qua Security Group. Storage của RDS được mã hóa bằng AWS KMS. CloudWatch alarms được dùng để giám sát CPU và dung lượng lưu trữ.

#### Tầng Queue và Job Processing

Amazon EventBridge tạo job định kỳ và gửi message vào Amazon SQS. EC2 Worker chủ động poll message từ SQS để xử lý. Mô hình này giúp tách rời producer và worker, tránh phụ thuộc vào xử lý đồng bộ, đồng thời hỗ trợ retry khi worker tạm thời không hoạt động.

#### Failure Handling

SQS được cấu hình với Dead Letter Queue. Nếu message xử lý thất bại nhiều lần, message sẽ được chuyển sang DLQ thay vì bị mất hoặc retry vô hạn. DLQ giúp Ops/Admin điều tra lỗi, phân tích nguyên nhân và xử lý lại job khi cần.

#### Monitoring và Alerting

CloudWatch thu thập logs/metrics và kích hoạt alarm khi phát hiện bất thường. SNS gửi email cảnh báo tới Ops/Admin. VPC Flow Logs gửi metadata lưu lượng mạng vào CloudWatch để hỗ trợ quan sát network traffic và điều tra sự cố.

#### Audit và Governance

CloudTrail ghi nhận API activity. AWS Config theo dõi thay đổi cấu hình tài nguyên. IAM kiểm soát quyền truy cập, KMS hỗ trợ mã hóa, S3 lưu trữ dữ liệu/log/evidence phục vụ kiểm tra sau triển khai. OPA/Rego đóng vai trò policy gate trước khi Terraform apply.

---

### 6. Luồng kiến trúc

Các số trên diagram thể hiện ba nhóm luồng chính: deployment flow, business processing flow và operational flow.

1. Developer thay đổi Terraform/IaC code.
2. IaC Pipeline tạo plan và chuyển qua Policy Gate.
3. Policy Gate kiểm tra cấu hình trước khi triển khai tài nguyên AWS.
4. EventBridge tạo business job định kỳ và gửi message vào SQS.
5. EC2 Worker trong VPC chủ động poll job từ SQS.
6. EC2 Worker xử lý job và kết nối RDS khi cần lưu hoặc đọc dữ liệu.
7. Nếu message xử lý thất bại nhiều lần, SQS chuyển message sang Dead Letter Queue.
8. VPC Flow Logs gửi metadata lưu lượng mạng vào CloudWatch.
9. EC2 truy cập CloudWatch thông qua VPC Endpoints để hỗ trợ monitoring/logging path.
10. CloudWatch Alarm kích hoạt SNS.
11. SNS gửi email cảnh báo tới Ops/Admin.
12. CloudTrail, AWS Config và S3 hỗ trợ lưu trữ và đối chiếu audit evidence.

---

### 7. Triển khai kỹ thuật

Project được triển khai theo các giai đoạn:

1. **Thiết kế kiến trúc:** Xác định hệ thống là backend job-processing platform, không phải website public.
2. **Thiết kế network:** Tạo VPC, private subnets, route tables và security groups.
3. **Thiết kế compute/database:** Tạo EC2 Worker và RDS PostgreSQL private.
4. **Thiết kế messaging:** Tạo SQS, Dead Letter Queue và EventBridge.
5. **Thiết kế monitoring/alerting:** Tạo CloudWatch alarms và SNS email notification.
6. **Thiết kế audit/security:** Tạo CloudTrail, AWS Config, VPC Flow Logs, KMS và IAM.
7. **Infrastructure as Code:** Viết Terraform modules cho từng nhóm tài nguyên.
8. **Policy as Code:** Dùng OPA/Rego để kiểm tra Terraform plan trước khi apply.
9. **Validation:** Kiểm thử luồng EventBridge → SQS → EC2 → processing log → DLQ/CloudWatch/SNS.
10. **Cleanup:** Xóa tài nguyên bằng Terraform destroy để tránh phát sinh chi phí.

---

### 8. Lộ trình và mốc triển khai

| Giai đoạn | Nội dung |
|---|---|
| Giai đoạn 1 | Nghiên cứu bài toán, xác định project là secure internal job-processing system. |
| Giai đoạn 2 | Thiết kế kiến trúc AWS và vẽ diagram phản ánh đúng tài nguyên triển khai thực tế. |
| Giai đoạn 3 | Viết Terraform modules cho VPC, EC2, RDS, SQS, EventBridge, CloudWatch, SNS, S3 và governance services. |
| Giai đoạn 4 | Viết OPA/Rego policy gate để kiểm tra Terraform plan. |
| Giai đoạn 5 | Deploy hạ tầng lên AWS và xác nhận tài nguyên trên AWS Console. |
| Giai đoạn 6 | Validate business flow và chụp evidence. |
| Giai đoạn 7 | Viết workshop end-to-end và cleanup tài nguyên để kiểm soát chi phí. |

---

### 9. Ước tính ngân sách

Project được thiết kế theo hướng kiểm soát chi phí trong phạm vi lab.

Đối với mô hình chạy liên tục 24/7, chi phí ước tính khoảng **85–95 USD/tháng**. Ước tính này dựa trên cấu hình:

- một EC2 Worker kích thước nhỏ;
- một RDS PostgreSQL Single-AZ;
- VPC Endpoints cần thiết;
- SQS, EventBridge, CloudWatch, SNS ở mức lưu lượng thấp;
- S3 lưu trữ dữ liệu/log/evidence dung lượng nhỏ;
- KMS, CloudTrail, AWS Config và VPC Flow Logs ở mức sử dụng lab.

Trong quá trình demo thực tế, chi phí có thể thấp hơn nhiều vì hạ tầng chỉ được triển khai trong thời gian ngắn, kiểm thử, chụp evidence và cleanup sau đó.

Project tránh sử dụng NAT Gateway và không dùng public IP cho EC2 Worker để vừa giảm chi phí, vừa hạn chế public exposure. Đây là lý do hệ thống sử dụng VPC Endpoints cho các đường truy cập AWS services cần thiết.

---

### 10. Đánh giá rủi ro

| Rủi ro | Ảnh hưởng | Cách giảm thiểu |
|---|---|---|
| Worker tạm dừng hoặc lỗi | Job chưa được xử lý ngay | SQS giữ message trước khi worker xử lý; DLQ giữ failed jobs. |
| Database Single-AZ không có HA đầy đủ | Ảnh hưởng availability nếu AZ gặp sự cố | Dùng Single-AZ cho lab để tiết kiệm chi phí; có thể bật Multi-AZ khi triển khai production. |
| Sai cấu hình security group hoặc public access | Rủi ro bảo mật | Dùng Terraform và OPA/Rego để kiểm soát cấu hình trước khi deploy. |
| Chi phí AWS tăng do để quên tài nguyên | Vượt ngân sách thực tập | Có cleanup bằng Terraform destroy và theo dõi Billing Forecast. |
| Thiếu bằng chứng vận hành | Khó audit hoặc review | Dùng CloudTrail, AWS Config, VPC Flow Logs, CloudWatch và evidence screenshots. |

---

### 11. Kết quả kỳ vọng

Sau khi hoàn thành, project kỳ vọng đạt được các kết quả sau:

- triển khai được một backend system xử lý job nội bộ trên AWS;
- EC2 Worker và RDS không expose trực tiếp ra Internet;
- business jobs được tạo bằng EventBridge và đưa vào SQS;
- EC2 Worker có thể poll job từ SQS và xử lý;
- failed jobs được cô lập bằng Dead Letter Queue;
- CloudWatch/SNS gửi cảnh báo khi có bất thường;
- CloudTrail, AWS Config, VPC Flow Logs, CloudWatch và S3 tạo audit evidence;
- Terraform giúp tái triển khai và cleanup hạ tầng;
- OPA/Rego giúp kiểm tra policy trước khi triển khai.

Về dài hạn, project có thể được mở rộng cho các workload nội bộ như financial reconciliation, customer data validation, compliance evidence generation, invoice processing hoặc scheduled reporting.