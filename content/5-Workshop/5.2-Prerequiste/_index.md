---
title: "Prerequisites"
date: 2026-07-03
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Before deploying the DMS project, make sure the following tools and accounts are ready.

#### 1. Required Accounts & Access

- **AWS Account** with sufficient permissions to create IAM roles, Lambda, S3, DynamoDB, Cognito, CloudFront, SQS, GuardDuty, SNS, CloudWatch, and CDK Bootstrap resources.
- **AWS CLI** configured with a profile that has the required permissions.

#### 2. Required Tools

| Tool | Version | Purpose |
|---|---|---|
| **Node.js** | >= 20.x | Run CDK and Lambda build |
| **npm** | >= 10.x | Package management |
| **AWS CDK CLI** | >= 2.x | Deploy infrastructure |
| **Git** | Any | Clone the repository |

Install CDK globally if not already installed:
```bash
npm install -g aws-cdk
```

Verify versions:
```bash
node --version
npm --version
cdk --version
aws --version
```

#### 3. Bootstrap CDK (First-time only)

CDK Bootstrap creates the required S3 bucket and IAM roles in your AWS account for CDK to store deployment artifacts.

```bash
cdk bootstrap aws://<ACCOUNT_ID>/<REGION>
```

Example:
```bash
cdk bootstrap aws://123456789012/ap-southeast-1
```

#### 4. Clone the Repository

```bash
git clone <your-repository-url>
cd DMS
npm install
```

#### 5. Configure Environment Variables

Copy the example environment file and fill in your values:
```bash
cp .env.example .env
```

The `.env.example` file contains the required variables:
```
# Alert email for CloudWatch cost/error notifications
DMS_ALERT_EMAIL=your-email@example.com
```

#### 6. Build Lambda Functions

Before deploying, compile the TypeScript Lambda code:
```bash
cd aws/functions
npm install
npm run build
cd ../..
```

#### 7. Verify Project Structure

Make sure the project structure looks like this before deploying:
```
DMS/
├── aws/
│   ├── functions/        # Lambda handlers (TypeScript)
│   │   └── dist/         # ← Must exist after npm run build
│   └── infrastructure/   # AWS CDK stack (TypeScript)
├── frontend/             # React + Vite frontend
├── contracts/            # OpenAPI schemas
└── docs/                 # Data model & decisions
```

{{% notice warning %}}
Make sure the `aws/functions/dist/` directory exists before running `cdk deploy`. If it does not exist, Lambda code will not be found and the deployment will fail.
{{% /notice %}}