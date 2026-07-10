---
title: "Evidence checklist"
date: 2026-07-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---


Mục này chỉ được sử dụng để giúp các reviewer hiểu rõ kết quả triển khai thực tế. Các bằng chứng (evidence) này không thay thế cho các bước triển khai từ đầu đến cuối trong các mục trước đó. Luồng workshop cốt lõi vẫn là: setup → source generation → Terraform/OPA deployment → business-flow validation → cleanup.

## A. Core end-to-end evidence

Các ảnh chụp màn hình bên dưới là quan trọng nhất vì chúng chứng minh rằng workload có đầy đủ đầu vào (input), xử lý (processing), xử lý lỗi (failure handling) và cảnh báo (alerting).

### 1. Architecture diagram

![Final architecture diagram](/images/5-Workshop/private-by-default/00-final-architecture-diagram.png)

  Sơ đồ kiến trúc cuối cùng của Private-by-Default AWS Workload Platform, hiển thị các deployment controls, private workload VPC, EC2 Private Worker, SQS, DLQ, CloudWatch Alarm, SNS Email, RDS PostgreSQL, và audit evidence plane.

### 2. SQS Processing Queue receives business jobs

**SQS Processing Queue nhận các business processing jobs để xử lý bất đồng bộ (asynchronous processing) bởi EC2 Private Worker.**

![SQS poll messages](/images/5-Workshop/private-by-default/01-sqs-poll-for-messages.png)

**Bằng chứng này chứng minh điều gì?**

Processing queue đang hoạt động và lưu trữ thành công các business jobs trước khi chúng được thực thi bởi private worker.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Thay vì chỉ đại diện cho một demo payload JSON đơn giản, mỗi message có thể đại diện cho một enterprise workload thực tế như:

Daily customer CSV validation
Financial transaction reconciliation
Compliance evidence generation
Invoice processing
Internal reporting

Việc sử dụng Amazon SQS giúp tách biệt (decouple) job producers khỏi các workers, cải thiện độ tin cậy (reliability) và cho phép xử lý bất đồng bộ mà không cần để lộ các backend systems ra Internet.

### 3. Dead Letter Queue enabled

  Processing Queue đã được kích hoạt Dead Letter Queue để lưu trữ các tin nhắn bị lỗi sau nhiều lần xử lý thất bại liên tiếp (repeated processing failures).

![SQS DLQ enabled](/images/5-Workshop/private-by-default/02-sqs-dlq-enabled.png)

**Bằng chứng này chứng minh điều gì?**

Nền tảng hỗ trợ cô lập lỗi (failure isolation). Các messages bị lỗi nhiều lần trong quá trình xử lý sẽ tự động được chuyển đến DLQ thay vì bị loại bỏ hoặc thử lại vô hạn (retried indefinitely).

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Trong môi trường doanh nghiệp, các jobs bị lỗi thường chứa thông tin chẩn đoán (diagnostic information) rất giá trị.

Giữ các jobs bị lỗi bên trong DLQ cho phép các đội ngũ vận hành (operations teams):

Điều tra các sự cố production (production incidents);
Phân tích nguyên nhân gốc rễ (root cause analysis);
Chạy lại (replay) các jobs lỗi sau khi đã sửa lỗi;
Ngăn chặn các vòng lặp thử lại vô hạn (retry loops) gây lãng phí tài nguyên tính toán (compute resources).

Điều này cải thiện đáng kể tính ổn định vận hành (operational reliability) và khả năng kiểm toán (auditability).

### 4. Session Manager receives SQS message

  EC2 Private Worker được truy cập thông qua AWS Systems Manager Session Manager và nhận một message từ SQS mà không cần sử dụng SSH hoặc quyền truy cập IP công khai (public IP access).

![Session Manager receive message](/images/5-Workshop/private-by-default/03-session-manager-receive-message.png)

**Bằng chứng này chứng minh điều gì?**

Worker giao tiếp thành công với các AWS managed services thông qua mạng nội bộ (private networking) trong khi vẫn hoàn toàn không thể truy cập được từ public Internet.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Điều này chứng minh rằng các quản trị viên có thể vận hành private infrastructure một cách an toàn mà không cần mở các cổng quản lý (management ports).

Message nhận được xác nhận rằng:

Các business jobs đã đến được worker;
Mạng nội bộ (private networking) hoạt động chính xác;
Session Manager thay thế hoàn toàn cho phương thức quản trị SSH truyền thống.

Cách tiếp cận này giúp giảm thiểu bề mặt tấn công (attack surface) trong khi vẫn duy trì được quyền truy cập vận hành (operational access).

### 5. Worker business flow evidence

  Worker đọc một business job từ SQS. Phần message body bao gồm thuộc tính `source = eventbridge_scheduler`, chứng minh luồng công việc đi từ scheduler/queue đến private worker.

![Worker business flow evidence](/images/5-Workshop/private-by-default/04-private-worker-business-flow-evidence.png)

**Bằng chứng này chứng minh điều gì?**

Toàn bộ business workflow đã khởi chạy thành công sau khi worker nhận được message từ Amazon SQS.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Mặc dù workshop sử dụng một demo payload, nhưng luồng công việc tương tự có thể đại diện cho các production jobs thực tế như:

Customer data validation;
Financial batch processing;
Compliance evidence generation;
Scheduled reporting.

Bằng chứng này xác nhận rằng tính năng xử lý nghiệp vụ bất đồng bộ (asynchronous business processing) đang hoạt động chính xác.

Trong môi trường production, các bản ghi thực thi (execution records) tương tự có thể hỗ trợ cho:

Operational troubleshooting;
Compliance audits;
SLA verification;
Incident investigation.

### 6. Worker process log

  Worker ghi lại quá trình nhận job, xử lý job và xóa message khỏi SQS sau khi hoàn thành.

![Worker process log](/images/5-Workshop/private-by-default/05-worker-process-log.png)

**Bằng chứng này chứng minh điều gì?**

Worker đã hoàn thành quá trình xử lý thành công và tạo ra các execution logs mô tả chi tiết từng giai đoạn xử lý (processing stage).

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Application logs là những bằng chứng vận hành (operational evidence) thiết yếu.

Chúng cho phép các quản trị viên:

Xác minh việc thực thi thành công;
Điều tra các lỗi xử lý (processing failures);
Tái dựng lại mốc thời gian xử lý (processing timelines);
Chứng minh các hoạt động tuân thủ trong các kỳ đánh giá kiểm toán (audits).

Các logs này trở thành một phần của bằng chứng vận hành được tạo ra bởi nền tảng.

### 7. CloudWatch Alarm to SNS Email

  CloudWatch Alarm kích hoạt SNS Email để thông báo cho người vận hành (operator). Điều này chứng minh luồng cảnh báo: DLQ metric → CloudWatch Alarm → SNS Email.

![CloudWatch SNS email](/images/5-Workshop/private-by-default/06-cloudwatch-alarm-sns-email.png)

**Bằng chứng này chứng minh điều gì?**

Hệ thống giám sát (monitoring) và cảnh báo (alerting) đang hoạt động bình thường.

Nền tảng có thể tự động thông báo cho các quản trị viên bất cứ khi nào xảy ra các điều kiện bất thường.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Trong môi trường production, việc phát hiện sự cố chậm trễ trực tiếp làm tăng rủi ro cho doanh nghiệp.

Các thông báo tự động cho phép phản hồi nhanh hơn đối với:

Worker thất bại (worker failures);
Tồn đọng hàng đợi (queue backlogs);
Các jobs xử lý bị lỗi;
Các vấn đề về hiệu năng database (database performance issues).

Điều này giúp nâng cao tính sẵn sàng vận hành (operational availability) và giảm thời gian phản hồi sự cố (incident response time).

## B. Supporting screenshots for reviewer clarity

Các ảnh chụp màn hình bên dưới mang tính chất bổ trợ. Chúng không bắt buộc để chứng minh luồng xử lý từ đầu đến cuối (end-to-end flow), nhưng chúng giúp các reviewer hiểu rõ hơn về input source, private compute, private database và chi phí demo.

### 8. AWS Billing forecast

  Giao diện dashboard của Billing cho thấy chi phí thực tế của bản demo vẫn ở mức thấp vì cơ sở hạ tầng chỉ được triển khai tạm thời, sau đó được xác thực, ghi nhận tài liệu và dọn dẹp (cleaned up). Con số ước tính 85–95 USD/tháng trong đề xuất là dành cho việc triển khai pilot chạy liên tục 24/7.

![Billing forecast](/images/5-Workshop/private-by-default/07-billing-forecast-demo-cost.png)

**Bằng chứng này chứng minh điều gì?**

Cơ sở hạ tầng đã được triển khai thành công và đang phát sinh chi phí vận hành có thể đo lường được.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Tính minh bạch về chi phí (cost visibility) là một yêu cầu vận hành quan trọng.

Tính năng dự báo hóa đơn (billing forecast) cho phép các tổ chức:

Ước tính chi phí vận hành hàng tháng (monthly operating expenses);
So sánh giữa môi trường pilot và môi trường production;
Đánh giá mức độ hiệu quả của cơ sở hạ tầng (infrastructure efficiency);
Xác minh rằng các quy trình dọn dẹp (cleanup procedures) đã dừng thành công các khoản phí không cần thiết.

Điều này chứng minh rằng nền tảng đã cân nhắc đến cả khía cạnh kỹ thuật lẫn quản lý chi phí vận hành (operational cost management).

### 9. EventBridge Scheduler enabled

  EventBridge Scheduler được kích hoạt với cấu hình `rate(5 minutes)` để tự động tạo ra các demo business processing jobs. Phần SQS/worker message body cung cấp thêm bằng chứng cho thấy job source chính là từ `eventbridge_scheduler`.

![EventBridge Scheduler](/images/5-Workshop/private-by-default/08-eventbridge-scheduler-enabled.png)

**Bằng chứng này chứng minh điều gì?**

Các business workloads có thể được kích hoạt tự động mà không cần can thiệp thủ công.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Nhiều hệ thống doanh nghiệp cần thực thi các công việc nội bộ theo lịch trình (scheduled internal jobs) như:

Đối soát ban đêm (nightly reconciliation);
Tạo báo cáo (report generation);
Xác thực tuân thủ (compliance validation);
Đồng bộ dữ liệu (data synchronization).

EventBridge Scheduler cung cấp một cơ chế lên lịch được quản lý hoàn toàn (managed scheduling mechanism) để tự động tạo ra các yêu cầu xử lý mới theo các khoảng thời gian đã định trước.

### 10. EC2 Worker has no public IP

  EC2 Private Worker không có địa chỉ public IPv4 và chạy hoàn toàn bên trong một private subnet. Công việc quản trị được thực hiện thông qua Session Manager thay vì SSH công khai (public SSH).

![EC2 no public IP](/images/5-Workshop/private-by-default/09-ec2-worker-no-public-ip.png)

**Bằng chứng này chứng minh điều gì?**
Application worker được cách ly hoàn toàn khỏi việc truy cập trực tiếp từ Internet.

Quyền truy cập quản trị được thực hiện duy nhất thông qua AWS Systems Manager Session Manager.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Việc loại bỏ các địa chỉ public IP giúp giảm thiểu đáng kể bề mặt tấn công từ bên ngoài (external attack surface).

Kiến trúc này làm giảm khả năng xảy ra các cuộc tấn công nhắm vào:

Các dịch vụ SSH;
Tấn công vét cạn (brute-force authentication);
Dò quét thông tin qua Internet (Internet-based reconnaissance);
Truy cập từ xa trái phép (unauthorized remote access).

### 11. RDS Internet access disabled

  Amazon RDS PostgreSQL không kích hoạt Internet access gateway/public access, giúp giữ cho database tier không bị phơi nhiễm trực tiếp ra Internet.

![RDS Internet access disabled](/images/5-Workshop/private-by-default/10-rds-internet-access-disabled.png)

**Bằng chứng này chứng minh điều gì?**

Database ở môi trường production không thể bị truy cập trực tiếp từ Internet.

Chỉ các tài nguyên được ủy quyền bên trong private VPC mới có thể thiết lập kết nối đến database.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Các thông tin nhạy cảm của doanh nghiệp (sensitive enterprise information) được bảo vệ an toàn bên trong mạng nội bộ.

Kiến trúc này giúp đáp ứng các yêu cầu bảo mật phổ biến đối với các tổ chức xử lý dữ liệu mật của khách hàng hoặc doanh nghiệp.

### 12. RDS security group restriction

  RDS sử dụng một security group chuyên dụng cho database tier. Quyền truy cập inbound được giới hạn nghiêm ngặt từ security group của application worker thay vì mở cổng database ra Internet.

![RDS security group](/images/5-Workshop/private-by-default/11-rds-security-group-private-access.png)

**Bằng chứng này chứng minh điều gì?**

Chỉ có security group của application worker mới được phép kết nối tới database.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Việc hạn chế giao tiếp mạng theo nguyên tắc đặc quyền tối thiểu (principle of least privilege) giúp giảm thiểu khả năng dịch chuyển ngang (lateral movement) bên trong môi trường mạng và giảm thiểu tác động nếu một tài nguyên nào đó bị chiếm quyền điều khiển (compromised resources).

### 13. RDS encryption enabled

  RDS PostgreSQL đã được bật tính năng mã hóa (encryption) cho phân vùng lưu trữ chính (primary storage), chứng minh cơ chế bảo vệ dữ liệu ở trạng thái nghỉ (data-at-rest protection) cho database tier.

![RDS encryption enabled](/images/5-Workshop/private-by-default/12-rds-encryption-enabled.png)

**Bằng chứng này chứng minh điều gì?**

Dữ liệu được lưu trữ bên trong database được mã hóa ở trạng thái nghỉ (encrypted at rest) bằng cách sử dụng AWS managed encryption.

**Ý nghĩa đối với doanh nghiệp (Business significance)**

Mã hóa giúp bảo vệ thông tin kinh doanh nhạy cảm ngay cả khi các phương tiện lưu trữ vật lý bên dưới bị xâm phạm.

Cấu hình này hỗ trợ xây dựng các tiêu chuẩn bảo mật doanh nghiệp (enterprise security baselines), đồng thời đơn giản hóa các quy trình đánh giá bảo mật (security reviews) và kiểm tra tuân thủ (compliance assessments) sau này.

## Evidence summary

| Nhóm bằng chứng (Evidence group) | Câu hỏi được giải đáp (Question answered) |
|---|---|
| SQS + EventBridge | Dữ liệu đầu vào/job đến từ đâu? (Where does the input/job come from?) |
| EC2 Session Manager + worker log | Worker xử lý job ở đâu và như thế nào? (Where and how does the worker process the job?) |
| DLQ + CloudWatch + SNS | Hệ thống xử lý và cảnh báo lỗi như thế nào? (How does the system handle and alert on failure?) |
| EC2 no public IP + RDS private/encrypted | Hệ thống có đảm bảo private-by-default không? (Is the system private-by-default?) |
| Billing forecast | Chi phí demo có được kiểm soát không? (Is the demo cost controlled?) |

## Reviewer conclusion

Các bằng chứng thu thập được chứng minh rằng nền tảng đã đạt được thành công các mục tiêu đã đề ra trong đề xuất dự án.

Các ảnh chụp màn hình xác nhận rằng:

- Các business jobs được tạo tự động bởi EventBridge Scheduler.
- Các jobs được chuyển qua Amazon SQS và được xử lý bởi một private EC2 Worker.
- Các jobs bị lỗi có thể được định tuyến đến Dead Letter Queue để phục vụ mục đích điều tra.
- CloudWatch Alarms và thông báo SNS Email cung cấp tính năng hiển thị vận hành (operational visibility) và cảnh báo sự cố (incident alerting).
- Workload hoạt động hoàn toàn bên trong ranh giới mạng nội bộ (private networking boundaries) mà không cần Internet Gateway, NAT Gateway, bastion hosts, hoặc địa chỉ public IP.
- Các tài nguyên database và storage sử dụng cơ chế mã hóa ở trạng thái nghỉ (encryption at rest) thông qua các customer-managed AWS KMS keys.
- Quyền truy cập quản trị được thực hiện thông qua AWS Systems Manager Session Manager.

Tựu trung lại, các kết quả này đã xác thực kiến trúc của Private-by-Default AWS Workload Platform và chứng minh rằng một workload an toàn, có khả năng kiểm toán (auditable) và có thể quản lý vận hành tốt hoàn toàn có thể được triển khai trên AWS mà không cần để lộ cơ sở hạ tầng trực tiếp ra public Internet.