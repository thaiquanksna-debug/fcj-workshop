---
title: "Worklog tuần 4"
weight: 4
chapter: false
pre: " <b>1.4 </b> "
---

#  EC2 cơ bản, kết nối máy chủ và triển khai ứng dụng demo

**Thời gian:** 04/05/2026 - 10/05/2026

## Mục tiêu tuần 4

- Thực hành EC2 Windows/Linux và các thao tác vận hành cơ bản.
- Triển khai ứng dụng demo để hiểu compute workload.
- Ghi chú cách xử lý sự cố key pair, instance type và security group.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Học EC2 instance, AMI, key pair, EBS và security group<br>- Launch Linux instance và kiểm tra trạng thái | 04/05/2026 | 04/05/2026 | [EC2 Workshop](https://000004.awsstudygroup.com/vi/)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |
| 3 | - Kết nối Linux bằng SSH/Session Manager tùy cấu hình<br>- Cài package cơ bản và kiểm tra system log | 05/05/2026 | 05/05/2026 | [Get Started EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)<br>[EC2 Workshop](https://000004.awsstudygroup.com/vi/) |
| 4 | - Triển khai ứng dụng CRUD mẫu AWS User Management<br>- Kiểm tra port, firewall, inbound rule và lỗi kết nối | 06/05/2026 | 06/05/2026 | [EC2 Workshop](https://000004.awsstudygroup.com/vi/)<br>[Security Group Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) |
| 5 | - Thực hành thay đổi instance type và tạo custom AMI<br>- Ghi lại khi nào cần dùng AMI thay vì cài lại thủ công | 07/05/2026 | 07/05/2026 | [EC2 Basic Lab](https://000004.awsstudygroup.com/vi/)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html) |
| 6-CN | - Tổng hợp lỗi EC2: key pair, port chưa mở, sai subnet, sai security group<br>- Cleanup instance, snapshot và EIP nếu có | 08/05/2026 | 10/05/2026 | [AWS Budgets Workshop](https://000007.awsstudygroup.com/vi/)<br>[EC2 Workshop](https://000004.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 4

- Triển khai được EC2 và ứng dụng demo theo lab.
- Hiểu các thành phần cần có để một instance hoạt động: AMI, subnet, security group, key/role, storage.
- Biết xác định lỗi kết nối EC2 theo hướng network/permission/config thay vì sửa mò.


---
