---
title: "Worklog tuần 11"
weight: 11
chapter: false
pre: " <b>1.11 </b> "
---

# Lên kế hoạch và hoàn thiện proposal

**Thời gian:** 22/06/2026 - 28/06/2026

## Mục tiêu tuần 11

- Chốt lại vấn đề nghiệp vụ và phạm vi giải pháp cho project cuối kỳ.
- Hoàn thiện proposal theo các mục: problem statement, solution architecture, technical implementation, timeline, budget, risk, expected outcomes.
- Đối chiếu proposal với nguyên tắc bảo mật, vận hành, độ tin cậy và tối ưu chi phí của AWS Well-Architected Framework.
- Phân chia nhiệm vụ triển khai tuần cuối để hai thành viên làm thống nhất.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Rà soát yêu cầu công ty và xác định project theo hướng private-by-default internal workload<br>- Viết lại problem statement để nhấn mạnh rủi ro public exposure, SSH/RDP mở ra Internet, database public và thiếu evidence sau triển khai | 22/06/2026 | 22/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)<br>[AWS Study Group](https://cloudjourney.awsstudygroup.com/vi/) |
| 3 | - Hoàn thiện kiến trúc giải pháp: VPC private subnet, EC2 worker, RDS private, VPC Endpoint, SQS, EventBridge, CloudWatch Logs/Metrics/Alarms, SNS và S3 evidence<br>- Ghi rõ vì sao workload không cần public URL và không cần public ALB | 23/06/2026 | 23/06/2026 | [Amazon VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)<br>[RDS Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 4 | - Viết phần technical implementation: Terraform modules, IAM role, security group, parameter store, log/evidence flow<br>- Bổ sung phần kiểm soát chi phí và dọn dẹp tài nguyên sau demo | 24/06/2026 | 24/06/2026 | [CloudFormation/IaC reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)<br>[AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |
| 5 | - Hoàn thiện timeline, milestone, ngân sách dự kiến và rủi ro kỹ thuật<br>- Kiểm tra nội dung proposal trên site Hugo, sửa tiêu đề, thứ tự mục và liên kết nội bộ | 25/06/2026 | 25/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[AWS Documentation](https://docs.aws.amazon.com/) |
| 6-CN | - Review chéo proposal giữa hai thành viên<br>- Chốt phạm vi triển khai tuần cuối: diagram, Terraform deploy, evidence capture, video/demo flow và checklist dọn dẹp | 26/06/2026 | 28/06/2026 | [CloudWatch Workshop](https://000008.awsstudygroup.com/vi/)<br>[AWS Study Group Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 11

- Hoàn thiện proposal theo cấu trúc website workshop, có đủ vấn đề, giải pháp, kiến trúc, triển khai, timeline, ngân sách, rủi ro và kết quả mong đợi.
- Thống nhất project cuối kỳ là mô hình triển khai internal worker trên AWS theo hướng private-by-default, có evidence để chứng minh triển khai đúng.
- Xác định rõ vai trò của từng dịch vụ trong business/action flow: EventBridge kích hoạt job, SQS giữ hàng đợi, EC2 worker xử lý, RDS lưu trạng thái, CloudWatch/S3 lưu bằng chứng.
- Chuẩn bị backlog cho tuần cuối gồm sửa diagram, triển khai hạ tầng, test luồng chạy, thu thập log/metric/alarm và hoàn thiện tài liệu nộp.


---
