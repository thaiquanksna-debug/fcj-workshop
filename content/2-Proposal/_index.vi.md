---
title: "Bản đề xuất"
date: 2026-07-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Nền tảng xử lý job nội bộ an toàn trên AWS

### 1. Tóm tắt điều hành

Secure Internal Job Processing Platform on AWS là nền tảng backend dành cho các tác vụ xử lý dữ liệu nội bộ cần được vận hành trong môi trường cloud riêng tư, có kiểm soát và có khả năng lưu vết. Hệ thống phù hợp với những workload như đối soát dữ liệu, kiểm tra file nghiệp vụ, xử lý batch logs, tạo bằng chứng audit hoặc thực hiện các tác vụ compliance định kỳ.

Project không phải là website public và không tiếp nhận công việc qua public API. Job được tạo theo lịch, đưa vào hàng đợi và được EC2 Worker chủ động lấy về xử lý. Worker và database không được expose trực tiếp ra Internet; việc truy cập các dịch vụ AWS cần thiết được thực hiện qua lớp VPC Endpoints.

Kiến trúc được triển khai theo mô hình Lean MVP nhằm cân bằng giữa bảo mật, khả năng kiểm chứng và chi phí. Hệ thống sử dụng một EC2 Worker và một RDS PostgreSQL Single-AZ, đồng thời dùng SQS để tách rời khâu tạo job khỏi khâu xử lý. Toàn bộ hạ tầng được quản lý bằng Terraform và được kiểm tra bằng OPA/Rego trước khi triển khai.

---

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Nhiều doanh nghiệp có các tác vụ nội bộ không phù hợp để triển khai như một dịch vụ public, ví dụ:

- đối soát hoặc kiểm tra dữ liệu giao dịch;
- xác thực file dữ liệu khách hàng;
- xử lý logs nghiệp vụ;
- tạo báo cáo và bằng chứng kiểm toán;
- chạy các tác vụ compliance theo lịch;
- đồng bộ hoặc kiểm tra dữ liệu giữa các hệ thống.

Một kịch bản điển hình là doanh nghiệp định kỳ tiếp nhận file CSV chứa dữ liệu khách hàng hoặc giao dịch từ một hệ thống nội bộ. File cần được đưa vào quy trình kiểm tra để xác định đúng định dạng, đầy đủ trường dữ liệu và có thể tiếp tục được xử lý bởi các hệ thống phía sau. Tác vụ này không cần cung cấp public API và cũng không nên được thực hiện trên một worker có thể truy cập trực tiếp từ Internet.

Nếu các workload này được triển khai thủ công hoặc đặt trong môi trường public thiếu kiểm soát, doanh nghiệp có thể gặp các vấn đề như worker và database bị expose, job bị gián đoạn khi worker lỗi, thiếu cơ chế lưu giữ và xử lý message thất bại, không có cảnh báo vận hành, thiếu audit evidence và khó kiểm soát cấu hình hạ tầng theo thời gian.

#### Giải pháp đề xuất

Project đề xuất một nền tảng xử lý job theo hướng private-first và queue-based. EventBridge Scheduler tạo job định kỳ và gửi message vào Amazon SQS. EC2 Worker trong private subnet chủ động lấy job từ queue, thực hiện tác vụ xử lý tương ứng và kết nối tới RDS PostgreSQL khi cần lưu hoặc đọc dữ liệu.

Trong kịch bản minh họa, một message có thể đại diện cho yêu cầu kiểm tra file CSV khách hàng, bao gồm các thông tin như mã job, loại tác vụ, nguồn tạo job và tên file cần xử lý. SQS đóng vai trò trung gian giữa thành phần tạo job và Worker, giúp Worker không cần tiếp nhận request qua public API.

Nếu Worker tạm thời không hoạt động, message vẫn được giữ trong queue để tiếp tục xử lý sau khi Worker được khôi phục. Nếu job thất bại nhiều lần, message được chuyển sang Dead Letter Queue để cô lập và phục vụ điều tra.

CloudWatch theo dõi trạng thái hệ thống; khi DLQ xuất hiện message, CloudWatch Alarm có thể kích hoạt SNS để gửi thông báo cho Ops/Admin. CloudTrail, AWS Config, VPC Flow Logs và S3 cung cấp dữ liệu phục vụ giám sát, kiểm tra cấu hình và lưu trữ audit evidence.

Toàn bộ hạ tầng được định nghĩa bằng Terraform. Trước khi triển khai, Terraform plan được kiểm tra bằng OPA/Rego nhằm phát hiện các cấu hình không phù hợp với mục tiêu private-first, chẳng hạn EC2 có public IP, RDS bật public access, security group mở cổng quản trị ra Internet hoặc tài nguyên lưu trữ không được mã hóa.

#### Giá trị mang lại

- hạn chế public exposure đối với Worker và database;
- không yêu cầu public API cho các tác vụ xử lý nội bộ;
- tách rời thành phần tạo job và Worker bằng hàng đợi bất đồng bộ;
- giữ job khi Worker tạm thời không hoạt động;
- cô lập failed jobs bằng Dead Letter Queue;
- hỗ trợ monitoring, alerting và audit evidence;
- quản lý hạ tầng nhất quán bằng Infrastructure as Code;
- kiểm tra yêu cầu bảo mật trước khi triển khai bằng Policy as Code;
- tạo một security baseline có thể tái sử dụng và mở rộng khi nhu cầu vận hành tăng.

---

### 3. Kiến trúc giải pháp

![Secure Internal Job Processing Platform Architecture](/fcj-workshop/images/2-Proposal/private-by-default/diagram.jpg)

**Hình:** Kiến trúc tổng quan của Secure Internal Job Processing Platform on AWS.

Kiến trúc được chia thành bốn vùng chức năng chính:

1. **External Deployment Controls:** Developer, Terraform Pipeline và OPA/Rego Policy Gate quản lý quá trình thay đổi hạ tầng.
2. **Private Workload trong VPC:** EC2 Worker và RDS PostgreSQL được đặt trong các private subnet, không nhận truy cập trực tiếp từ Internet.
3. **Job Processing & Operations:** EventBridge Scheduler, SQS, CloudWatch, SNS và Session Manager hỗ trợ tạo job, xử lý bất đồng bộ, cảnh báo và quản trị.
4. **Security & Governance:** IAM, KMS, Parameter Store, CloudTrail, AWS Config, VPC Flow Logs và S3 hỗ trợ kiểm soát truy cập, bảo vệ dữ liệu và lưu bằng chứng audit.

RDS được triển khai theo cấu hình Single-AZ để phù hợp phạm vi MVP. Hai private DB subnet thuộc hai Availability Zone được sử dụng làm DB Subnet Group, nhưng hệ thống chỉ vận hành một DB instance và không tuyên bố khả năng failover Multi-AZ.

Lớp VPC Endpoints trên diagram đại diện cho cơ chế truy cập riêng tới các AWS managed services cần thiết. Chi tiết từng loại endpoint, route table, security group và policy được quản lý trong Terraform thay vì trình bày trên sơ đồ tổng quan.

---

### 4. Dịch vụ AWS sử dụng

| Dịch vụ / Thành phần | Vai trò trong project |
|---|---|
| Amazon VPC | Cô lập workload và tổ chức private subnets cho compute, database và private service access. |
| Amazon EC2 | Worker xử lý các internal jobs lấy từ SQS. |
| Amazon RDS for PostgreSQL | Lưu trữ dữ liệu và metadata nghiệp vụ. |
| Amazon SQS | Hàng đợi bất đồng bộ giữa Scheduler và Worker; hỗ trợ retry và DLQ. |
| Amazon EventBridge Scheduler | Tạo job định kỳ và gửi vào processing queue. |
| Amazon CloudWatch | Thu thập telemetry, theo dõi hệ thống và đánh giá alarm. |
| Amazon SNS | Gửi thông báo vận hành cho Ops/Admin. |
| Amazon S3 | Lưu logs và audit evidence. |
| AWS Systems Manager Session Manager | Quản trị EC2 mà không cần public SSH. |
| AWS IAM | Cấp quyền cho người dùng, workload và các AWS services. |
| AWS KMS | Bảo vệ dữ liệu ứng dụng và dữ liệu logs/audit. |
| AWS Systems Manager Parameter Store | Lưu trữ bí mật database dưới dạng được bảo vệ. |
| AWS CloudTrail | Ghi nhận hoạt động API trong AWS account. |
| AWS Config | Theo dõi trạng thái và thay đổi cấu hình tài nguyên. |
| VPC Flow Logs | Ghi nhận metadata lưu lượng mạng để hỗ trợ quan sát và điều tra. |
| VPC Endpoints | Cung cấp đường truy cập riêng tới các AWS services cần thiết. |
| Terraform | Định nghĩa, triển khai và cleanup hạ tầng. |
| OPA/Rego | Kiểm tra policy trên Terraform plan trước khi apply. |

---

### 5. Thiết kế thành phần

#### Tầng Compute và Database

EC2 Worker vận hành trong private app subnet và không được truy cập trực tiếp từ Internet. Worker sử dụng IAM role để tương tác với các AWS services được cấp phép. RDS PostgreSQL nằm trong tầng database private và chỉ phục vụ kết nối nội bộ từ workload phù hợp.

Thiết kế Single-AZ là quyết định tối ưu chi phí cho giai đoạn MVP. Nếu yêu cầu availability tăng, kiến trúc có thể được mở rộng thông qua thay đổi Terraform thay vì thiết kế lại toàn bộ nền tảng.

#### Tầng Queue và Job Processing

EventBridge Scheduler đóng vai trò producer, còn EC2 Worker đóng vai trò consumer. SQS nằm giữa hai thành phần để giảm phụ thuộc trực tiếp, giữ job khi worker tạm dừng và hỗ trợ retry. Dead Letter Queue cô lập các message không thể xử lý thành công để Ops/Admin điều tra hoặc xử lý lại.

#### Tầng Monitoring và Operations

CloudWatch tiếp nhận telemetry của workload và network. DLQ Alarm theo dõi failed jobs và kích hoạt SNS khi cần thông báo. Session Manager cung cấp kênh quản trị EC2 mà không cần mở cổng quản trị public.

#### Tầng Security và Governance

IAM kiểm soát quyền truy cập; KMS bảo vệ dữ liệu; Parameter Store lưu trữ thông tin nhạy cảm. CloudTrail và AWS Config tạo audit trail về hoạt động và cấu hình, trong khi VPC Flow Logs hỗ trợ quan sát network behavior. Dữ liệu audit được tập trung vào S3 để phục vụ kiểm tra sau triển khai.

---

### 6. Luồng kiến trúc

Các số trên diagram thể hiện ba nhóm hoạt động: deployment, job processing và operations.

1. Developer cập nhật mã nguồn Terraform.
2. Terraform Pipeline tạo kế hoạch triển khai và chuyển kết quả qua OPA/Rego Policy Gate.
3. Sau khi vượt qua policy gate, Terraform triển khai hoặc cập nhật tài nguyên AWS.
4. EventBridge Scheduler tạo scheduled job và gửi message vào SQS Queue.
5. EC2 Worker chủ động poll queue, nhận message và xác nhận message đã xử lý.
6. VPC Endpoints cung cấp đường truy cập riêng giữa worker và các AWS managed services cần thiết.
7. Worker kết nối tới RDS PostgreSQL khi job cần đọc hoặc ghi dữ liệu.
8. Message thất bại nhiều lần được chuyển từ processing queue sang DLQ.
9. CloudWatch Alarm theo dõi sự xuất hiện của failed messages trong DLQ.
10. Khi alarm được kích hoạt, SNS gửi thông báo tới Ops/Admin.
11. Ops/Admin sử dụng Session Manager để thực hiện các tác vụ quản trị EC2 mà không cần public SSH.

Ngoài luồng được đánh số, CloudTrail và AWS Config đưa audit evidence vào S3; VPC Flow Logs gửi network telemetry tới CloudWatch.

---

### 7. Triển khai kỹ thuật

Hạ tầng được định nghĩa dưới dạng các Terraform modules theo từng nhóm chức năng như network, endpoints, compute, database, messaging, monitoring, logging và encryption. OPA/Rego được sử dụng để kiểm tra Terraform plan trước khi apply nhằm phát hiện những thay đổi không phù hợp với mục tiêu private-first.

Quá trình triển khai gồm ba bước chính:

1. kiểm tra mã nguồn và tạo Terraform plan;
2. đánh giá plan bằng policy gate;
3. apply, validate luồng hệ thống và thu thập evidence.

Chi tiết cấu hình tài nguyên, IAM permissions, security-group rules, endpoint definitions, alarm thresholds và lifecycle settings được lưu trong Terraform source code và không lặp lại trong proposal này.

---

### 8. Lộ trình và mốc triển khai

| Giai đoạn | Nội dung |
|---|---|
| Giai đoạn 1 | Xác định bài toán internal job processing và các yêu cầu private-first. |
| Giai đoạn 2 | Thiết kế network boundary, workload flow và security controls. |
| Giai đoạn 3 | Xây dựng Terraform modules và OPA/Rego policies. |
| Giai đoạn 4 | Triển khai môi trường MVP trên AWS. |
| Giai đoạn 5 | Kiểm thử job flow, failure handling, monitoring và audit evidence. |
| Giai đoạn 6 | Hoàn thiện workshop, tài liệu và cleanup tài nguyên. |

---

### 9. Ước tính ngân sách

Project được thiết kế cho môi trường lab/MVP có lưu lượng thấp. Chi phí dự kiến khoảng **85–95 USD/tháng** nếu duy trì toàn bộ hạ tầng liên tục, nhưng có thể thấp hơn đáng kể khi chỉ triển khai trong thời gian thực hành, kiểm thử và chụp evidence.

Thiết kế ưu tiên các tài nguyên quy mô nhỏ, RDS Single-AZ và kiến trúc không phụ thuộc NAT Gateway. Terraform hỗ trợ cleanup nhất quán sau khi hoàn thành bài lab để hạn chế chi phí ngoài dự kiến.

Con số thực tế phụ thuộc vào Region, thời gian vận hành, lượng logs và mức sử dụng các AWS services.

---

### 10. Đánh giá rủi ro

| Rủi ro | Ảnh hưởng | Cách giảm thiểu |
|---|---|---|
| EC2 Worker tạm dừng hoặc lỗi | Job không được xử lý ngay | SQS giữ job để worker tiếp tục xử lý sau khi khôi phục. |
| RDS Single-AZ gặp sự cố | Database có thể gián đoạn | Chấp nhận trong phạm vi MVP; có lộ trình nâng cấp khi yêu cầu availability tăng. |
| Message xử lý thất bại lặp lại | Retry kéo dài hoặc che khuất lỗi nghiệp vụ | DLQ cô lập failed messages và CloudWatch/SNS thông báo cho Ops/Admin. |
| Sai cấu hình hạ tầng | Có thể làm giảm tính riêng tư hoặc khả năng kiểm soát | Terraform và OPA/Rego kiểm tra thay đổi trước khi triển khai. |
| Thiếu bằng chứng vận hành | Khó review hoặc điều tra sau sự cố | CloudTrail, AWS Config, VPC Flow Logs, CloudWatch và S3 cung cấp audit evidence. |
| Chi phí phát sinh ngoài dự kiến | Vượt ngân sách của môi trường lab | Theo dõi chi phí và cleanup tài nguyên bằng Terraform. |

---

### 11. Kết quả kỳ vọng

Sau khi hoàn thành, project kỳ vọng chứng minh được rằng:

- một internal job-processing platform có thể vận hành mà không cần public API;
- EC2 Worker và RDS không bị expose trực tiếp ra Internet;
- scheduled jobs được đưa vào queue và xử lý theo mô hình pull-based;
- SQS và DLQ hỗ trợ bảo vệ, retry và cô lập failed jobs;
- CloudWatch và SNS hỗ trợ phát hiện và thông báo sự cố vận hành;
- Session Manager cho phép quản trị EC2 theo đường private;
- CloudTrail, AWS Config, VPC Flow Logs và S3 tạo được audit evidence;
- Terraform và OPA/Rego giúp hạ tầng có thể tái triển khai, kiểm tra và cleanup nhất quán.

Về dài hạn, nền tảng có thể được mở rộng cho các workload như financial reconciliation, customer data validation, compliance evidence generation, invoice processing hoặc scheduled reporting mà không làm thay đổi mô hình kiến trúc cốt lõi.
