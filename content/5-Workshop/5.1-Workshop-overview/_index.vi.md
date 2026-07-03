---
title: "Giới thiệu"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### DMS Project là gì?

**Document Management System (DMS)** là dự án cuối kỳ của chương trình thực tập, tổng hợp toàn bộ kiến thức về AWS Serverless vào một hệ thống hoàn chỉnh, chuẩn production.

Hệ thống giải quyết các bài toán thực tế trong doanh nghiệp:
- Lưu trữ tài liệu tập trung, bảo mật, có lịch sử phiên bản
- Phân quyền truy cập theo vai trò (upload, xem, chia sẻ)
- Pipeline upload bất đồng bộ xử lý file lớn không bị nghẽn cổ chai
- Audit trail bất biến cho mọi thao tác với tài liệu
- Tự động quét malware mỗi khi có file được tải lên

#### Sơ đồ kiến trúc hệ thống

```
[React Frontend] → [CloudFront CDN]
        ↓
[Cognito] ← Xác thực JWT
        ↓
[API Gateway + Cognito Authorizer]
        ↓
[Các Lambda Function]
   ├── documents         → [DynamoDB]
   ├── document-detail   → [DynamoDB]
   ├── upload-intents    → [S3 QuarantineBucket (Presigned URL)]
   ├── download-intents  → [S3 DocumentsBucket (Presigned URL)]
   ├── document-sharing  → [DynamoDB]
   ├── document-audit-events → [DynamoDB]
   ├── admin-users       → [Cognito User Pool]
   ├── analytics         → [DynamoDB]
   └── me                → [DynamoDB / S3]

[S3 QuarantineBucket]
   → [S3 Event Notification] → [SQS UploadQueue]
   → [GuardDuty Malware Protection] (tag: CLEAN / THREAT_FOUND)
   → [Lambda process-upload]
       ├── CLEAN → chuyển sang [S3 DocumentsBucket] + ghi [DynamoDB]
       └── THREAT_FOUND → loại bỏ + ghi [DynamoDB AuditLog]

[CloudWatch Logs] ← tất cả Lambda function
[SNS Alert] ← CloudWatch Alarms (chi phí / lỗi hệ thống)
```

#### Các dịch vụ AWS sử dụng

| Dịch vụ | Vai trò trong hệ thống |
|---|---|
| **Amazon Cognito** | User pool, xác thực JWT, 3 nhóm người dùng (EMPLOYEE / DEPT_ADMIN / SYS_ADMIN) |
| **API Gateway** | REST API, Cognito Authorizer, phân quyền IAM từng endpoint |
| **AWS Lambda** | 11 handler serverless, Node.js 22, ARM64, X-Ray tracing |
| **Amazon S3** | QuarantineBucket (khu kiểm dịch upload), DocumentsBucket (lưu trữ chính thức) |
| **Amazon DynamoDB** | Single-table design, 4 GSI, billing PAY_PER_REQUEST, bật PITR |
| **Amazon SQS** | Hàng đợi upload (visibility timeout 6 phút) + Dead Letter Queue (giữ 14 ngày) |
| **GuardDuty** | Malware Protection Plan quét toàn bộ file tải vào quarantine |
| **Amazon CloudFront** | CDN phân phối React SPA từ S3 |
| **AWS CDK** | Triển khai hạ tầng hoàn toàn bằng TypeScript (IaC) |
| **CloudWatch + SNS** | Lưu log, cảnh báo metric, gửi email thông báo |

#### Các quyết định thiết kế quan trọng

- **Presigned URL khi upload**: File không bao giờ đi qua API Gateway (tránh giới hạn 10MB payload). Frontend nhận Presigned PUT URL và upload thẳng lên S3 QuarantineBucket.
- **Upload qua Quarantine trước**: Mọi file đều được giữ trong QuarantineBucket cho đến khi GuardDuty xác nhận an toàn. Chỉ file được tag `CLEAN` mới được chuyển sang DocumentsBucket chính thức.
- **DynamoDB single-table**: Tất cả entity (Document, Version, Share, AuditLog, UploadIntent, UserProfile) dùng chung một bảng với composite key — tiết kiệm chi phí và đơn giản hoá quản lý.
- **Audit log bất biến**: Bản ghi AuditLog chỉ được thêm mới (append-only), dùng điều kiện `attribute_not_exists` để ngăn sửa xóa.
- **IAM tối thiểu quyền**: Mỗi Lambda function có IAM role riêng, chỉ được cấp đúng quyền cần thiết — không dùng role chung.
