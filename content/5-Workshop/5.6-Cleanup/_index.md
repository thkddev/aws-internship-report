---
title: "Clean Up Resources"
date: 2026-07-03
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

When you are done with the workshop, follow these steps to delete all AWS resources and avoid ongoing charges.

{{% notice warning %}}
**Important:** The cleanup steps below will permanently delete all data including uploaded documents, DynamoDB records, and Cognito users. Make sure you have backed up anything you need before proceeding.
{{% /notice %}}

#### 1. Destroy the CDK Stack

Run the following command from the `aws/infrastructure` directory:

```bash
cd aws/infrastructure
npx cdk destroy -c environment=dev
```

CDK will ask for confirmation:
```
Are you sure you want to delete: DmsStack-dev (y/n)? y
```

This command will automatically delete:
- All 3 S3 buckets (FrontendBucket, QuarantineBucket, DocumentsBucket) and their contents
- DynamoDB table `dms-dev` and all data
- All 11 Lambda functions and their CloudWatch Log Groups
- API Gateway REST API
- Cognito User Pool and all users
- SQS queues (UploadQueue and DeadLetterQueue)
- CloudFront distribution
- SNS topic and email subscriptions
- GuardDuty Malware Protection Plan and IAM role

{{% notice warning %}}
In `production` environment, S3 buckets and DynamoDB table use `RemovalPolicy.RETAIN` to prevent accidental data loss. You would need to delete those manually from the AWS Console if desired.
{{% /notice %}}

#### 2. Verify Deletion in AWS Console

After the destroy command completes, verify that resources have been removed:

1. **CloudFormation** → confirm `DmsStack-dev` stack no longer exists
2. **S3** → confirm the 3 DMS buckets are gone
3. **DynamoDB** → confirm table `dms-dev` is deleted
4. **Lambda** → confirm DMS functions are removed
5. **Cognito** → confirm user pool `dms-dev` is deleted

#### 3. Remove CDK Bootstrap (Optional)

If you want to fully clean up and remove the CDK bootstrap stack as well:

```bash
aws cloudformation delete-stack --stack-name CDKToolkit
```

{{% notice warning %}}
Only do this if you are not using CDK for any other projects in this AWS account and region.
{{% /notice %}}

#### 4. Check for Remaining Charges

After cleanup, verify in **AWS Billing → Cost Explorer** that no DMS-related charges appear in the next billing cycle. Common items to check:
- S3 storage charges
- DynamoDB read/write units
- Lambda invocations
- CloudWatch log storage
- GuardDuty scan charges

#### Summary

Congratulations on completing the DMS Workshop! You have successfully:

- Deployed a fully serverless Document Management System on AWS using CDK
- Configured role-based access control with Amazon Cognito
- Implemented a secure upload pipeline with S3 Presigned URLs and GuardDuty malware scanning
- Used a single-table DynamoDB design with 4 GSIs for efficient querying
- Set up CloudWatch logging, X-Ray tracing, and SNS cost alerts
- Verified the full end-to-end document upload, scan, and retrieval flow