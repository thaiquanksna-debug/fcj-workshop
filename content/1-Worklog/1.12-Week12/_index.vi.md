---
title: "Worklog tuần 12"
weight: 12
chapter: false
pre: " <b>1.12 </b> "
---

#  Hoàn thiện diagram, triển khai proposal và triển khai project

**Thời gian:** 29/06/2026 - 06/07/2026

## Mục tiêu tuần 12

- Hoàn thiện AWS architecture diagram đúng với proposal và workload thực tế.
- Triển khai project theo hướng private-by-default bằng Terraform, tránh public exposure không cần thiết.
- Kiểm thử business/action flow và thu thập evidence từ AWS Console, CloudWatch, log, metric, alarm và output của worker.
- Hoàn thiện nội dung website Hugo để nộp báo cáo thực tập.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Kiểm tra lại proposal, source workshop và sơ đồ hiện tại<br>- Liệt kê các lỗi diagram: service đặt sai boundary, thiếu luồng business/action, thiếu metric/alarm, thiếu SNS/evidence plane hoặc nối sai hướng dữ liệu | 29/06/2026 | 29/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[Amazon VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)<br>[Security Group Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) |
| 3 | - Hoàn thiện diagram bị sai theo đúng yêu cầu: EC2 worker trong private subnet, RDS private, VPC Endpoint, SQS queue, EventBridge rule, CloudWatch Logs/Metrics/Alarms, SNS Notification và S3 evidence<br>- Đánh số luồng để người chấm hiểu workload xử lý gì từ đầu đến cuối | 30/06/2026 | 30/06/2026 | [CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)<br>[CloudWatch Logs Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)<br>[Amazon VPC Workshop](https://000003.awsstudygroup.com/vi/) |
| 4 | - Triển khai Terraform cho VPC, private subnet, IAM role, security group, RDS, EC2 worker và các service phụ trợ<br>- Kiểm tra output, state, tags và các biến cấu hình trước khi chạy demo | 01/07/2026 | 01/07/2026 | [CloudFormation/IaC concepts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)<br>[RDS Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 5 | - Chạy thử luồng workload: event/schedule tạo job, queue giữ message, worker xử lý, ghi dữ liệu vào RDS và sinh log/evidence<br>- Kiểm tra CloudWatch Logs, Metrics, Alarms và SNS notification | 02/07/2026 | 02/07/2026 | [CloudWatch Workshop](https://000008.awsstudygroup.com/vi/)<br>[CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Alarms.html) |
| 6 | - Chụp evidence bắt buộc: diagram, Terraform output, AWS Console, CloudWatch Logs, metric/alarm, RDS record, queue/job flow và cleanup checklist<br>- Đưa evidence vào các trang Workshop, Proposal, Self-Assessment và Feedback trong Hugo | 03/07/2026 | 03/07/2026 | [AWS Study Group Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/)<br>[AWS Documentation](https://docs.aws.amazon.com/) |
| 7-CN | - Review toàn bộ website Hugo, sửa lỗi chính tả, link nội bộ, tiêu đề sidebar và thứ tự trang<br>- Kiểm tra lần cuối trước khi nộp: proposal khớp diagram, diagram khớp project, worklog khớp timeline | 04/07/2026 | 05/07/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[AWS Study Group](https://cloudjourney.awsstudygroup.com/vi/) |
| 2 | - Tổng hợp bản nộp cuối cùng ngày 06/07/2026<br>- Kiểm tra lại cleanup tài nguyên để tránh phát sinh chi phí sau demo | 06/07/2026 | 06/07/2026 | [AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/)<br>[AWS Free Tier Workshop](https://000001.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 12

- Diagram cuối cùng thể hiện rõ boundary và luồng xử lý: private subnet, managed services, audit/evidence plane và notification plane.
- Proposal, project triển khai và website workshop thống nhất với nhau, không còn tình trạng sơ đồ nói một kiểu nhưng project chạy một kiểu.
- Hoàn thành demo business/action flow của internal worker: trigger → queue → worker → database → logs/metrics/alarms → notification/evidence.
- Có bộ evidence đủ để nộp gồm ảnh diagram, ảnh triển khai, log, metric, alarm, output, ghi nhận database và checklist cleanup.
- Hoàn tất dọn dẹp/kiểm soát chi phí sau khi test để hạn chế phát sinh tài nguyên AWS không cần thiết.
