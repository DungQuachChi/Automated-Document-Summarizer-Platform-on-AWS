---
title: "Workshop"
date: 2026-07-14
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Project: Building a Secure Serverless AI Pipeline on AWS

This workshop is a DevOps and Cloud Engineering project that uses AI document summarizer as the vehicle to teach secure serverless architecture, Infrastructure as Code, CI/CD automation, and observability on AWS.

In this workshop build as a serverless document summarization platform. The platform accepts long-form text documents through a secured REST API and returns AI-generated summaries powered by Amazon Bedrock.

There are four core components that work together to process a request end to end: Amazon Cognito, Amazon API Gateway, AWS Lambda, and Amazon Bedrock. Each component has a distinct role in the request pipeline, from authentication through to AI inference.

+ **Amazon Cognito** — Manages user sign-up, login, and JWT token issuance through a Hosted UI, so your API never has to handle passwords directly.
+ **Amazon API Gateway** — Sits in front of your compute layer, validating every JWT token and enforcing a usage plan with an API key before any request reaches your code.
+ **AWS Lambda** — Processes verified requests, calls Amazon Bedrock for AI summarization, and reads/writes to DynamoDB. Runs only when invoked, and scales to zero when idle.
+ **Amazon Bedrock** — Generates the actual text summary using the Amazon Nova Lite foundation model, called via a cross-region inference profile.

You will also build the infrastructure that core pipeline was recently use for product. Terraform for repeatable infrastructure, a CodePipeline CI/CD pipeline with automated tests and a manual approval gate, CloudWatch monitoring with a real firing alarm, and a CIS-aligned security baseline.

---

#### Content

1. [Overview](1.1-Overview/)
2. [Prerequisites](2-Prerequisites/)
3. [Architecture Design](3-Architecture/)
4. [Implementation](4-Implementation/)
   1. [Backend Foundation — DynamoDB & Lambda](4.1-Backend-Foundation/)
   2. [Auth Layer — Cognito](4.2-Auth-Layer/)
   3. [API Layer — API Gateway](4.3-API-Layer/)
   4. [AI Integration — Bedrock](4.4-AI-Integration/)
   5. [Frontend Hosting — S3 & CloudFront](4.5-Frontend-Hosting/)
   6. [Weekly Report Pipeline — EventBridge](4.6-Report-Pipeline/)
   7. [CI/CD Pipeline — CodePipeline](4.7-CICD-Pipeline/)
   8. [Monitoring & Security Layer](4.8-Monitoring-Security/)
5. [Testing & Measurement](5-Testing-Measurement/)
6. [Clean Up](6.3-Cleanup/)