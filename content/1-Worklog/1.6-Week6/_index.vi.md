---
title: "Worklog tuần 6"
weight: 6
chapter: false
pre: " <b>1.6 </b> "
---

#  RDS và tích hợp database cho ứng dụng

**Thời gian:** 18/05/2026 - 24/05/2026

## Mục tiêu tuần 6

- Thực hành tạo RDS và kết nối từ EC2/application layer.
- Hiểu backup, snapshot, endpoint và DB subnet group.
- Ghi chú các yêu cầu bảo mật database trong project.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Học RDS, DB instance, engine, subnet group, parameter group và backup window<br>- So sánh RDS với database cài trên EC2 | 18/05/2026 | 18/05/2026 | [RDS Workshop](https://000005.awsstudygroup.com/vi/)<br>[RDS Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 3 | - Tạo DB subnet group và RDS trong private subnet<br>- Cấu hình security group cho EC2 kết nối DB | 19/05/2026 | 19/05/2026 | [RDS Workshop](https://000005.awsstudygroup.com/vi/)<br>[Security Group Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) |
| 4 | - Kết nối database từ EC2/app và tạo dữ liệu kiểm thử<br>- Kiểm tra endpoint, port, username/password và lỗi network | 20/05/2026 | 20/05/2026 | [RDS Getting Started](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 5 | - Thực hành backup/snapshot và quan sát trạng thái monitoring của RDS<br>- Ghi chú bằng chứng cần chụp khi demo database | 21/05/2026 | 21/05/2026 | [RDS Backup Lab](https://000005.awsstudygroup.com/vi/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |
| 6-CN | - Tổng hợp yêu cầu DB cho project: private, không public, chỉ worker truy cập, có evidence record<br>- Cleanup RDS để tránh chi phí | 22/05/2026 | 24/05/2026 | [RDS Cleanup Lab](https://000005.awsstudygroup.com/vi/)<br>[AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 6

- Tạo và kết nối được RDS trong lab.
- Hiểu cách database managed service giảm phần vận hành so với tự quản trị DB server.
- Chuẩn bị được checklist evidence cho RDS: endpoint private, security group, record được ghi từ worker và trạng thái resource.


---
