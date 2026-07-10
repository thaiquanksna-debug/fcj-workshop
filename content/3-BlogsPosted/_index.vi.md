---
title: "Blogs Posted"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

{{% notice info %}}
Mục này tổng hợp các bài viết blog được thực hiện trong quá trình phát triển dự án Giải pháp và Kiến trúc Điện toán Đám mây AWS. Các bài viết sẽ tóm tắt những quyết định thiết kế, kinh nghiệm triển khai thực tế và các bài học kỹ thuật rút ra xuyên suốt dự án.
{{% /notice %}}

Trong suốt dự án này, chúng tôi đã khám phá nhiều khía cạnh khác nhau về lưu trữ web an toàn, tích hợp Generative AI và kiến trúc hướng sự kiện serverless trên AWS. Các bài blog dưới đây sẽ nêu bật những thông tin chuyên sâu quan trọng thu thập được trong quá trình thiết kế và triển khai.

### [3.1 Blog 1 - Deploy Static Website on AWS with Amazon S3 Private, CloudFront and OAC](3.1-Blog1/)

Bài viết này hướng dẫn bạn quy trình triển khai một website tĩnh an toàn, sẵn sàng cho môi trường production trên AWS. Nội dung bao gồm cách cấu hình Amazon S3 ở chế độ hoàn toàn riêng tư (private), phân phối nội dung toàn cầu với độ trễ thấp bằng Amazon CloudFront, và chặn truy cập trực tiếp vào bucket nhờ Origin Access Control (OAC).

### [3.2 Blog 2 - Designing a Vietnam AI Travel Assistant MVP with Amazon Bedrock, AWS Amplify, and Amazon Location Service](3.2-Blog2/)

Bài viết này trình bày cách thiết kế một ứng dụng Trợ lý Du lịch AI bản địa hóa sử dụng mô hình Retrieval-Augmented Generation (RAG). Nội dung bao gồm việc điều phối luồng xử lý end-to-end bằng AWS Lambda, thiết lập các bộ lọc kiểm soát prompt với Amazon Bedrock, trực quan hóa bản đồ qua Amazon Location Service và tích hợp full-stack với AWS Amplify.

### [3.3 Blog 3 - Designing an Event-Driven Order Management System MVP on AWS](3.3-Blog3/)

Bài viết này phân tích cách xây dựng một hệ thống quản lý và xử lý đơn hàng có khả năng mở rộng cao và hoàn toàn độc lập (decoupled) cho thương mại điện tử hiện đại. Nội dung đi sâu vào việc điều phối workflow serverless với AWS Step Functions, xử lý thông điệp bất đồng bộ bằng Amazon EventBridge và Amazon SQS, cùng các mô hình chịu lỗi vận hành cốt lõi như Idempotency và Dead-Letter Queues (DLQ).