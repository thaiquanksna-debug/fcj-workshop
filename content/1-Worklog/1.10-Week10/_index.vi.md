---
title: "Worklog tuần 10"
weight: 10
chapter: false
pre: " <b>1.10 </b> "
---

#  Chuẩn bị nội dung kỹ thuật cho proposal và project

**Thời gian:** 15/06/2026 - 21/06/2026

## Mục tiêu tuần 10

- Tổng hợp bài học từ các lab để chuyển thành yêu cầu kỹ thuật project.
- Chuẩn bị nội dung mô tả business/action flow của internal worker.
- Rà soát website Hugo trước khi vào 2 tuần làm chung.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Tổng hợp kiến thức identity/network/compute/database/observability thành yêu cầu kỹ thuật<br>- Viết nháp checklist service cần có trong project | 15/06/2026 | 15/06/2026 | [IAM Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)<br>[VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)<br>[EC2 Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)<br>[RDS Docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) |
| 3 | - Mô tả business/action flow: trigger job, queue, worker xử lý, DB lưu kết quả, CloudWatch/SNS/S3 lưu evidence<br>- Ghi chú từng service trả lời câu hỏi “tại sao cần dùng?” | 16/06/2026 | 16/06/2026 | [CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 4 | - Chuẩn bị phần security requirement: private subnet, no public DB, no inbound SSH/RDP từ Internet, least privilege, encrypted/evidence-aware logging<br>- Mapping requirement với diagram boundary | 17/06/2026 | 17/06/2026 | [Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)<br>[Security Group Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) |
| 5 | - Review nội dung Hugo liên quan Proposal và Workshop<br>- Đánh dấu chỗ cần sửa trong tuần 11: problem statement, architecture, timeline, risk, expected outcome | 18/06/2026 | 18/06/2026 | [AWS Study Group Cloud Journey](https://cloudjourney.awsstudygroup.com/vi/)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) |
| 6-CN | - Chốt danh sách việc cá nhân đã hoàn thành trước giai đoạn làm chung<br>- Chuẩn bị họp nhóm để thống nhất proposal tuần 11 | 19/06/2026 | 21/06/2026 | [AWS Documentation](https://docs.aws.amazon.com/)<br>[AWS Study Group](https://cloudjourney.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 10

- Chuyển được kiến thức lab thành yêu cầu kỹ thuật project.
- Có bản nháp business/action flow rõ ràng cho internal worker.
- Sẵn sàng làm chung với bạn ở tuần 11-12: proposal, diagram, triển khai project và evidence.


---
