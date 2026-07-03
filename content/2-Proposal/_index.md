---
title: "Proposal"
date: 2026-07-03
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Internal Document Management System (DMS) using AWS Serverless

### 1. Executive Summary  
The **Document Management System (DMS)** project is designed to build a secure and automated system for storing, authorizing, and managing the lifecycle of internal documents. By fully leveraging the **AWS Serverless** ecosystem (Cognito, API Gateway, Lambda, S3, DynamoDB, SQS), the system optimizes operational costs (pay-as-you-go) while ensuring enterprise-level scalability and security.

### 2. Problem Statement  
**Current Issues:**  
Storing and sharing internal documents via traditional platforms presents multiple challenges: difficulty in tracing version history, lack of granular access control mechanisms, high risk of sensitive data leakage, and the significant cost of maintaining physical servers 24/7 despite irregular usage patterns.

**Proposed Solution:**  
Build a 100% Serverless architecture. Specifically:
- Use **Amazon S3** as the document storage space with versioning and encryption enabled.
- Use **DynamoDB** to store metadata (creator, timestamp, edit history) with millisecond latency.
- Handle file upload processes asynchronously using **Amazon SQS** to avoid bottlenecks during high concurrent user interactions.
- Implement identity and access management via **Amazon Cognito**.

**Return on Investment (ROI):**  
The system requires zero fixed costs for server management and automatically scales with traffic. The estimated maintenance cost during the development phase is less than $2/month thanks to the AWS Free Tier.

### 3. Solution Architecture  

The system utilizes the following core services to build a highly secure and performant data flow:

**AWS Services Used:**  
- **Amazon Cognito**: User authentication and JWT Token issuance.  
- **Amazon API Gateway**: The single entry point (REST API) for the frontend, directly integrated with Cognito Authorizer.  
- **AWS Lambda**: Business logic processing (Document CRUD, access authorization).  
- **Amazon S3**: Secure storage for physical document files (Private S3 Bucket).  
- **Amazon DynamoDB**: NoSQL database for storing document metadata.  
- **Amazon SQS**: Asynchronous queue processing for large file uploads or malware scanning tasks.  
- **Amazon CloudWatch & GuardDuty**: System log monitoring and continuous threat detection.

### 4. Technical Implementation  

The project is implemented with the following technical requirements:
- **Frontend**: A simple Web Application (SPA) using React/Next.js communicating with AWS via REST APIs.
- **Backend (Serverless)**: Lambda functions written in Python/Node.js. Utilizing the boto3 SDK to interact with S3 and DynamoDB.
- **Security**: Applying IAM Roles with the *Principle of Least Privilege* for each individual Lambda function.

### 5. Roadmap & Milestones  
The project spans across the last 4 weeks of the internship:
- **Week 9**: Business requirement analysis, system architecture design, and data flow modeling.
- **Week 10**: Network setup, IAM authorization, and Amazon Cognito configuration. Developing the login/registration flow.
- **Week 11**: Deploying the Serverless Backend (Lambda, API Gateway, DynamoDB, S3) and integrating SQS for asynchronous upload processing.
- **Week 12**: Comprehensive testing (Function Test, Load Test, Security Test with GuardDuty), architecture optimization, and handover documentation.

### 6. Budget Estimation  
Expected deployment costs are extremely low due to the Pay-as-you-go model.  
- **AWS Lambda**: ~$0.00 (within the free tier of 1 million requests/month).  
- **Amazon S3**: ~$0.15 for storage and outbound data transfer.  
- **DynamoDB**: ~$0.00 (within the 25GB free tier).  
- **API Gateway & Cognito**: Free for low internal user volume.  
- **Estimated Total Budget**: Under $1/month for the testing environment and roughly under $10/month for a medium-scale production environment.

### 7. Risk Assessment & Contingency
- **Data Leakage Risk:** High.  
  *Mitigation:* Server-side encryption (S3 SSE) and blocking all Public access to S3. Granting only temporary presigned-URLs to the client.
- **Large File Upload Bottleneck Risk:** Medium.  
  *Mitigation:* Handling uploads directly via S3 Presigned URLs and using SQS for Lambda to process metadata in the background instead of passing massive payloads through API Gateway.

### 8. Expected Outcomes  
- Deliver a digitized, secure, and easily traceable document management system for the company.
- Eliminate document fragmentation across disjointed platforms.
- Master the design and implementation of Enterprise applications using AWS Serverless Architecture.