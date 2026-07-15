---
title : "Introduction"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Serverless Architecture

+ **Serverless architecture** runs code only when a request arrives, then shuts down immediately after. There are no servers to patch, scale, or keep running 24/7.
+ This workshop uses four core AWS services to build a serverless request pipeline: **Cognito** authenticates the user, **API Gateway** validates the request, **Lambda** runs the business logic, and **Bedrock** generates the AI summary.

#### Workshop overview

In this workshop, you will build a document summarization platform end to end.

+ **The feature** — client submits long-form text (lecture notes, readings, articles) through a secured API and receives an AI-generated summary back, powered by Amazon Bedrock.
+ **The actual subject** — the DevOps practices required to build, secure, and operate that feature: Infrastructure as Code with Terraform, a CI/CD pipeline with a manual approval gate, CloudWatch monitoring with a real firing alarm, and a CIS-aligned security baseline.

Partway through building this system, the AWS account in use hit a **zero on-demand quota** for Bedrock models — a limit AWS applies to new accounts. A feature flag, MOCK_SUMMARIZE, the rest of the pipeline continues to function correctly while the quota issue is resolved. 

![overview](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)

#### Goals / Desired Outputs

By the end of this workshop, the following will be deployed and verified:

+ A working, authenticated REST API (Cognito-protected /summarize and /history endpoints)
+ A static frontend deployed via S3 + CloudFront
+ A live CloudWatch dashboard showing real traffic
+ At least one SNS alert triggered and received by email
+ A fully automated CI/CD pipeline with a manual approval gate before any infrastructure change is applied
+ A clean, verified teardown with zero ongoing cost

#### Success Criteria

+ [ ] Auth flow functional end-to-end (Cognito Hosted UI → JWT → authorized API call)
+ [ ] CloudWatch dashboard displays live request traffic
+ [ ] At least one CloudWatch alarm fires and delivers an SNS email
+ [ ] Monthly cost stays under $50 across all services
+ [ ] All resources destroyed with zero orphaned billing (verified in Cost Explorer)