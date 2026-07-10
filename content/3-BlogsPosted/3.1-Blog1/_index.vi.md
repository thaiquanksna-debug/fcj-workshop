---
title: "Triển khai Website Tĩnh trên AWS với Amazon S3 Private, CloudFront và OAC"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 3.1 Blog 1. </b> "
---


## Giới thiệu

Khi bắt đầu học AWS, nhiều người thường triển khai website tĩnh bằng cách bật **Static Website Hosting** trên Amazon S3, sau đó upload các file HTML, CSS và JavaScript lên bucket rồi cho phép truy cập công khai.

Cách làm này rất phù hợp cho mục đích học tập hoặc demo vì đơn giản và dễ thực hiện. Tuy nhiên, khi tiếp cận theo hướng gần với môi trường production hơn, việc public trực tiếp S3 bucket không phải lúc nào cũng là lựa chọn tối ưu.

Trong thực tế, một kiến trúc phổ biến hơn là:

- Amazon S3 Private Bucket
- Amazon CloudFront
- Origin Access Control (OAC)

Mô hình này giúp tách biệt giữa lớp lưu trữ dữ liệu và lớp phân phối nội dung, đồng thời cải thiện tính bảo mật và khả năng mở rộng.

---

# Kiến trúc tổng quan

Luồng truy cập của hệ thống theo thứ tự:

- **User** gửi yêu cầu qua giao thức **HTTPS** đến **CloudFront**
- **CloudFront** thực hiện **OAC Signed Request** để lấy dữ liệu từ **Private S3 Bucket**

Người dùng chỉ truy cập CloudFront.

CloudFront sẽ thay mặt người dùng lấy dữ liệu từ S3 thông qua Origin Access Control.

Trong mô hình này:

- CloudFront là public entry point
- S3 chỉ đóng vai trò lưu trữ dữ liệu
- Người dùng không thể truy cập trực tiếp vào bucket

---

# Các thành phần chính

## Amazon S3

Amazon S3 chịu trách nhiệm lưu trữ toàn bộ static files:

- index.html
- JavaScript
- CSS
- Images
- Fonts
- Các static assets khác

Một số cấu hình nên áp dụng:

- Block Public Access
- Bucket Owner Enforced
- Không bật Static Website Hosting
- Chỉ cho phép CloudFront đọc object

Điều này giúp giảm nguy cơ truy cập trực tiếp từ Internet.

---

## Amazon CloudFront

CloudFront đóng vai trò CDN và là điểm truy cập công khai của website.

Các lợi ích chính:

- HTTPS
- CDN Cache
- Custom Domain
- HTTP → HTTPS Redirect
- Edge Locations toàn cầu
- Security Headers
- Logging

CloudFront giúp giảm độ trễ và tăng tốc độ tải trang cho người dùng ở nhiều khu vực khác nhau.

---

## Origin Access Control (OAC)

Origin Access Control cho phép CloudFront truy cập S3 một cách an toàn hơn.

Với OAC:

- CloudFront ký request gửi tới S3
- Bucket Policy chỉ cho phép CloudFront Distribution được chỉ định truy cập

Điều này tốt hơn nhiều so với việc public bucket.

Một điểm cần lưu ý là OAC chỉ hoạt động với **S3 REST Endpoint**, hoàn toàn không hoạt động với **S3 Website Endpoint**.

---

# Bảo mật và nguyên tắc Least Privilege

Bucket Policy chỉ nên cho phép hành động `s3:GetObject` và chỉ áp dụng cho:

- Static website files
- CloudFront Distribution cụ thể

Tuy nhiên cần hiểu rằng:

OAC không phải là cơ chế phân quyền theo từng file.

Nếu một file nằm trong bucket và CloudFront có quyền đọc, file đó vẫn có thể được truy cập nếu người dùng biết chính xác URL.

Vì vậy:

- Không lưu tài liệu nội bộ trong bucket web
- Không lưu backup
- Không lưu dữ liệu nhạy cảm

Nên sử dụng bucket riêng cho website artifacts.

---

# Chiến lược Cache

Một trong những lỗi phổ biến nhất là: **Upload version mới nhưng website vẫn hiển thị version cũ**.

Nguyên nhân thường do CloudFront hoặc trình duyệt đang cache object cũ.

---

## Sử dụng Hashed Filename

Các framework hiện đại như React, Vue, Angular, Vite thường sinh ra file có kèm chuỗi hash như: `app.8f3a1c.js` hoặc `style.92ac0d.css`.

Mỗi lần build:

- Tên file thay đổi
- Cache cũ không ảnh hưởng

Nhờ đó có thể cache assets trong thời gian dài.

---

## Cache cho Assets

Ví dụ cấu hình tiêu đề phản hồi: `Cache-Control: public, max-age=31536000, immutable`

Áp dụng cho:

- JS
- CSS
- Images
- Fonts

---

## Cache cho index.html

Ngược lại: `Cache-Control: no-cache, no-store, must-revalidate` hoặc thiết lập TTL ngắn.

Lý do: index.html luôn phải trỏ đến version mới nhất của ứng dụng.

---

## CloudFront Invalidation

Khi deploy:

1. Upload assets có hash
2. Upload index.html
3. Invalidate `/index.html`

Bạn nên nhắm chính xác đường dẫn `/index.html`. Không nên lạm dụng việc làm mới toàn bộ `/*` vì sẽ làm tăng chi phí, làm chậm quá trình đồng bộ (propagate) và không tối ưu hiệu năng.

---

# SPA Routing

Các ứng dụng SPA thường có các route ảo như `/dashboard`, `/profile`, `/settings`. Nhưng thực tế trong S3 không tồn tại các file hay thư mục tương ứng này.

Khi người dùng thực hiện refresh tại trang `/dashboard`, CloudFront có thể nhận về mã lỗi số `403` hoặc `404`.

---

## Giải pháp phổ biến

Cấu hình Custom Error Response để chuyển hướng lỗi `403 → /index.html` và `404 → /index.html`, đồng thời trả về mã trạng thái `HTTP 200`.

Điều này cho phép các thư viện như React Router, Vue Router, Angular Router tiếp tục xử lý việc điều hướng route ở phía client.

---

## Lưu ý

Nếu redirect toàn bộ lỗi về index.html:

- Có thể che mất lỗi thật
- Khó debug hơn

Một cách tốt hơn là:

- Rewrite các route không có extension
- Giữ nguyên request tới JS, CSS, PNG, SVG

CloudFront Functions thường được dùng cho mục đích này.

---

# Custom Domain và SSL

Khi sử dụng CloudFront với custom domain, Certificate trong AWS Certificate Manager (ACM) bắt buộc phải nằm tại vùng **us-east-1 (N. Virginia)**.

Đây là lỗi rất phổ biến với người mới. Ví dụ cấu hình lỗi:

- Bucket ở Singapore
- Certificate ở Singapore

Kết quả là CloudFront sẽ không nhận và sử dụng được Certificate đó.

---

# Các lỗi thường gặp

## 403 Access Denied

Cần kiểm tra lại:

- Bucket Policy
- OAC đã attach chưa
- Origin có dùng đúng REST Endpoint không
- Object có tồn tại không
- Path có đúng chữ hoa chữ thường không

---

## Website vẫn hiển thị bản cũ

Cần kiểm tra lại các giá trị: Cache-Control, Age, X-Cache, ETag, Last-Modified.

---

## Custom Domain không hoạt động

Cần kiểm tra lại:

- ACM đã đặt ở us-east-1 chưa
- Certificate đã được validate thành công chưa
- Alternate Domain Name (CNAME) trong CloudFront
- Cấu hình DNS Record tại nhà đăng ký tên miền

---

# Ưu điểm của kiến trúc

- S3 không public
- HTTPS mặc định
- CDN toàn cầu
- Hỗ trợ custom domain
- Quản lý cache hiệu quả
- Dễ mở rộng với WAF và Monitoring

---

# Một số hạn chế

- Cấu hình phức tạp hơn
- Khó debug hơn
- Phát sinh chi phí CloudFront
- Cần hiểu rõ Bucket Policy và OAC

---

# Hướng phát triển tiếp theo

Sau khi triển khai thành công bằng AWS Console, bước tiếp theo nên là chuyển sang Infrastructure as Code.

Một số lựa chọn phổ biến:

- Terraform
- AWS CDK
- CloudFormation

Có thể quản lý toàn bộ S3 Bucket, CloudFront Distribution, OAC, Bucket Policy, Cache Behavior, Custom Domain bằng mã nguồn thay vì thao tác thủ công.

---

# Tích hợp CI/CD

Một pipeline đơn giản có thể bao gồm các bước tuần tự:

1. Build frontend
2. Upload assets lên S3
3. Upload index.html
4. CloudFront Invalidation
5. Smoke Test

Các công cụ phổ biến: GitHub Actions, GitLab CI/CD, AWS CodePipeline.

---

# Kết luận

Việc host static website bằng Amazon S3 là một bài thực hành rất tốt cho người mới học AWS.

Tuy nhiên, khi muốn tiếp cận theo hướng gần production hơn, kiến trúc sử dụng Amazon S3 Private Bucket, Amazon CloudFront và Origin Access Control sẽ là lựa chọn phù hợp hơn.

Mô hình này giúp nâng cao bảo mật, tối ưu hiệu năng, quản lý cache hiệu quả và tạo nền tảng tốt để tiếp tục mở rộng sang Infrastructure as Code, CI/CD, Monitoring và các quy trình vận hành thực tế.

<h2 style="text-align:center;">Sơ đồ kiến trúc</h2>

![Photo](/images/3-Blog/diagram1.jpg)

**Link bài viết** : https://www.facebook.com/groups/awsstudygroupfcj/permalink/2180621306036163