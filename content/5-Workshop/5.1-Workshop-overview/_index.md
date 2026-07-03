---
title: "Introduction"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### What is the DMS Project?

The **Document Management System (DMS)** is the final project of the internship program, applying everything learned about AWS Serverless into a single, cohesive, production-grade system.

The system addresses real-world enterprise pain points:
- Secure, centralized document storage with version history
- Role-based access control for upload, view, and share operations
- Asynchronous upload pipeline that handles large files without bottlenecks
- Immutable audit trail for all document operations
- Automatic malware scanning on every uploaded file

#### System Architecture Diagram

```
[React Frontend] → [CloudFront CDN]
        ↓
[Cognito] ← JWT Authentication
        ↓
[API Gateway + Cognito Authorizer]
        ↓
[Lambda Functions]
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
   → [GuardDuty Malware Protection] (tags: CLEAN / THREAT_FOUND)
   → [Lambda process-upload]
       ├── CLEAN → move to [S3 DocumentsBucket] + write [DynamoDB]
       └── THREAT_FOUND → discard + [DynamoDB AuditLog]

[CloudWatch Logs] ← all Lambda functions
[SNS Alert] ← CloudWatch Alarms (cost / errors)
```

#### AWS Services Used

| Service | Role in the System |
|---|---|
| **Amazon Cognito** | User pool, JWT auth, 3 user groups (EMPLOYEE / DEPT_ADMIN / SYS_ADMIN) |
| **API Gateway** | REST API, Cognito Authorizer, per-endpoint IAM authorization |
| **AWS Lambda** | 11 serverless handlers, Node.js 22, ARM64, X-Ray tracing |
| **Amazon S3** | QuarantineBucket (upload staging), DocumentsBucket (final storage) |
| **Amazon DynamoDB** | Single-table design, 4 GSIs, PAY_PER_REQUEST billing, PITR enabled |
| **Amazon SQS** | Upload queue (6-min visibility timeout) + Dead Letter Queue (14-day retention) |
| **GuardDuty** | Malware Protection Plan scanning all quarantine uploads |
| **Amazon CloudFront** | CDN for React SPA frontend served from S3 |
| **AWS CDK** | Full IaC deployment in TypeScript |
| **CloudWatch + SNS** | Log retention, metric alarms, email notifications |

#### Key Design Decisions

- **Presigned URL for uploads**: Files never pass through API Gateway (avoids 10MB payload limit). The frontend gets a presigned PUT URL and uploads directly to S3 QuarantineBucket.
- **Quarantine-first upload**: All files are held in QuarantineBucket until GuardDuty clears them. Only `CLEAN`-tagged files are moved to the production DocumentsBucket.
- **Single-table DynamoDB**: All entities (Document, Version, Share, AuditLog, UploadIntent, UserProfile) share one table with composite keys for cost efficiency.
- **Immutable audit log**: AuditLog records are append-only, written with `attribute_not_exists` conditional PutItem to prevent tampering.
- **Least Privilege IAM**: Each Lambda function has its own IAM role with only the permissions it needs — no shared service roles.