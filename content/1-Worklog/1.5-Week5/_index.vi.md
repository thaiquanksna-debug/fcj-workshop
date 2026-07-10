---
title: "Worklog tuần 5"
weight: 5
chapter: false
pre: " <b>1.5 </b> "
---

#  VPC nâng cao cho workload private

**Thời gian:** 11/05/2026 - 17/05/2026

## Mục tiêu tuần 5

- Xây dựng lại network theo hướng public/private rõ ràng.
- Thực hành security group và route để chuẩn bị cho RDS/worker private.
- Hiểu vì sao diagram phải thể hiện đúng boundary.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Ôn lại VPC, CIDR, subnet, route table và gateway<br>- Vẽ mô hình public subnet/private subnet cho workload nội bộ | 11/05/2026 | 11/05/2026 | [VPC Workshop](https://000003.awsstudygroup.com/vi/)<br>[Amazon VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| 3 | - Tạo VPC và subnet theo nhiều Availability Zone ở mức lab<br>- Kiểm tra route table từng subnet | 12/05/2026 | 12/05/2026 | [VPC Workshop](https://000003.awsstudygroup.com/vi/)<br>[VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| 4 | - Thiết kế security group cho web/app/db layer<br>- Ghi chú nguyên tắc chỉ mở đúng port và đúng nguồn | 13/05/2026 | 13/05/2026 | [Security Group Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)<br>[Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html) |
| 5 | - Tìm hiểu cách đặt RDS trong private subnet và chỉ cho phép app/worker truy cập<br>- Đánh dấu các thành phần phải nằm ngoài public Internet trong diagram | 14/05/2026 | 14/05/2026 | [RDS Workshop](https://000005.awsstudygroup.com/vi/)<br>[VPC Workshop](https://000003.awsstudygroup.com/vi/) |
| 6-CN | - Review network diagram ở mức logic và sửa nhãn dễ gây hiểu sai<br>- Dọn dẹp tài nguyên network lab nếu không còn dùng | 15/05/2026 | 17/05/2026 | [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)<br>[AWS Free Tier Workshop](https://000001.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 5

- Hiểu rõ hơn cách VPC tách public/private zone.
- Biết thiết kế security group theo luồng truy cập thực tế thay vì mở rộng mặc định.
- Có cơ sở để kiểm tra diagram tuần cuối: boundary, mũi tên và service placement phải khớp project.


---
