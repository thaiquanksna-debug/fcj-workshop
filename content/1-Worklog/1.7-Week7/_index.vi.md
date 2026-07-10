---
title: "Worklog tuần 7"
weight: 7
chapter: false
pre: " <b>1.7 </b> "
---

# Auto Scaling Group, Launch Template và health check

**Thời gian:** 25/05/2026 - 31/05/2026

## Mục tiêu tuần 7

- Thực hành mô hình triển khai ứng dụng có khả năng mở rộng.
- Hiểu Launch Template, ASG, health check và quan hệ với CloudWatch metrics.
- Phân tích điểm khác giữa autoscaling web app và worker chạy theo queue.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Học ASG và Launch Template trong workshop 000006<br>- Ghi chú AMI, user data và instance profile trong template | 25/05/2026 | 25/05/2026 | [ASG Workshop](https://000006.awsstudygroup.com/vi/)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |
| 3 | - Tạo Launch Template và cấu hình instance cơ bản<br>- Kiểm tra tag và thông tin launch instance | 26/05/2026 | 26/05/2026 | [Launch Template Lab](https://000006.awsstudygroup.com/vi/) |
| 4 | - Tạo Auto Scaling Group và kiểm tra desired/min/max capacity<br>- Quan sát instance được tạo tự động | 27/05/2026 | 27/05/2026 | [ASG Workshop](https://000006.awsstudygroup.com/vi/) |
| 5 | - Tìm hiểu metric dùng cho scaling và health check<br>- Liên hệ ASG với CloudWatch alarms | 28/05/2026 | 28/05/2026 | [CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)<br>[ASG Workshop](https://000006.awsstudygroup.com/vi/) |
| 6-CN | - Phân tích vì sao project cuối kỳ không cần public ALB nếu là internal worker<br>- Cleanup ASG, launch template và instance tạo ra | 29/05/2026 | 31/05/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 7

- Hiểu cách ASG tạo và thay thế instance dựa trên template/health.
- Biết đọc capacity và health check ở mức cơ bản.
- Có luận điểm để giải thích lựa chọn kiến trúc project: worker nội bộ có thể scale theo queue/metric, không nhất thiết cần public traffic entrypoint.


---
