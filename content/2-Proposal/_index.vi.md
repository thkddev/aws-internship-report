---
title: "Bản đề xuất"
date: 2026-07-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Hệ thống Quản lý tài liệu nội bộ (DMS) trên nền tảng AWS Serverless

### 1. Tóm tắt dự án  
Dự án **Document Management System (DMS)** được thiết kế nhằm xây dựng một hệ thống lưu trữ, phân quyền và quản lý vòng đời tài liệu nội bộ một cách an toàn và tự động hóa. Bằng việc tận dụng hoàn toàn hệ sinh thái **AWS Serverless** (Cognito, API Gateway, Lambda, S3, DynamoDB, SQS), hệ thống giúp tối ưu hóa chi phí vận hành (pay-as-you-go) đồng thời đảm bảo khả năng mở rộng (scalability) và tính bảo mật cao theo tiêu chuẩn doanh nghiệp.  

### 2. Tuyên bố vấn đề  
**Vấn đề hiện tại:**  
Việc lưu trữ và chia sẻ tài liệu nội bộ qua các nền tảng truyền thống gặp nhiều bất cập: khó khăn trong việc truy xuất lịch sử, thiếu cơ chế phân quyền chi tiết, nguy cơ lộ lọt dữ liệu nhạy cảm cao, và chi phí duy trì server vật lý 24/7 rất tốn kém dù tần suất sử dụng không đồng đều.  

**Giải pháp:**  
Xây dựng một kiến trúc 100% Serverless. Trong đó:
- Dùng **Amazon S3** làm không gian lưu trữ tài liệu với cơ chế versioning và mã hóa.
- Dùng **DynamoDB** để lưu trữ metadata (người tạo, thời gian, lịch sử chỉnh sửa) với tốc độ phản hồi tính bằng mili-giây.
- Quá trình upload file được xử lý bất đồng bộ thông qua **Amazon SQS** để tránh nghẽn cổ chai khi có nhiều người dùng thao tác cùng lúc.
- Định danh và xác thực qua **Amazon Cognito**.

**Lợi ích (ROI):**  
Hệ thống không yêu cầu chi phí cố định cho máy chủ (Zero Server Management), tự động scale theo lượng truy cập. Ước tính chi phí duy trì hệ thống trong giai đoạn phát triển chưa tới 2 USD/tháng nhờ tận dụng AWS Free Tier.

### 3. Kiến trúc giải pháp  

Hệ thống sử dụng các dịch vụ cốt lõi sau để xây dựng luồng dữ liệu bảo mật và hiệu suất cao:

**Các dịch vụ AWS sử dụng:**  
- **Amazon Cognito**: Quản lý xác thực người dùng (Authentication) và cấp phát JWT Token.  
- **Amazon API Gateway**: Điểm giao tiếp duy nhất (REST API) cho frontend, tích hợp trực tiếp với Cognito Authorizer.  
- **AWS Lambda**: Xử lý logic nghiệp vụ (CRUD tài liệu, xác thực quyền truy cập).  
- **Amazon S3**: Lưu trữ các file tài liệu vật lý một cách an toàn (S3 Bucket Private).  
- **Amazon DynamoDB**: Cơ sở dữ liệu NoSQL lưu trữ metadata của tài liệu.  
- **Amazon SQS**: Xử lý hàng đợi bất đồng bộ cho các tác vụ tải lên file dung lượng lớn hoặc quét mã độc.
- **Amazon CloudWatch & GuardDuty**: Giám sát log hệ thống và dò tìm mối đe dọa liên tục.

### 4. Triển khai kỹ thuật  

Dự án được triển khai với các yêu cầu kỹ thuật:
- **Frontend**: Ứng dụng Web đơn giản (SPA) bằng React/Next.js giao tiếp với AWS qua REST API.
- **Backend (Serverless)**: Code Lambda bằng Python/Node.js. Dùng boto3 SDK để tương tác với S3 và DynamoDB.
- **Bảo mật**: Áp dụng IAM Role với tiêu chí *Principle of Least Privilege* (Quyền tối thiểu) cho từng function Lambda.

### 5. Lộ trình & Mốc triển khai  
Dự án được thực hiện xuyên suốt trong 4 tuần cuối của kỳ thực tập:
- **Tuần 9**: Phân tích yêu cầu nghiệp vụ, thiết kế kiến trúc hệ thống và luồng dữ liệu (Data flow).
- **Tuần 10**: Thiết lập mạng, phân quyền IAM, và cấu hình Amazon Cognito. Phát triển luồng đăng nhập/đăng ký.
- **Tuần 11**: Triển khai Backend Serverless (Lambda, API Gateway, DynamoDB, S3) và tích hợp SQS để xử lý upload bất đồng bộ.
- **Tuần 12**: Kiểm thử toàn diện (Function Test, Load Test, Security Test với GuardDuty), tối ưu hóa kiến trúc và viết tài liệu bàn giao.

### 6. Ước tính ngân sách  
Chi phí dự kiến khi triển khai thực tế rất thấp nhờ chính sách dùng đến đâu trả đến đó (Pay-as-you-go).  
- **AWS Lambda**: ~0.00 USD (nằm trong mức miễn phí 1 triệu request/tháng).  
- **Amazon S3**: ~0.15 USD cho lưu trữ và băng thông ra ngoài.  
- **DynamoDB**: ~0.00 USD (mức miễn phí 25GB lưu trữ).  
- **API Gateway & Cognito**: Miễn phí với lưu lượng người dùng nội bộ nhỏ.  
- **Tổng ngân sách dự kiến**: Dưới 1 USD/tháng cho môi trường thử nghiệm và khoảng dưới 10 USD/tháng cho môi trường thực tế quy mô vừa.

### 7. Đánh giá rủi ro & Dự phòng
- **Rủi ro rò rỉ dữ liệu (Data Leakage):** Mức độ cao.  
  *Khắc phục:* Mã hóa dữ liệu tại chỗ (S3 SSE) và chặn toàn bộ quyền truy cập Public vào S3. Chỉ cấp presigned-URL tạm thời cho client.
- **Rủi ro nghẽn cổ chai khi upload file lớn:** Mức độ trung bình.  
  *Khắc phục:* Xử lý upload trực tiếp qua S3 Presigned URL và dùng SQS để Lambda xử lý metadata trong nền thay vì gửi toàn bộ payload qua API Gateway.

### 8. Kết quả kỳ vọng  
- Hoàn thiện một hệ thống quản lý tài liệu số hóa, an toàn, dễ dàng truy xuất lịch sử cho công ty.
- Xóa bỏ tình trạng phân mảnh tài liệu.
- Làm quen và làm chủ hoàn toàn cách thiết kế một ứng dụng Enterprise bằng AWS Serverless Architecture.