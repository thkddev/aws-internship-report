---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Emailing Daily Delta Cost Reports in a Multi-Account Environment Using AWS Lambda and Cost Explorer

In an AWS multi-account environment, tracking costs is always a challenging task, especially when finance teams or management do not have direct access to the AWS Billing Console.

While researching cost management and automation on AWS, I read a blog post sharing how to automatically send daily delta cost reports in a multi-account environment. I find this solution highly practical because it increases "cost visibility" while significantly reducing manual operations in cloud cost management.

![Delta Cost Report Architecture](../../images/cost-report-architecture.png)

## Common Problems

AWS Organizations allows managing and consolidating the billing of multiple accounts within the same organization. However, cost reports are usually only available to those with billing access in the management account.
This leads to several issues:
- Finance teams or management lack AWS access.
- Sharing cost reports is often a manual process.
- Difficulty tracking cost changes across accounts.
- Lack of "cost visibility" within the organization.
- Cost management processes become more complex with multiple accounts like Production, Non-Production, or Sandbox.

## Core Solution

The solution uses AWS Lambda to automatically generate and email daily "delta cost" reports.
"Delta cost" refers to the cost difference between the current day and the previous day, helping to quickly detect abnormal fluctuations in cloud costs.
The system leverages native AWS services such as:
- AWS Lambda
- AWS Cost Explorer API
- Amazon EventBridge
- Amazon Simple Email Service (SES)

What I find interesting is that the entire architecture is serverless and based on managed services, requiring almost no infrastructure management.

## Architecture Overview

Workflow:
1. Amazon EventBridge triggers Lambda on a daily cron schedule.
2. Lambda calls the AWS Cost Explorer API to retrieve cost data from accounts in the organization.
3. Lambda calculates the percentage change in cost (delta) compared to the previous day.
4. The data is formatted into an HTML report.
5. Amazon SES sends the email to the list of recipients.

## Key Benefits

- Fully automates the cost reporting process.
- Makes information easily accessible to leadership and finance teams.
- Operates effectively in a multi-account environment.
- Supports increased cost visibility across the organization.
- Easy to deploy using a CloudFormation template.
- Can be extended to store report history in S3 for long-term analysis.

Additionally, this solution helps technical and finance teams collaborate more effectively in optimizing cloud costs following FinOps principles.

## Main Deployment Steps

- Download the CloudFormation template from the original blog post.
- Configure account IDs and email recipients.
- Verify emails in Amazon SES.
- Deploy the stack.
- Grant Billing permissions to the IAM Role.
- Configure an EventBridge rule with a cron expression.
Once completed, the system will automatically send daily cost reports via email.

## Personal Thoughts

I find this solution quite practical as it addresses a common problem for many teams using AWS: cloud costs increase, but they are not always tracked in time.

The great thing is that the architecture is compact, primarily using native AWS services like Lambda, EventBridge, and SES, so deployment is not overly complex. To me, this is also a clear example of how AWS uses automation to reduce manual tasks in system operations.

Beyond just sending cost reports, I think this model could be expanded by saving history to S3, creating monitoring dashboards, or incorporating alerts when costs rise abnormally.

**Reference:** [Email delta cost usage report in a multi-account organization using AWS Lambda](https://aws.amazon.com/vi/blogs/architecture/email-delta-cost-usage-report-in-a-multi-account-organization-using-aws-lambda/)
