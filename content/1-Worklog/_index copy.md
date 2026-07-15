---
title: "Week 12 Worklog"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

* Implement the core serverless compute tier and integrate Amazon Bedrock AI text processing.
* Configure identity pools for user authentication and build secure API routing networks.
* Enforce request rate limiting, token validation layers, and cost-optimization caching mechanisms.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Request model access for Claude 3 Haiku and Titan Text inside `us-east-1` <br> - Set up a clean local Python 3.12 environment and write local mock scripts | 2026-06-15 | 2026-06-15 | Amazon Bedrock Console |
| 3 | - Code the Lambda function handler logic to invoke the Amazon Bedrock API <br> - Write payload parsing handlers for documents up to 10,000 characters | 2026-06-16 | 2026-06-16 | AWS Lambda / Bedrock Integration |
| 4 | - Troubleshoot local Boto3 credential configurations <br> - Deploy the single-table DynamoDB instance and integrate Lambda writing hooks | 2026-06-17 | 2026-06-17 | Amazon DynamoDB Console |
| 5 | - Provision an **Amazon Cognito User Pool** with mandatory email verification <br> - Create an App Client and test the user registration flows via the Hosted UI | 2026-06-18 | 2026-06-18 | Amazon Cognito Console |
| 6 | - Build the API Gateway REST architecture with `/summarize` and `/history` routes <br> - Attach a Cognito Authorizer and debug JWT payload validation mismatches <br> - Connect Lambda proxy hooks, deploy usage plans (100 req/min), and enable a 1-hour cache | 2026-06-19 | 2026-06-20 | Amazon API Gateway Developer Guide |

### Week 11 Achievements:

* AI Model Pipeline Isolation: Navigated Amazon Bedrock model processing requests for Claude 3 Haiku and Titan Text. Built robust fallback mock Python structures locally inside an isolated Python 3.12 environment using Boto3 to protect development timelines while model approval was pending.

* Payload Engineering & Compute Logic: Developed an AWS Lambda core processing function capable of safely handling large text inputs up to 10,000 characters. Built flexible input transformers to properly map the differing request schemas required by both Anthropic and Amazon LLM runtimes.

* Single-Table Database Provisioning: Created the project's single-table DynamoDB structure utilizing `user_id` as the Partition Key (PK) and `timestamp` as the Sort Key (SK). Overcame local terminal credential conflicts and implemented automated write hooks within the Lambda execution flow to log transactional metrics seamlessly.

* OAuth 2.0 Identity Management: Configured secure user onboarding paths by launching an Amazon Cognito User Pool with mandatory email verification. Deployed a specialized Cognito App Client supporting code grant flows and validated user login workflows using the Cognito Hosted UI.

* Secure API Routing Network Infrastructure: Built a production-grade Amazon API Gateway REST instance featuring `/summarize` and `/history` resource routes. Secured the endpoints with a Cognito Authorizer and successfully resolved token validation rejections by tracing the token header payload structure (correcting the request to pass the Cognito ID Token instead of the Access Token).

* Traffic Control & Cost Optimization Optimization: Configured API Gateway usage plans to enforce a strict rate limit of 100 requests per minute per user, protecting backend functions from surge abuse. Successfully lowered Amazon Bedrock model invocation costs by activating a 1-hour API Gateway caching layer to intercept identical, repeated text requests.