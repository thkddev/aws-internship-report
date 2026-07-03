---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:
* Phát triển ứng dụng DMS (Frontend và Backend).
* Thiết lập AWS Infrastructure as Code (IaC) bằng AWS CDK.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2 | - Khởi tạo monorepo DMS - frontend React/Vite, aws/functions, aws/infrastructure, contracts và docs | 22/06/2026 | 22/06/2026 | |
| 3 | - Xây dựng React frontend - Dashboard tài liệu, filter, upload flow, protected route và document detail | 23/06/2026 | 23/06/2026 | |
| 4 | - Xây dựng AWS Lambda backend - /me, upload intents, documents, download intents, sharing và audit events | 24/06/2026 | 24/06/2026 | |
| 5 | - Xây dựng AWS CDK stack - Cognito, S3, DynamoDB, API Gateway, Lambda, CloudFront, SQS, SNS và GuardDuty | 25/06/2026 | 25/06/2026 | |
| 6 | - Chạy lint, typecheck, test, build, cdk synth và hoàn thiện báo cáo tuần 10 cho project DMS | 26/06/2026 | 26/06/2026 | |

### Kết quả đạt được tuần 10:
* Khởi tạo thành công cấu trúc dự án Monorepo giúp quản lý code Frontend, Backend và Infrastructure một cách thống nhất, chuyên nghiệp.
* Lập trình hoàn thiện giao diện React (Frontend) trực quan, có tính năng xác thực an toàn và dashboard quản lý tài liệu thân thiện.
* Phát triển thành công toàn bộ các API Backend Serverless bằng AWS Lambda để xử lý logic upload, download, phân quyền và ghi log (Audit).
* Triển khai toàn bộ hạ tầng (IaC) tự động thông qua AWS CDK, kết hợp SQS, SNS và GuardDuty đảm bảo tính sẵn sàng và bảo mật cao.
