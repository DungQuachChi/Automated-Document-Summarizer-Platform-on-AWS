---
title: "Workshop"
date: 2026-07-14
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Project: Infrastructure as Code for a Secure Serverless AI Workload on AWS

This workshop is a DevOps and Cloud Engineering project that uses AI document summarizer as the vehicle to teach secure serverless architecture, Infrastructure as Code, CI/CD automation, and observability on AWS.

In this workshop build as a serverless document summarization platform. The platform accepts long-form text documents through a secured REST API and returns AI-generated summaries powered by Amazon Bedrock.

There are four main building blocks that operate collectively throughout the entire request pipeline: Amazon Cognito, Amazon API Gateway, AWS Lambda, and Amazon Bedrock. Each block performs its own responsibility within the request flow, from authentication to inference.

+ **Amazon Cognito**: Takes care of the user’s sign-up, log-in, and generation of the JWT token via Hosted UI, thus not involving your API in working with any passwords.
+ **Amazon API Gateway**: Comes in front of your compute block and validates each JWT token, as well as implements the usage plan via an API key.
+ **AWS Lambda**: Executes the validated requests, invokes Amazon Bedrock to get the AI-based summary, reads/writes into DynamoDB. Invoked on-demand and scaled down to zero capacity in between.
+ **Amazon Bedrock**: Provides the text summary based on the Amazon Nova Lite foundation model, invoked via the cross-region inference profile.

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