---
title: "Dọn dẹp tài nguyên"
date: 2026-07-03
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Khi hoàn thành workshop, hãy thực hiện các bước sau để xóa toàn bộ tài nguyên AWS và tránh phát sinh chi phí không cần thiết.

{{% notice warning %}}
**Quan trọng:** Các bước dọn dẹp bên dưới sẽ xóa vĩnh viễn mọi dữ liệu bao gồm tài liệu đã tải lên, bản ghi DynamoDB và người dùng Cognito. Hãy đảm bảo bạn đã sao lưu những gì cần giữ lại trước khi tiến hành.
{{% /notice %}}

#### 1. Xóa CDK Stack

Chạy lệnh sau từ thư mục `aws/infrastructure`:

```bash
cd aws/infrastructure
npx cdk destroy -c environment=dev
```

CDK sẽ hỏi xác nhận:
```
Are you sure you want to delete: DmsStack-dev (y/n)? y
```

Lệnh này sẽ tự động xóa:
- Cả 3 S3 bucket (FrontendBucket, QuarantineBucket, DocumentsBucket) và toàn bộ nội dung bên trong
- Bảng DynamoDB `dms-dev` và toàn bộ dữ liệu
- Tất cả 11 Lambda function và CloudWatch Log Group của chúng
- REST API trên API Gateway
- Cognito User Pool và tất cả người dùng
- SQS queue (UploadQueue và DeadLetterQueue)
- CloudFront distribution
- SNS topic và các email subscription
- GuardDuty Malware Protection Plan và IAM role

{{% notice warning %}}
Ở môi trường `production`, S3 bucket và DynamoDB table sử dụng `RemovalPolicy.RETAIN` để tránh mất dữ liệu ngoài ý muốn. Bạn sẽ cần xóa thủ công những tài nguyên đó trên AWS Console nếu muốn.
{{% /notice %}}

#### 2. Xác minh việc xóa trên AWS Console

Sau khi lệnh destroy hoàn thành, kiểm tra lại để đảm bảo tài nguyên đã bị xóa:

1. **CloudFormation** → xác nhận stack `DmsStack-dev` không còn tồn tại
2. **S3** → xác nhận 3 bucket DMS đã biến mất
3. **DynamoDB** → xác nhận bảng `dms-dev` đã bị xóa
4. **Lambda** → xác nhận các function DMS đã được gỡ bỏ
5. **Cognito** → xác nhận user pool `dms-dev` đã bị xóa

#### 3. Xóa CDK Bootstrap (Tùy chọn)

Nếu bạn muốn dọn dẹp hoàn toàn và xóa cả CDK bootstrap stack:

```bash
aws cloudformation delete-stack --stack-name CDKToolkit
```

{{% notice warning %}}
Chỉ thực hiện bước này nếu bạn không dùng CDK cho bất kỳ dự án nào khác trong tài khoản AWS và vùng (region) này.
{{% /notice %}}

#### 4. Kiểm tra các khoản phí còn lại

Sau khi dọn dẹp, xác minh trong **AWS Billing → Cost Explorer** rằng không có khoản phí liên quan đến DMS xuất hiện trong chu kỳ thanh toán tiếp theo. Các mục thường cần kiểm tra:
- Phí lưu trữ S3
- Đơn vị đọc/ghi DynamoDB
- Số lần gọi Lambda
- Lưu trữ log CloudWatch
- Phí quét GuardDuty

#### Tổng kết

Xin chúc mừng bạn đã hoàn thành DMS Workshop! Bạn đã thực hiện thành công:

- Triển khai một hệ thống Document Management System hoàn toàn serverless trên AWS bằng CDK
- Cấu hình phân quyền theo vai trò với Amazon Cognito
- Xây dựng pipeline upload bảo mật với S3 Presigned URL và quét malware bằng GuardDuty
- Thiết kế DynamoDB single-table với 4 GSI để query hiệu quả
- Thiết lập CloudWatch logging, X-Ray tracing và cảnh báo chi phí qua SNS
- Xác minh toàn bộ luồng upload, quét và tải xuống tài liệu hoạt động đúng