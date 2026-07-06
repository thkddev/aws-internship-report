---
title: "Chuẩn bị môi trường"
date: 2026-07-03
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Trước khi triển khai dự án DMS, hãy đảm bảo các công cụ và tài khoản sau đây đã sẵn sàng.

#### 1. Tài khoản & Quyền truy cập

- **Tài khoản AWS** có đủ quyền để tạo IAM Role, Lambda, S3, DynamoDB, Cognito, CloudFront, SQS, GuardDuty, SNS, CloudWatch và CDK Bootstrap.
- **AWS CLI** đã được cấu hình với profile có các quyền cần thiết.

#### 2. Công cụ cần thiết

| Công cụ | Phiên bản | Mục đích |
|---|---|---|
| **Node.js** | >= 20.x | Chạy CDK và build Lambda |
| **npm** | >= 10.x | Quản lý package |
| **AWS CDK CLI** | >= 2.x | Triển khai hạ tầng |
| **Git** | Bất kỳ | Clone repository |

Cài đặt CDK toàn cục nếu chưa có:
```bash
npm install -g aws-cdk
```

Kiểm tra phiên bản:
```bash
node --version
npm --version
cdk --version
aws --version
```

#### 3. Bootstrap CDK (Chỉ cần làm lần đầu)

CDK Bootstrap tạo S3 bucket và IAM role cần thiết trong tài khoản AWS để CDK lưu artifact triển khai.

```bash
cdk bootstrap aws://<ACCOUNT_ID>/<REGION>
```

Ví dụ:
```bash
cdk bootstrap aws://123456789012/ap-southeast-1
```

#### 4. Clone Repository

```bash
git clone <your-repository-url>
cd DMS
npm install
```

#### 5. Cấu hình biến môi trường

Copy file môi trường mẫu và điền thông tin của bạn:
```bash
cp .env.example .env
```

File `.env.example` chứa các biến cần thiết:
```
# Email nhận cảnh báo CloudWatch (chi phí / lỗi)
DMS_ALERT_EMAIL=your-email@example.com
```

#### 6. Build Lambda Functions

Trước khi deploy, cần biên dịch code TypeScript của Lambda:
```bash
cd aws/functions
npm install
npm run build
cd ../..
```

#### 7. Kiểm tra cấu trúc dự án

Đảm bảo cấu trúc dự án như sau trước khi deploy:
```
DMS/
├── aws/
│   ├── functions/        # Lambda handlers (TypeScript)
│   │   └── dist/         # ← Phải tồn tại sau khi npm run build
│   └── infrastructure/   # AWS CDK stack (TypeScript)
├── frontend/             # React + Vite frontend
├── contracts/            # OpenAPI schemas
└── docs/                 # Data model & decisions
```