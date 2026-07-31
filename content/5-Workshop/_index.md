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
4. [Backend & AI](5.4-Backend&AI/)
   1. [DynamoDB & Lambda](5.4.1-DynamoDB&Lambda/)
   2. [AI Integration](5.4.2-AI_Integration/)
5. [AccessLayer](5.5-Access Layer/)
   1. [API Layer — API Gateway](5.5.1-APIGateway/)
   2. [S3&CloudFront](5.5.2-S3&CloudFront/)
6. [Automation&Operations](5.6-Automation&Operations/)
   1. [CICD&Automation](5.6.1-CICD&Automation/)
   2. [Monitoring&Security](5.6.2-Monitoring&Security/)
7. [Testing](5.7-Testing/)
8. [Clean Up](5.8-Cleanup/)