---
title: "Deploy Infrastructure with AWS CDK"
date: 2026-07-03
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This step deploys the entire DMS infrastructure to AWS using a single CDK command. The CDK stack provisions all services automatically.

#### What Gets Deployed

The `DmsStack` creates the following resources in order:

**1. Amazon Cognito — User Pool**
- Pool name: `dms-dev`
- Sign-in by email only, self-registration disabled
- 3 user groups: `EMPLOYEE` (precedence 30), `DEPARTMENT_ADMIN` (precedence 20), `SYSTEM_ADMIN` (precedence 10)
- Password policy: minimum 12 characters, requires digits, uppercase, lowercase, and symbols
- Token validity: Access/ID token = 60 minutes, Refresh token = 7 days

**2. Amazon S3 — Three Private Buckets**
- `FrontendBucket` — hosts the compiled React SPA
- `QuarantineBucket` — receives all file uploads first; GuardDuty scans here
- `DocumentsBucket` — stores only clean, approved documents
- All buckets block public access; CORS configured on QuarantineBucket for PUT operations

**3. Amazon DynamoDB — Single Table**
- Table name: `dms-dev`
- Billing: PAY_PER_REQUEST (no capacity planning needed)
- Encryption: AWS_MANAGED
- Point-In-Time Recovery (PITR): enabled
- TTL attribute: `expiresAtEpoch` (for auto-expiring UploadIntent records)
- 4 Global Secondary Indexes (GSI1–GSI4) for various query patterns

**4. AWS Lambda — 11 Functions (Node.js 22, ARM64)**

| Handler | Purpose |
|---|---|
| `me` | Get current user profile |
| `documents` | List documents for a user/department |
| `document-detail` | Get a single document with versions |
| `document-audit-events` | Get audit log for a document |
| `upload-intents` | Create presigned PUT URL to QuarantineBucket |
| `download-intents` | Create presigned GET URL from DocumentsBucket |
| `document-sharing` | Share/revoke document access |
| `admin-users` | Create/manage users in Cognito |
| `analytics` | Aggregate storage and activity statistics |
| `process-upload` | SQS consumer: move clean files, write DynamoDB |
| `process-upload-dlq` | Handle failed upload messages from Dead Letter Queue |

**5. Amazon SQS — Upload Queue**
- Main queue: 6-minute visibility timeout, 4-day retention
- Dead Letter Queue: 14-day retention, triggers after 10 failed receive attempts
- S3 QuarantineBucket triggers SQS on every `s3:ObjectCreated:Put` under the `quarantine/` prefix

**6. GuardDuty Malware Protection Plan**
- Dedicated IAM role grants GuardDuty access to scan QuarantineBucket
- Files are tagged `CLEAN` or `THREAT_FOUND` after scanning

**7. Amazon API Gateway — REST API**
- Cognito Authorizer validates JWT on all protected routes
- Routes map to Lambda functions by HTTP method and path

**8. Amazon CloudFront — CDN for Frontend**
- Serves the React SPA from `FrontendBucket` via CloudFront OAC

**9. CloudWatch + SNS Alerts**
- Log groups with 2-week retention (dev) or 3-month retention (production)
- SNS topic sends email alerts for cost anomalies or Lambda errors

#### Deploy Command

Navigate to the infrastructure directory and run:

```bash
cd aws/infrastructure
npm install
npx cdk deploy -c environment=dev -c alertEmail=your-email@example.com
```

Or use the environment variable:
```bash
DMS_ALERT_EMAIL=your-email@example.com npx cdk deploy -c environment=dev
```

{{% notice warning %}}
After deployment, AWS will send a **subscription confirmation email** to your alert address. You must click **"Confirm subscription"** in that email to activate SNS notifications.
{{% /notice %}}

#### Verify Deployment

After `cdk deploy` completes, check the **Outputs** section in the terminal. You should see:
- `UserPoolId` — the Cognito User Pool ID
- `UserPoolClientId` — the Cognito App Client ID
- `ApiUrl` — the API Gateway endpoint URL
- `CloudFrontUrl` — the frontend URL

You can also verify in the **AWS Console**:
1. **CloudFormation** → Stack `DmsStack-dev` → Status: `CREATE_COMPLETE`
2. **DynamoDB** → Table `dms-dev` exists with 4 GSIs
3. **S3** → Three buckets created
4. **Lambda** → 11 functions visible
5. **Cognito** → User pool `dms-dev` with 3 groups