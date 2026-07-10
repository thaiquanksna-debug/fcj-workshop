---
title: "Thiết kế MVP Trợ lý Du lịch Việt Nam với Amazon Bedrock, AWS Amplify và Amazon Location Service"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 3.2 Blog 2. </b> "
---


## Giới thiệu

Khi nói đến ứng dụng AI cho du lịch, nhiều người mới học thường nghĩ ngay đến việc tạo một chatbot đơn giản: người dùng nhập câu hỏi, backend gọi một large language model, sau đó trả lại câu trả lời. Cách tiếp cận này khá dễ để demo, vì chỉ cần có giao diện chat, một API backend và một model để sinh phản hồi.

Tuy nhiên, nếu nhìn theo hướng gần với sản phẩm thực tế hơn, một chatbot du lịch không nên chỉ dựa vào kiến thức chung của model. Người dùng có thể hỏi những câu rất cụ thể như:
- "Ở Hội An nên đi đâu nếu chỉ có 1 ngày?"
- "Quán nào phù hợp với người không ăn hải sản?"
- "Lịch trình Đà Nẵng 3 ngày cho gia đình có trẻ nhỏ?"
- "Nên tránh đi đâu vào mùa mưa?"

Nếu hệ thống chỉ trả lời dựa trên kiến thức chung, câu trả lời có thể nghe rất tự tin nhưng không đủ kiểm chứng, thiếu ngữ cảnh địa phương, hoặc không phù hợp với dữ liệu mà doanh nghiệp thật sự muốn đưa cho người dùng.

Vì vậy, với MVP này, mình không thiết kế một chatbot trả lời tự do hoàn toàn. Thay vào đó, kiến trúc sẽ tách rõ giữa lớp giao diện, lớp xử lý yêu cầu, lớp AI, lớp dữ liệu tri thức, lớp bản đồ/địa điểm và lớp quan sát vận hành. Mục tiêu là tạo một baseline đủ tốt để học cách xây dựng ứng dụng AI có cấu trúc, thay vì chỉ gọi model rồi hiển thị response.

---

## Định nghĩa bài toán ở mức MVP

Người dùng nhập nhu cầu du lịch bằng ngôn ngữ tự nhiên, ví dụ: 

> "Tôi có 3 ngày ở Đà Nẵng, thích biển, đồ ăn địa phương, ngân sách vừa phải, không muốn di chuyển quá nhiều".

Hệ thống sẽ phân tích yêu cầu, truy xuất dữ liệu liên quan từ knowledge base, gọi Amazon Bedrock để tạo lịch trình gợi ý, sử dụng Amazon Location Service để lấy thông tin địa điểm hoặc tọa độ khi cần, sau đó trả lại lịch trình có cấu trúc cho người dùng.

Một output đơn giản có thể gồm:
- Lịch trình chi tiết theo từng ngày.
- Gợi ý địa điểm ăn uống, tham quan.
- Lưu ý di chuyển và các lựa chọn thay thế.
- Các câu hỏi follow-up (Ví dụ: *"Bạn muốn lịch trình thiên về nghỉ dưỡng, ăn uống hay trải nghiệm văn hóa hơn?"*).

> **Lưu ý:** Mô hình này không nên được hiểu là "production-ready" hoàn chỉnh. Một hệ thống production thật còn cần nhiều phần khác như kiểm soát dữ liệu đầu vào, kiểm thử chất lượng, cost control, rate limit, CI/CD, Infrastructure as Code... Tuy nhiên, đây là một MVP rất phù hợp để junior cloud engineer hoặc người đang học AWS rèn luyện tư duy thiết kế thực tế.

---

# Kiến trúc tổng quan và các thành phần

Luồng dữ liệu của hệ thống di chuyển qua các thành phần cốt lõi:
- **Frontend:** Người dùng truy cập ứng dụng web/mobile triển khai bằng **AWS Amplify**.
- **Backend API & Orchestration:** **AWS Lambda** tiếp nhận và điều phối request.
- **Generative AI & RAG:** **Amazon Bedrock** kết hợp **Knowledge Bases** để xử lý context và tạo câu trả lời.
- **Location Services:** **Amazon Location Service** cung cấp bản đồ và tọa độ địa lý.
- **Storage & Observability:** **Amazon DynamoDB** lưu lịch sử hội thoại; **Amazon CloudWatch** đảm nhận vai trò monitoring.

---

## Lớp Frontend và Backend Orchestration

### AWS Amplify
Phù hợp ở lớp frontend vì nó giúp xây dựng và triển khai web/mobile app nhanh hơn, hỗ trợ các nhu cầu thường gặp như hosting, authentication, data và backend integration. AWS mô tả Amplify là nền tảng giúp xây dựng ứng dụng fullstack với trải nghiệm thân thiện cho frontend developer.

### AWS Lambda
Đóng vai trò orchestration nhẹ ở backend. Lambda nhận request từ API, kiểm tra input, gọi Knowledge Base để retrieve context, gọi Amazon Bedrock để generate response, gọi Amazon Location Service nếu cần dữ liệu địa điểm, sau đó format response trả về frontend. Cách tiếp cận serverless này giúp giảm nhu cầu quản lý server và dễ thử nghiệm nhiều flow khác nhau.

---

## Lớp Generative AI và Chiến lược RAG

### Amazon Bedrock
Đóng vai trò là lớp Generative AI chính. Đây là nơi backend gửi prompt, context và yêu cầu của người dùng để model sinh ra lịch trình. Trong bài toán này, Bedrock được dùng để biến một yêu cầu tự nhiên thành một kế hoạch có cấu trúc (Overview, Itinerary by day, Places, Food suggestions, Transport notes, Safety notes).

### Retrieval Augmented Generation (RAG)
Không nên để model tự bịa toàn bộ nội dung (hallucination). Với RAG, hệ thống sẽ tìm những đoạn dữ liệu liên quan trong **Amazon Bedrock Knowledge Bases**, đưa chúng vào prompt, sau đó model tạo câu trả lời dựa trên ngữ cảnh đó.

Dữ liệu tri thức được chuẩn bị sẵn trong **Amazon S3** bao gồm:
- Danh sách điểm đến được chuẩn hóa tại các địa phương.
- Gợi ý lịch trình mẫu và FAQ cho khách quốc tế.
- Lưu ý văn hóa và thông tin mùa du lịch Việt Nam.

---

## Lớp Bản đồ và Lưu trữ Dữ liệu

### Amazon Location Service
Được sử dụng cho phần bản đồ và dữ liệu địa điểm. Đối với bối cảnh du lịch Việt Nam, phần location khá quan trọng vì lịch trình cần tối ưu theo khoảng cách, khu vực và thời gian di chuyển thực tế (ví dụ: nhóm các điểm Mỹ Khê, Sơn Trà thành một cụm hợp lý thay vì gợi ý các điểm quá xa nhau trong cùng một buổi).

### Amazon DynamoDB
Dùng để lưu các dữ liệu có cấu trúc đơn giản như user profile, session ID, conversation history, itinerary đã tạo, preference của người dùng hoặc feedback. Với MVP, chỉ cần một table có partition key theo `userId` hoặc `sessionId`, cùng một số item type như `PROFILE`, `CONVERSATION`, `ITINERARY`, `FEEDBACK` là đủ để bắt đầu.

---

# Chi tiết Request Flow trong hệ thống

Luồng xử lý một yêu cầu từ người dùng sẽ đi qua các bước tuần tự sau:

1. **Gửi yêu cầu:** Người dùng nhập yêu cầu trên frontend (Ví dụ: *"Tôi muốn đi Hội An 2 ngày, thích chụp ảnh, ăn local food..."*). Frontend gửi request này đến backend API.
2. **Kiểm tra & Chuẩn hóa:** Lambda kiểm tra input (địa điểm, số ngày, ngân sách...). Nếu input còn thiếu, backend có thể yêu cầu model đặt câu hỏi follow-up thay vì tạo lịch trình ngay.
3. **Truy xuất tri thức (Retrieve):** Backend gọi Amazon Bedrock Knowledge Bases để lấy thông tin chính xác về Hội An từ S3 và đưa các đoạn dữ liệu này vào prompt để làm giàu ngữ cảnh (context).
4. **Sinh phản hồi (Generate):** Backend gọi Amazon Bedrock để tạo lịch trình có cấu trúc (Day 1, Day 2, Morning, Afternoon, Evening) kèm lý do gợi ý.
5. **Tích hợp bản đồ:** Nếu response chứa địa điểm cụ thể, backend gọi Amazon Location Service để lấy dữ liệu tọa độ phục vụ hiển thị trực quan trên bản đồ frontend.
6. **Lưu trữ & Ghi log:** Hệ thống lưu lại conversation history vào DynamoDB, ghi nhận logs/metrics về CloudWatch và trả phản hồi cuối cùng về cho frontend.

---

# Các yếu tố vận hành thực tế

## Lớp Bảo vệ (Amazon Bedrock Guardrails)
Hệ thống có thể gặp những input không phù hợp, prompt injection, hoặc yêu cầu ngoài phạm vi. **Amazon Bedrock Guardrails** được cấu hình như một lớp kiểm soát để đánh giá user input và model response theo các chính sách:
- Content filters (Bộ lọc nội dung độc hại).
- Denied topics (Từ chối các chủ đề nhạy cảm về chính trị, y tế, pháp lý).
- Word filters & Sensitive information filters (Ẩn thông tin cá nhân).
- Contextual grounding checks (Kiểm tra câu trả lời có bịa đặt hay không).

## Quản lý chi phí (Cost Control)
Mỗi request đến model đều tạo chi phí theo token. Để tối ưu ngay từ giai đoạn MVP:
- Giới hạn độ dài tối đa của user input.
- Kiểm soát tham số `max_tokens` của output.
- Cache các câu trả lời cho câu hỏi phổ biến.
- Tuyệt đối không cho frontend gọi trực tiếp model mà phải qua backend kiểm soát để áp dụng rate limit.

## Giám sát (Observability) & Feedback Loop
- **CloudWatch:** Theo dõi số lượng request thành công/thất bại, thời gian xử lý trung bình (latency), lỗi gọi Bedrock hoặc lỗi kết nối giữa các service.
- **Feedback Loop:** Cho phép người dùng đánh giá "hữu ích" / "không hữu ích" kèm lý do. Dữ liệu này lưu vào DynamoDB để phục vụ việc tinh chỉnh prompt và cập nhật kiến thức (knowledge base) sau này.

---

# Tư duy thiết kế MVP và các lỗi thường gặp

Khi thiết kế ứng dụng Generative AI, người học rất dễ mắc phải hai sai lầm lớn:

1. **Xem prompt là toàn bộ hệ thống:** Prompt chỉ là một phần nhỏ. Một hệ thống tốt cần kiểm soát dữ liệu đầu vào, giới hạn phạm vi, có logging và cơ chế fallback. Nếu knowledge base không có dữ liệu, hệ thống nên trả lời *"mình chưa có đủ dữ liệu"* thay vì cố bịa ra câu trả lời.
2. **Nhồi nhét quá nhiều tính năng (Scope Creep):** Đưa tính năng booking, đặt vé, thanh toán, hệ thống giới thiệu cá nhân hóa sâu... vào phiên bản đầu tiên sẽ làm kiến trúc quá tải.

### Chiến lược thu hẹp phạm vi cho MVP:
- **Địa lý:** Giới hạn dữ liệu ở 3 thành phố lớn: Đà Nẵng, Hội An và Hà Nội để đảm bảo dữ liệu sạch và chuẩn hóa trước khi mở rộng.
- **Ngôn ngữ:** Hỗ trợ Tiếng Việt và Tiếng Anh. Lưu ý rằng đa ngôn ngữ không chỉ là dịch thuật, mà cần chuẩn bị tri thức phù hợp với hành vi du lịch của từng nhóm khách hàng quốc tế.
- **Bảo mật (Security):** Sử dụng **Amazon Cognito** hoặc **Amplify Auth** để quản lý định danh. Giai đoạn MVP tránh lưu trữ các thông tin nhạy cảm như số hộ chiếu, giấy tờ cá nhân hay thông tin thẻ thanh toán.

---

# Hướng phát triển và mở rộng tương lai

Sau khi xây dựng thành công hệ thống chạy ổn định bằng giao diện AWS Console, lộ trình nâng cấp hệ thống bao gồm:

- **Infrastructure as Code (IaC):** Chuyển toàn bộ tài nguyên (S3, Lambda, Bedrock, DynamoDB...) sang quản lý bằng mã nguồn với **Terraform**, **AWS CDK** hoặc **CloudFormation**.
- **Tích hợp CI/CD:** Thiết lập các pipeline tự động (GitHub Actions, GitLab CI/CD hoặc AWS CodePipeline) chia làm 3 luồng:
  - Pipeline cập nhật frontend web qua Amplify.
  - Pipeline cập nhật và tự động tái nạp dữ liệu tri thức (data ingestion) trên S3 vào Knowledge Base.
  - Pipeline kiểm thử và triển khai mã nguồn backend Lambda.
- **Tính năng nâng cao:** Xây dựng admin portal quản lý điểm đến, xuất lịch trình thành file PDF, tích hợp dữ liệu thời tiết và sự kiện địa phương theo thời gian thực.

---

# Kết luận

Xây dựng một "Vietnam AI Travel Assistant" là một bài thực hành xuất sắc cho người học AWS vì nó liên kết chặt chẽ nhiều mảng kiến thức: frontend, serverless backend, Generative AI, RAG, database và monitoring.

Mô hình này giúp người học bước ra khỏi tư duy của một "demo chatbot" dạng hộp đen (black box) để tiếp cận gần hơn với môi trường doanh nghiệp thực tế – nơi dữ liệu phải được kiểm duyệt, chi phí phải được tối ưu, vận hành phải được giám sát và an toàn thông tin luôn là ưu tiên hàng đầu.

<h2 style="text-align:center;">Sơ đồ kiến trúc đề xuất</h2>

![Photo](/images/3-Blog/diagram2.jpg)

**Link bài viết tham khảo** : https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201739207257706