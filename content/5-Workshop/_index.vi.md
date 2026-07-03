---
title: "Workshop"
date: 2026-07-03
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Hệ thống Quản lý Tài liệu (DMS) trên nền tảng AWS Serverless

#### Tổng quan

**Document Management System (DMS)** là ứng dụng cloud-native được xây dựng hoàn toàn trên hệ sinh thái **AWS Serverless**. Hệ thống cho phép người dùng nội bộ tải lên, quản lý phiên bản, chia sẻ và theo dõi lịch sử thao tác với tài liệu công ty — mà không cần quản lý bất kỳ máy chủ nào.

Hệ thống được triển khai qua **AWS CDK** (Infrastructure as Code) và tích hợp các dịch vụ AWS cốt lõi sau:

- **Amazon Cognito** — Xác thực người dùng và phân quyền theo vai trò (EMPLOYEE / DEPARTMENT_ADMIN / SYSTEM_ADMIN)
- **Amazon API Gateway** — Điểm vào REST API duy nhất, tích hợp Cognito Authorizer
- **AWS Lambda** — Xử lý logic nghiệp vụ serverless (Node.js 22, ARM64)
- **Amazon S3** — Lưu trữ tài liệu private (DocumentsBucket) và khu vực kiểm dịch khi upload (QuarantineBucket)
- **Amazon DynamoDB** — Thiết kế single-table với GSI index cho metadata, versioning, chia sẻ và audit log
- **Amazon SQS** — Hàng đợi xử lý upload bất đồng bộ với Dead Letter Queue
- **Amazon GuardDuty** — Quét malware cho các file được tải lên
- **Amazon CloudFront** — Phân phối CDN cho giao diện React frontend
- **Amazon CloudWatch** — Giám sát log tập trung và cảnh báo chi phí
- **Amazon SNS** — Gửi email cảnh báo khi chi phí bất thường hoặc hệ thống gặp lỗi

#### Kiến trúc hệ thống

Luồng dữ liệu hoạt động theo trình tự sau:

1. Người dùng đăng nhập qua giao diện React bằng **Cognito** và nhận về JWT token.
2. Mọi lời gọi API đều đi qua **API Gateway**, được xác thực JWT thông qua Cognito Authorizer.
3. Việc tải file lên sử dụng **Presigned URL** — frontend tải trực tiếp lên **QuarantineBucket** (S3), bỏ qua API Gateway để tránh giới hạn 10MB.
4. Một **S3 event notification** kích hoạt **SQS message** khi có file mới vào quarantine bucket.
5. **GuardDuty Malware Protection** quét file và gắn tag `CLEAN` hoặc `THREAT_FOUND`.
6. Một **Lambda** worker lắng nghe SQS queue, đọc tag kết quả quét GuardDuty, di chuyển file sạch sang **DocumentsBucket** và ghi metadata + bản ghi version vào **DynamoDB**.
7. Chia sẻ tài liệu, audit log, analytics và quản lý người dùng được xử lý bởi các **Lambda** function riêng biệt.

#### Nội dung Workshop

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường](5.2-Prerequiste/)
3. [Triển khai hạ tầng bằng AWS CDK](5.3-S3-vpc/)
4. [Cấu hình Backend và kiểm thử API](5.4-S3-onprem/)
5. [Bảo mật & Giám sát với GuardDuty và CloudWatch](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)