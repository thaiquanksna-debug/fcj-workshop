---
title: "Thiết kế MVP Hệ thống Quản lý Đơn hàng Hướng Sự kiện (Event-Driven) trên AWS"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 3.3 Blog 3. </b> "
---


## Giới thiệu

Bối cảnh của bài viết xuất phát từ một bài toán khá gần với thực tế tại Việt Nam: nhiều shop online, shop livestream, cửa hàng nhỏ hoặc team bán hàng nội bộ thường nhận đơn từ nhiều kênh khác nhau như website, form, inbox, livestream, nhân viên nhập tay hoặc các chiến dịch bán hàng ngắn hạn. Khi số lượng đơn còn ít, việc xử lý bằng Google Sheet, Excel hoặc một backend đơn giản có thể vẫn ổn. Tuy nhiên, khi đơn tăng lên, hệ thống bắt đầu gặp các vấn đề như trùng đơn, xử lý chậm, khó retry khi lỗi, khó biết đơn đang kẹt ở bước nào, hoặc mất dữ liệu khi một bước xử lý phía sau bị lỗi.

Theo báo cáo từ các tổ chức thương mại quốc tế, thị trường thương mại điện tử Việt Nam đang tăng trưởng mạnh mẽ với quy mô hàng chục tỷ USD. Điều này cho thấy bài toán vận hành đơn hàng online không chỉ phù hợp với các sàn lớn, mà còn rất gần với các shop nhỏ và doanh nghiệp vừa đang chuyển dần sang kênh bán hàng số.

Trong bài viết này, mình không thiết kế một hệ thống thương mại điện tử hoàn chỉnh. Thay vào đó, mình sẽ tập trung vào một MVP nhỏ hơn: **Hệ thống nhận đơn hàng, lưu đơn, phát event khi đơn được tạo, xử lý các bước phía sau bất đồng bộ và đưa đơn lỗi vào queue hoặc dead-letter queue để không bị mất dữ liệu.**

> **Mục tiêu của MVP:** Thiết kế này không phải là "production-ready" hoàn chỉnh (vốn cần thêm phân quyền nội bộ, audit log, đối soát cổng thanh toán, tích hợp đơn vị vận chuyển...). Tuy nhiên, đây là một baseline tốt để người học AWS làm quen với tư duy event-driven, serverless workflow, retry, DLQ, idempotency và observability.

---

# Thách thức của Kiến trúc Đồng bộ (Synchronous)

Khi mới bắt đầu, cách đơn giản nhất thường là tạo một API nhận request, sau đó trong cùng một function thực hiện tất cả các bước: validate đơn hàng, lưu database, trừ tồn kho, gửi email, gửi thông báo cho nhân viên, cập nhật trạng thái và trả response về frontend.

Cách làm này dễ hiểu và phù hợp cho demo nhỏ. Tuy nhiên, việc nhồi quá nhiều logic vào một API synchronous có thể gây ra nhiều vấn đề:
- Nếu bước gửi email bị lỗi, toàn bộ request tạo đơn có thể bị fail.
- Nếu hệ thống kho phản hồi chậm, người dùng phải chờ đợi lâu trên giao diện.
- Nếu cần retry một bước nào đó, ta phải tự viết logic retry phức tạp trong code.
- Nếu một bước phía sau bị lỗi sau khi đơn đã được lưu, rất khó biết trạng thái thật của đơn hàng là gì.

---

# Giải pháp: Event-Driven Architecture với AWS Serverless

Một cách tiếp cận hợp lý hơn là tách phần **"nhận đơn"** và phần **"xử lý đơn"** thành các bước riêng biệt. Khi đơn được tạo thành công, hệ thống phát ra một event (Ví dụ: `OrderCreated`). Các thành phần phía sau (consumers) sẽ lắng nghe event này để xử lý độc lập các tác vụ liên quan.

## Các thành phần cốt lõi trong MVP

- **Frontend / Admin Portal:** Triển khai bằng **AWS Amplify**, hoặc sử dụng static frontend trên **Amazon S3** và **Amazon CloudFront**. Nhiệm vụ chính là gửi yêu cầu tạo đơn và theo dõi trạng thái đơn hàng.
- **Amazon API Gateway:** Đóng vai trò là entry point cho backend API, tiếp nhận request từ frontend và chuyển đổi thành event cho Lambda xử lý.
- **AWS Lambda:** Chịu trách nhiệm xử lý business logic ở từng bước nhỏ (ví dụ: Lambda tạo đơn, Lambda kiểm tra tồn kho, Lambda gửi thông báo). Cách chia nhỏ này giúp mỗi function có trách nhiệm rõ ràng (single responsibility), dễ test và dễ cô lập lỗi.
- **Amazon DynamoDB:** Database chính để lưu đơn hàng, trạng thái đơn, lịch sử cập nhật. Thiết kế cấu trúc bảng dựa trên access pattern (ví dụ: phân vùng theo `shopId` và sắp xếp theo `orderId` hoặc `createdAt` để dễ truy vấn).
- **Amazon EventBridge:** Event bus trung tâm đóng vai trò decouple giữa producer và consumer. Service tạo đơn chỉ cần publish event `OrderCreated` lên EventBridge mà không cần biết phía sau có bao nhiêu thành phần khác đang lắng nghe.
- **Amazon SQS & Dead-Letter Queue (DLQ):** SQS queue được dùng cho các tác vụ bất đồng bộ cần xếp hàng đợi và có khả năng retry (gửi SMS/email, gọi API đơn vị vận chuyển). Nếu một message bị lỗi nhiều lần vượt quá cấu hình `maxReceiveCount`, nó sẽ tự động chuyển sang DLQ để bảo toàn dữ liệu và phục vụ debug.
- **AWS Step Functions:** Sử dụng khi quy trình xử lý đơn có nhiều bước phụ thuộc nhau phức tạp. Step Functions giúp phối hợp (orchestration) các dịch vụ thành một serverless workflow (state machine) có kiểm soát.

---

# Chi tiết Request Flow trong MVP

Luồng xử lý một đơn hàng từ lúc khởi tạo đến khi sẵn sàng bàn giao được thiết kế như sau:

1. **Khởi tạo đơn:** Khách hàng hoặc nhân viên tạo đơn từ frontend (thông tin sản phẩm, số lượng, địa chỉ, số điện thoại, hình thức thanh toán). Frontend gửi request đến API Gateway.
2. **Validate & Ghi nhận nhanh:** Lambda `CreateOrder` kiểm tra dữ liệu đầu vào. Nếu dữ liệu hợp lệ, Lambda tạo một `orderId` và lưu đơn vào DynamoDB với trạng thái ban đầu là `CREATED`. Hệ thống không xử lý các tác vụ phụ ở bước này để đảm bảo đơn được ghi nhận nhanh nhất.
3. **Phát dữ liệu sự kiện:** Lambda publish một event nhỏ gọn tên là `OrderCreated` lên Amazon EventBridge (chứa thông tin cốt lõi như `orderId`, `shopId`, `totalAmount`, `paymentMethod`).
4. **Định tuyến sự kiện (Routing):** EventBridge dựa trên các rule cấu hình để route event đến các target song song:
   - Gửi đến một SQS Queue đảm nhận nhiệm vụ gửi thông báo (Notification).
   - Kích hoạt một Step Functions Workflow đảm nhận quy trình xử lý nghiệp vụ đơn hàng.
5. **Điều phối Workflow (Step Functions):** 
   - **Bước 5.1 (Inventory Check):** Kiểm tra tồn kho. Nếu đủ hàng, cập nhật trạng thái đơn thành `INVENTORY_RESERVED`. Nếu thiếu hàng, chuyển trạng thái thành `WAITING_FOR_STOCK` và bắn cảnh báo cho nhân viên.
   - **Bước 5.2 (Payment Handling):** Nếu đơn hàng chọn hình thức COD, hệ thống chuyển thẳng sang bước đóng gói. Nếu chọn chuyển khoản hoặc ví điện tử, workflow sẽ tạm dừng để đợi sự kiện `PaymentConfirmed` (rất phù hợp với thói quen thanh toán và xác nhận thủ công tại Việt Nam).
6. **Hoàn tất giai đoạn:** Sau khi các bước chính hoàn tất, hệ thống cập nhật trạng thái đơn thành `READY_TO_PACK` hoặc `READY_TO_SHIP`. Worker đọc message từ SQS để gửi thông báo hoàn tất cho khách hàng.

---

# Các bài toán vận hành thực tế cần giải quyết

## 1. Xử lý trùng lặp đơn hàng (Idempotency)
Tình trạng người dùng bấm đặt hàng 2 lần do mạng chập chờn hoặc nhân viên nhập tay trùng lặp rất phổ biến. Để giải quyết, frontend cần đính kèm một `idempotencyKey` duy nhất cho mỗi phiên submit đơn. Backend sẽ lưu key này trong DynamoDB. Nếu nhận được request trùng key, backend sẽ không tạo đơn mới mà trả về ngay kết quả của đơn đã xử lý trước đó.

## 2. Tư duy về Tính nhất quán sau cùng (Eventual Consistency)
Khi chuyển sang kiến trúc hướng sự kiện, hệ thống không còn xử lý theo kiểu thời gian thực đồng bộ (synchronous). Trạng thái tồn kho, thông báo hay vận chuyển sẽ được cập nhật sau đó vài giây. Vì vậy, giao diện người dùng (UI) cần được thiết kế phù hợp: hiển thị trạng thái *"Đơn hàng đã được ghi nhận và đang xử lý"* thay vì hiển thị *"Đã hoàn tất mọi bước"*.

## 3. Khả năng quan sát (Observability) & Debug
Do một đơn hàng đi qua rất nhiều dịch vụ phân tán (API Gateway, Lambda, EventBridge, SQS, Step Functions), việc debug sẽ cực kỳ khó khăn nếu không có chiến lược.
- **Correlation ID:** Sử dụng chính `orderId` làm correlation ID để ghi log xuyên suốt tất cả các bước xử lý của các Lambda function.
- **CloudWatch Logs & Metrics:** Theo dõi số đơn lỗi, số message bị kẹt trong DLQ, tỉ lệ lỗi của Lambda hoặc các workflow bị fail của Step Functions.

## 4. Cấu hình Queue & Xử lý lỗi (Poison Message)
Không phải lỗi nào cũng giống nhau. Lỗi timeout kết nối tạm thời thì nên retry, nhưng lỗi do dữ liệu sai cấu trúc (poison message) thì có retry vô hạn cũng không thành công. Hệ thống cần:
- Đưa các cấu trúc lỗi dữ liệu thẳng vào DLQ để xử lý thủ công.
- Cấu hình tham số `visibilityTimeout` của SQS lớn hơn thời gian xử lý tối đa của Lambda worker để tránh tình trạng các worker khác giành quyền xử lý trùng lặp cùng một message.

---

# Quản lý Trạng thái Đơn hàng (State Model)

Để dễ theo dõi và vận hành, trường trạng thái của đơn hàng trong DynamoDB cần được định nghĩa rõ ràng theo từng giai đoạn của workflow, tránh dùng một trạng thái quá chung chung:

- `CREATED`: Đơn hàng mới được ghi nhận thành công vào hệ thống.
- `INVENTORY_RESERVED`: Đã giữ hàng trong kho thành công.
- `WAITING_FOR_PAYMENT`: Đang chờ xác nhận thanh toán (đối với đơn chuyển khoản/ví điện tử).
- `READY_TO_PACK`: Đơn hàng hợp lệ, đã chuẩn bị đủ cấu phần và sẵn sàng đóng gói.
- `SHIPPED`: Đã bàn giao thành công cho đơn vị vận chuyển.
- `FAILED` / `CANCELLED`: Đơn hàng bị hủy hoặc thất bại do thiếu hàng/lỗi thanh toán.

---

# Lộ trình phát triển hệ thống từ MVP

1. **Giai đoạn 1 (Thử nghiệm):** Triển khai thủ công bằng AWS Console các dịch vụ cơ bản: API Gateway, một Lambda `CreateOrder`, một bảng DynamoDB, một Event Bus trên EventBridge, một SQS Queue và một DLQ.
2. **Giai đoạn 2 (Chuẩn hóa):** Chuyển toàn bộ kiến trúc trên sang quản lý bằng mã nguồn (Infrastructure as Code) sử dụng **Terraform**, **AWS CDK** hoặc **CloudFormation**. Khởi tạo các IAM role phân quyền nghiêm ngặt theo nguyên tắc Least Privilege (ví dụ: Lambda tạo đơn chỉ được quyền ghi vào DynamoDB và publish vào EventBridge).
3. **Giai đoạn 3 (Tự động hóa):** Xây dựng các pipeline CI/CD (GitHub Actions, GitLab CI/CD) để tự động hóa việc kiểm thử mã nguồn Lambda, cập nhật cấu hình State Machine của Step Functions và deploy hạ tầng.

---

# Kết luận

Hệ thống quản lý đơn hàng theo kiến trúc event-driven là một bài thực hành rất giá trị cho người học AWS. Nó giúp tách biệt hoàn toàn phần nhận đơn và xử lý đơn, nâng cao khả năng chịu lỗi và giúp hệ thống dễ dàng mở rộng trong tương lai (ví dụ: khi muốn thêm một service phân tích dữ liệu, ta chỉ cần tạo một rule mới trên EventBridge mà không cần sửa bất kỳ dòng code nào ở backend tạo đơn cũ).

<h2 style="text-align:center;">Sơ đồ kiến trúc đề xuất</h2>

![Photo](/images/3-Blog/diagram3.jpg)

**Link bài viết tham khảo** : https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201747483923545