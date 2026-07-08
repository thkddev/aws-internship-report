---
title: "Workshop"
date: 2026-07-03
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Document Management System (DMS) on AWS Serverless

#### Overview

The **Document Management System (DMS)** is a cloud-native application built entirely on the **AWS Serverless** ecosystem. It allows internal users to securely upload, version, share, and audit company documents — without managing any servers.

The system is deployed via **AWS CDK** (Infrastructure as Code) and integrates the following core AWS services:

- **Amazon Cognito** — User authentication and role-based access control (EMPLOYEE / DEPARTMENT_ADMIN / SYSTEM_ADMIN)
- **Amazon API Gateway** — Single REST API entry point with Cognito Authorizer
- **AWS Lambda** — Serverless business logic handlers (Node.js 22, ARM64)
- **Amazon S3** — Private document storage (DocumentsBucket) and upload quarantine (QuarantineBucket)
- **Amazon DynamoDB** — Single-table design with GSI indexes for metadata, versioning, sharing, and audit logs
- **Amazon EventBridge** — Serverless event bus routing S3 events to Lambda
- **Amazon GuardDuty** — Malware Protection scanning for uploaded files
- **Amazon CloudFront** — CDN delivery for the React frontend
- **Amazon CloudWatch** — Centralized log monitoring and cost alerting
- **Amazon SNS** — Email alerts for anomalous cost or system errors

#### Architecture

The data flow follows this pattern:

1. The user logs in via the React frontend using **Cognito** and receives a JWT token.
2. All API calls go through **API Gateway**, which validates the JWT via Cognito Authorizer.
3. File uploads use **Presigned URLs** — the frontend uploads directly to **QuarantineBucket** (S3), bypassing API Gateway.
4. **Amazon EventBridge** routes an `Object Created` event to Lambda when a new file lands in the quarantine bucket.
5. **GuardDuty Malware Protection** scans the file and tags it with `CLEAN` or `THREAT_FOUND`.
6. A **Lambda** worker triggered by EventBridge reads the GuardDuty scan tag, moves clean files to **DocumentsBucket**, and writes metadata + version records to **DynamoDB**.
7. Document sharing, audit logs, analytics, and user management are all handled by dedicated **Lambda** functions.

#### Workshop Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Deploy Infrastructure with AWS CDK](5.3-deploy-infrastructure/)
4. [Configure Backend & Test APIs](5.4-test-apis/)
5. [Security & Monitoring with GuardDuty and CloudWatch](5.5-security-monitoring/)
6. [Clean Up Resources](5.6-Cleanup/)