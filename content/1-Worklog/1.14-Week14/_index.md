---
title: "Week 14 Worklog"
date: 2026-06-29
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Week 14 Objectives:
* Codify the tested cloud architecture into **Terraform (Infrastructure as Code)**.
* Build an automated deployment **CI/CD Pipeline** using GitHub Actions.
* Conduct load testing, perform a comprehensive security audit, and finalize all Capstone project documentation.

### Daily Tasks:
| Day | Task | Start Date | Completion Date | References |
| --- | --- | --- | --- | --- |
| Mon | Write Terraform modules for auth (Cognito), data (S3, DynamoDB), and compute (Lambda Functions, Bedrock IAM Roles) <br> Implement remote state management using S3 backend and DynamoDB state locking | 2026-06-29 | 2026-06-29 | Terraform AWS Provider Docs |
| Tue | Finalize Terraform module for api (API Gateway, Routes, Usage Plans, CloudFront OAC) <br> Execute terraform apply to provision the entire cloud infrastructure from scratch | 2026-06-30 | 2026-06-30 | Terraform Best Practices |
| Wed | Build a **GitHub Actions Workflow** to automate linting, execute Lambda Unit Tests, and trigger auto-deployment on pushes to main | 2026-07-01 | 2026-07-01 | GitHub Actions Docs |
| Thu | Perform Load Testing using **k6** <br> Validate Rate Limiting policies (100 req/min) and verify API Gateway Cache performance | 2026-07-02 | 2026-07-02 | k6 Load Testing Guide |
| Fri | Audit IAM Roles following the Principle of Least Privilege <br> Finalize architecture diagrams, API specs, cost breakdown reports, and Capstone defense slides | 2026-07-03 | 2026-07-03 | Project Capstone Docs |

### Key Achievements:
* **Infrastructure as Code (IaC) with Terraform:** Successfully codified the entire cloud architecture (Cognito, DynamoDB, S3, Lambda, API Gateway, CloudFront) into structured Terraform modules, enabling single-command provisioning.
* **Automated CI/CD Pipeline Integration:** Configured a GitHub Actions Pipeline that performs code quality checks (linting), executes unit tests for text processing functions, and automates terraform plan/apply upon merging code into main.
* **Load Testing & System Performance Evaluation:** Simulated concurrent user workloads using k6, confirming that API Rate Limiting effectively blocks abusive traffic and that 1-hour caching reduces Bedrock model invocation costs by over 40%.
* **Security Hardening & Infrastructure Audit:** Restricted public access to the S3 hosting bucket by implementing CloudFront Origin Access Control (OAC). Refined Lambda IAM Policies to strictly follow the Principle of Least Privilege.
* **Capstone Documentation & Defense Readiness:** Completed all technical deliverables, including overall Architecture Diagrams, OpenAPI/Swagger specifications, an operating cost report (< $5/month under standard usage), and presentation slides for the Capstone review board.