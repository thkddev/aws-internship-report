---
title: "Introduction"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

{{% notice info %}}
**Source Code:** [Document-Management-System](https://github.com/thkddev/Document-Management-System.git)
{{% /notice %}}

#### What is the DMS Project?

The **Document Management System (DMS)** is the final project of the internship program, applying everything learned about AWS Serverless into a single, cohesive, production-grade system.

The system addresses real-world enterprise pain points:
- Secure, centralized document storage with version history
- Role-based access control for upload, view, and share operations
- Asynchronous upload pipeline that handles large files without bottlenecks
- Immutable audit trail for all document operations
- Automatic malware scanning on every uploaded file

#### Test Accounts & Access

**Website URL:** [https://dmt3lkrot0e6x.cloudfront.net/](https://dmt3lkrot0e6x.cloudfront.net/)

| Role | Username | Password |
|---|---|---|
| System Admin | admin@gmail.com | Admin12345@ |
| Employee | fin123@gmail.com | Test12345.@A |
| Department Admin | depart123@gmail.com | Test12345.@A |

*Note: This is an internal website, so the UI will not have a self-registration option. Accounts must be granted by the Admin.*

#### System Architecture Diagram

![System Architecture Diagram](../../images/architecturalcomplex.png)

#### AWS Services Used

| Service | Role in the System |
|---|---|
| **Amazon Cognito** | User pool, JWT auth, 3 user groups (EMPLOYEE / DEPT_ADMIN / SYS_ADMIN) |
| **API Gateway** | REST API, Cognito Authorizer, per-endpoint IAM authorization |
| **AWS Lambda** | 12 serverless handlers, Node.js 22, ARM64, X-Ray tracing |
| **Amazon S3** | QuarantineBucket (upload staging), DocumentsBucket (final storage) |
| **Amazon DynamoDB** | Single-table design, 4 GSIs, PAY_PER_REQUEST billing, PITR enabled |
| **Amazon EventBridge** | Event bus routing S3 events to processing Lambda functions |
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