---
title: "Worklog tuần 3"
weight: 3
chapter: false
pre: " <b>1.3 </b> "
---

# IAM thực hành: user, group, role và policy

**Thời gian:** 27/04/2026 - 03/05/2026

## Mục tiêu tuần 3

- Thực hành IAM theo lab AWS Study Group 000002.
- Hiểu user/group/role/policy và cách áp dụng least privilege.
- Chuẩn bị nền tảng phân quyền cho project deploy bằng worker/service role.

---

## Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Học IAM user, group và policy<br>- Tạo mô hình nhóm quyền cơ bản cho người quản trị và người vận hành | 27/04/2026 | 27/04/2026 | [IAM Workshop](https://000002.awsstudygroup.com/vi/)<br>[IAM Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) |
| 3 | - Thực hành tạo IAM group/user và gán policy<br>- Kiểm tra đăng nhập bằng user thay vì root | 28/04/2026 | 28/04/2026 | [Create IAM Group/User](https://000002.awsstudygroup.com/vi/)<br>[IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| 4 | - Học IAM Role và temporary credentials<br>- Thực hành switch role theo hướng dẫn lab | 29/04/2026 | 29/04/2026 | [IAM Role Lab](https://000002.awsstudygroup.com/vi/) |
| 5 | - Viết policy giới hạn quyền theo Region/resource tag ở mức thử nghiệm<br>- Ghi chú lỗi AccessDenied và cách đọc thông báo lỗi | 30/04/2026 | 30/04/2026 | [IAM Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)<br>[Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html) |
| 6-CN | - Review quyền cần có cho EC2 worker: đọc parameter, ghi log, đọc/ghi queue, truy cập DB qua network<br>- Dọn dẹp user/role thử nghiệm | 01/05/2026 | 03/05/2026 | [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)<br>[AWS Free Tier Workshop](https://000001.awsstudygroup.com/vi/) |

---

## Kết quả đạt được tuần 3

- Nắm được cách tổ chức quyền IAM theo nhóm và theo role.
- Hiểu vì sao service role an toàn hơn hard-code access key trong source code.
- Có ghi chú lỗi phân quyền thường gặp để dùng khi triển khai project cuối kỳ.


---
