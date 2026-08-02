---
title: "Proposal: Automated Serverless Document Summarizer Platform on AWS"
date: 2026-08-02
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Proposal: Automated Serverless Document Summarizer Platform on AWS

## 1. Project Overview

This project builds a production-grade, fully serverless document summarization platform on AWS, developed by a two-person team. Authenticated users submit a block of text through a REST API (or a simple web page) and receive an AI-generated summary powered by Amazon Bedrock. Every summarization is stored per user, browsable through a history endpoint, and aggregated into a weekly CSV usage report generated automatically on a schedule.

Beyond the application itself, the project's core purpose is to practice modern cloud engineering end-to-end: the entire infrastructure is defined as code with Terraform, deployed through an automated CI/CD pipeline with security scanning, monitored with dashboards and alarms, and hardened against the CIS AWS Foundations Benchmark — all within a strict student budget.

**Team split:** one member owns the backend, Bedrock integration, and all infrastructure/CI-CD; the other owns the frontend styling, the weekly report business logic, and load testing.

## 2. Objectives

- **Serverless compute:** AWS Lambda (Python) invoking Amazon Bedrock (Amazon Nova Lite) for text summarization. Synchronous calls, input up to 5,000 characters.
- **Authentication:** Amazon Cognito with OAuth 2.0 (Hosted UI login page, JWT tokens) protecting every API route.
- **API management:** Amazon API Gateway (REST) with a Cognito authorizer, an API key, and a usage plan enforcing rate and monthly-quota limits.
- **Data persistence:** Amazon DynamoDB single-table design (user_id partition key, timestamp sort key) with a GSI for date-based report queries.
- **Scheduled reporting:** Amazon EventBridge cron triggering a report Lambda weekly, writing per-user CSV summaries to S3 with a Glacier lifecycle policy.
- **CI/CD:** AWS CodePipeline + CodeBuild: pytest unit tests, bandit and tfsec security scans, terraform plan, manual approval, then terraform apply.
- **Infrastructure as Code:** modular Terraform (auth, api, compute, data, scheduling, frontend, monitoring, pipeline modules) with remote state on S3 and DynamoDB state locking.
- **Observability:** CloudWatch dashboard (Lambda duration and Bedrock latency at p50/p95/p99, API errors, DynamoDB usage) with SNS email alarms.
- **Security & compliance:** least-privilege IAM, multi-region CloudTrail, AWS Config with the CIS AWS Foundations Benchmark conformance pack.
- **Demo frontend:** static HTML/JS site on S3 behind CloudFront, logging in via the Cognito Hosted UI and calling the API with the JWT.

## 3. Problem Statement

Deploying AI-based applications by hand is fragile: manual console configuration causes environment drift, undocumented dependencies, and setups that cannot be reproduced or reviewed. Calling generative-AI APIs in production raises further problems that a simple script ignores — authentication and per-user identity, abuse prevention and rate limiting, transient model errors and quota limits, cost visibility, and auditability.

This project addresses those problems with a fully serverless, reproducible architecture: every resource is version-controlled Terraform, every deployment passes automated tests and security scans, every request is authenticated and rate-limited, and the AI call path is defensive (retries with exponential backoff, fast failure on quota exhaustion). For a student team, it also answers a practical question: can a production-grade GenAI backend be run on a budget of tens of dollars per month?

## 4. Solution Architecture

**Request flow:** the user signs in through the Cognito Hosted UI and is redirected back with a JWT. The frontend calls POST /summarize or GET /history on API Gateway with the token; the Cognito authorizer validates it and the usage plan applies rate limiting before the Lambda runs. The summarizer Lambda invokes Amazon Nova Lite on Bedrock, stores the input text and summary in DynamoDB keyed by the user's identity, and returns the summary. Weekly, EventBridge triggers a second Lambda that aggregates the past week's activity from DynamoDB into a CSV report in S3.

**Architectural layers:**

| Layer | Services |
|---|---|
| User / Edge | Static HTML/JS on S3 + CloudFront (HTTPS) |
| API | API Gateway REST API, Cognito authorizer, usage plan + API key, CORS |
| Compute | Two Lambda functions (Python): synchronous summarizer, scheduled weekly reporter |
| AI | Amazon Bedrock — Amazon Nova Lite |
| Data | DynamoDB (single table + GSI), S3 reports bucket (AES-256, Glacier lifecycle) |
| Scheduling | EventBridge weekly cron rule |
| DevOps | CodePipeline + CodeBuild (pytest, bandit, tfsec, plan/approve/apply), Terraform remote state (S3 + DynamoDB lock) |
| Security & Observability | CloudTrail (multi-region), AWS Config + CIS conformance pack, CloudWatch dashboard + alarms, SNS |

**Primary region:** ap-southeast-1 (Singapore).

## 5. Timeline

12-week internship period, April – July 2026:

| Weeks | Phase |
|---|---|
| 1–5 | AWS foundations study: networking, IAM, compute, storage, DNS, containers |
| 6 | Project kickoff: proposal, architecture design, model selection, work split |
| 7 | Environment setup: AWS CLI, Terraform, shared repository, service research |
| 8 | Core backend: Bedrock integration, response parsing, DynamoDB persistence, retries, unit tests |
| 9 | Auth & API layer: Cognito User Pool + Hosted UI, API Gateway routes, authorizer, usage plan, CORS |
| 10 | Infrastructure as Code: modular Terraform, remote state with locking |
| 11 | CI/CD pipeline + observability: CodePipeline, buildspec automation, custom metrics, dashboard, alarms |
| 12 | Hardening & delivery: CIS benchmark, load testing, frontend deployment on CloudFront, cleanup, documentation |

## 6. Budget

Hard ceiling: $50/month across all AWS services. Design decisions follow from it: on-demand DynamoDB instead of provisioned capacity, no NAT gateways or VPC endpoints, smallest viable Lambda memory, and API Gateway response caching explicitly skipped (the smallest cache cluster alone costs ~$14–19/month).

Rough monthly estimate at expected traffic (low thousands of requests):

| Item | Est. cost/month |
|---|---|
| Lambda (both functions) | ~$0 (free tier) |
| DynamoDB on-demand + PITR | < $1 |
| API Gateway REST | < $1 |
| Bedrock — Nova Lite tokens | ~$1 |
| S3 + CloudFront (static site + reports) | < $1 |
| CodePipeline + CodeBuild minutes | ~$2 |
| CloudTrail (first trail) + AWS Config recorder & rules | ~$3–5 |
| CloudWatch alarms/metrics, SNS | ~$1 |
| **Total** | **≈ $8–12 / month — well under the $50 ceiling** |

## 7. Risks

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Bedrock model access / on-demand quota limits on a new AWS account | High | Medium | Request quota early via AWS Support; keep a mock summarize path so development, testing, and the demo are never blocked by the model quota |
| Cost overrun on a student budget | Medium | Medium | Free-tier-first defaults, on-demand billing, budget alerts, avoid always-on resources; check every change against the $50 ceiling |
| Two people breaking shared infrastructure (state conflicts, Cognito/CORS config) | Medium | Medium | Terraform remote state with DynamoDB locking, branch protection with PR review, coordinate before changing shared resources |
| Security misconfiguration (over-broad IAM, public buckets) | High | Low | tfsec + bandit gates in the pipeline, least-privilege IAM, CIS conformance pack, CloudTrail audit |
| Model/region availability (chosen model not served in the home region) | Medium | Low | Use cross-region inference profiles; keep the model ID configurable via environment variable |