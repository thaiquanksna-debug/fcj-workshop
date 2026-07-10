---
title: "Worklog tuần 8"
weight: 8
chapter: false
pre: " <b>1.8 </b> "
---

#  CloudWatch observability cho vận hành

**Thời gian:** 01/06/2026 - 07/06/2026

## Mục tiêu tuần 8

- Thực hành CloudWatch metrics, logs, alarms và dashboard.
- Hiểu observability như một phần của vận hành và evidence.
- Chuẩn bị cách chứng minh project chạy thật bằng log/metric/alarm.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Học CloudWatch tổng quan và cấu trúc metrics/logs/alarms<br>- Ghi chú vai trò CloudWatch trong giảm MTTR và giám sát hệ thống | 01/06/2026 | 01/06/2026 | [CloudWatch Workshop](https://000008.awsstudygroup.com/vi/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |
| 3 | - Thực hành xem metrics của EC2/RDS<br>- Tạo ghi chú các metric hữu ích cho project: CPU, network, error count, queue depth nếu có | 02/06/2026 | 02/06/2026 | [CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html)<br>[RDS Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 4 | - Thực hành CloudWatch Logs và log group<br>- Tìm cách trình bày log worker trong evidence | 03/06/2026 | 03/06/2026 | [CloudWatch Logs Lab](https://000008.awsstudygroup.com/vi/)<br>[CloudWatch Logs Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) |
| 5 | - Tạo alarm và dashboard trong lab<br>- Ghi chú alarm nào liên quan đến cảnh báo lỗi hệ thống | 04/06/2026 | 04/06/2026 | [CloudWatch Dashboard Lab](https://000008.awsstudygroup.com/vi/)<br>[CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) |
| 6-CN | - Tổng hợp evidence plane cho project: log, metric, alarm, notification và ảnh console<br>- Dọn dẹp dashboard/log thử nghiệm không cần giữ | 05/06/2026 | 07/06/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 8

- Thực hành được CloudWatch ở mức logs, metrics, alarms và dashboard.
- Hiểu vì sao project cần evidence plane thay vì chỉ có source code hoặc diagram.
- Chuẩn bị danh sách ảnh/log phải chụp trong tuần cuối.


---
