---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Amazon Bedrock Baseline Architecture in an AWS Landing Zone: Deployment Platform for Enterprise Generative AI

Building an AI application is not overly difficult. However, when an enterprise has dozens of AI applications running concurrently, the challenge is no longer just connecting to an AI model, but managing the entire AI system securely, stably, and with ease of operation.

Through the **Amazon Bedrock Baseline Architecture in an AWS Landing Zone**, AWS proposes a reference architecture to help enterprises build a Generative AI platform in a centralized manner, ensuring governance, security, and long-term scalability.

![Amazon Bedrock Architecture](../../images/bedrock-architecture.png)

## Why is a standard architecture for Generative AI necessary?

In the experimental phase, each development team might build their own AI applications and connect directly to Amazon Bedrock.
However, as the number of applications grows, enterprises face several challenges:
- Difficulty managing access permissions.
- Difficulty tracking AI usage costs.
- Difficulty controlling data and traffic.
- Difficulty applying unified security policies.
- Difficulty performing auditing and compliance.

## Proposed Architecture

- **Service Network Account:** Manages network connections, controls AI access, manages security policies, orchestrates traffic across accounts, and centralizes all AI traffic at a single point.
- **Generative AI Account:** Deploys Amazon Bedrock, Bedrock Knowledge Bases, Bedrock Guardrails, Foundation Models, and related AI services; enables centralized governance, cost tracking, and operational standardization.
- **Workload Accounts:** Deploys customer chatbots, internal assistants, document processing, RAG applications, and business applications; accesses AI through a centralized control layer rather than connecting directly to Bedrock.

## Highlights of the Architecture

- Separates the AI Platform Account and Workload Account.
- All AI traffic is centrally controlled via VPC Lattice.
- Amazon Bedrock is deployed as a shared service for the entire organization.
- Integrates Bedrock Guardrails to enhance content control and security.
- Aligns with the multi-account model and AWS Landing Zone.

## Conclusion

Amazon Bedrock Baseline Architecture is not simply a technical architecture but a Generative AI governance model at the enterprise level.

By centralizing AI resource management, controlling traffic, enhancing security, and standardizing operational processes, AWS helps enterprises solve the three biggest challenges when deploying AI:
1. Security.
2. Governance.
3. Scalability.

## Recommendations

- **For enterprises testing AI:** Start small, build governance early, standardize permissions.
- **For enterprises scaling AI:** Separate the AI Account, centrally manage AI resources, track usage costs.
- **For large-scale enterprises:** Build a shared AI Platform, deploy a Landing Zone, establish centralized governance, and set up AI Governance and AI Security.

**Reference:** [Amazon Bedrock baseline architecture in an AWS Landing Zone](https://aws.amazon.com/blogs/architecture/amazon-bedrock-baseline-architecture-in-an-aws-landing-zone/)
