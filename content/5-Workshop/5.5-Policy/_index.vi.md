---
title: "Bảo mật & Giám sát"
date: 2026-07-03
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Hệ thống DMS áp dụng nhiều lớp bảo mật và giám sát để bảo vệ dữ liệu và phát hiện bất thường theo thời gian thực.

#### 1. IAM — Nguyên tắc Quyền Tối Thiểu

Mỗi Lambda function có **IAM role riêng** với chỉ các quyền cần thiết:

| Lambda | Quyền được cấp |
|---|---|
| `documents` | `dynamodb:Query` trên bảng Documents (chỉ đọc) |
| `upload-intents` | `dynamodb:PutItem` + `s3:PutObject` chỉ trên QuarantineBucket |
| `download-intents` | `dynamodb:GetItem` + `s3:GetObject` chỉ trên DocumentsBucket |
| `admin-users` | `cognito-idp:AdminCreateUser`, `AdminAddUserToGroup`,... |
| `process-upload` | `s3:GetObject` từ Quarantine + `s3:PutObject` vào Documents + `dynamodb:PutItem` |

Không có Lambda function nào được cấp quyền `*` hay quyền vượt phạm vi cần thiết.

#### 2. S3 — Bucket Private & Presigned URL

- Tất cả 3 S3 bucket đều bật **Block All Public Access**.
- Tài liệu **không bao giờ truy cập được** qua URL công khai.
- Quyền truy cập chỉ được cấp thông qua **Presigned URL có thời hạn** do Lambda tạo ra:
  - Upload: Presigned PUT URL (hết hạn sau ~15 phút)
  - Download: Presigned GET URL (hết hạn sau ~5 phút)
- Mã hóa server-side (SSE-S3) được áp dụng cho tất cả object được lưu trữ.

#### 3. GuardDuty — Malware Protection

Tài nguyên `MalwareProtectionPlan` quét mọi file được tải lên QuarantineBucket:

1. File đến prefix `quarantine/` qua Presigned URL.
2. GuardDuty tự động quét file.
3. GuardDuty gắn tag vào S3 object:
   - `GuardDutyMalwareScanStatus: CLEAN` → an toàn để xử lý tiếp
   - `GuardDutyMalwareScanStatus: THREAT_FOUND` → loại bỏ, ghi vào audit log
4. Lambda `process-upload` đọc tag trước khi chuyển file.

Để xem kết quả quét:
- Vào **GuardDuty → Malware Protection → Protected buckets**
- Hoặc kiểm tra tag của S3 object trực tiếp trên Console

#### 4. Cognito — Phân quyền theo vai trò

Ba nhóm người dùng quy định các mức quyền khác nhau trong backend:

| Nhóm | Mức quyền truy cập |
|---|---|
| `EMPLOYEE` (ưu tiên 30) | Upload, xem và tải tài liệu của chính mình |
| `DEPARTMENT_ADMIN` (ưu tiên 20) | Quản lý tài liệu của phòng ban, chia sẻ với người dùng |
| `SYSTEM_ADMIN` (ưu tiên 10) | Toàn quyền — quản lý người dùng, xem analytics, truy cập mọi tài liệu |

Thông tin nhóm được nhúng vào JWT token và được Lambda xác nhận ở runtime.

#### 5. CloudWatch — Giám sát Log

Tất cả 11 Lambda function ghi structured log vào CloudWatch Log Groups:
- Lưu log: **2 tuần** cho dev, **3 tháng** cho production
- X-Ray tracing bật trên tất cả Lambda function để theo dõi request end-to-end

Để xem log:
1. Vào **CloudWatch → Log Groups**
2. Tìm `/aws/lambda/DmsStack-dev-<FunctionName>`
3. Dùng **Insights** để query: `fields @timestamp, @message | sort @timestamp desc | limit 50`

#### 6. SNS — Cảnh báo Chi phí & Lỗi

CloudWatch Alarm kết nối với SNS topic và gửi email tới `alertEmail` khi:
- Tỷ lệ lỗi Lambda vượt ngưỡng cho phép
- Chi phí ước tính hàng tháng vượt ngân sách

Để xác minh cảnh báo đang hoạt động:
1. Vào **SNS → Topics → DmsAlertTopic**
2. Xác nhận subscription email của bạn là `Confirmed`
3. Vào **CloudWatch → Alarms** để xem các alarm đang được định nghĩa
