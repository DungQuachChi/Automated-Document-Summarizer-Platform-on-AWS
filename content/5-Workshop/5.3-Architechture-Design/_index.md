---
title : "Architecture Design"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### End-to-End Dataflow

![overview](/images/5-Workshop/5.3-Architechture-Design/project_arch.jpeg)

#### CI/CD Pipeline Path

    GitHub push (main branch)
         |
         v
    CodePipeline: Source ──── pulls repo via CodeStar connection
         |
         v
    CodeBuild: Test ──── pytest tests/
         |
         v
    CodeBuild: Security Scan ──── bandit (Python) + tfsec (Terraform)
         |
         v
    CodeBuild: Terraform Plan ──── plan output saved as pipeline artifact
         |
         v
    Manual Approval ──── human reviews plan output before proceeding
         |
         v
    CodeBuild: Terraform Apply ──── deploys the reviewed plan

#### Weekly Report Path

    EventBridge (weekly cron rule)
         |
         v
    Report Lambda
         |
         v
    S3 Reports Bucket (AES256 encryption)
         |
         | after 30 days
         v
    Glacier (lifecycle transition)
         |
         v
    Email notification

#### Services Table

| Service | Role | Why chosen |
|---|---|---|
| Cognito | User authentication, JWT issuance | Managed identity service — no custom auth code, built-in Hosted UI |
| API Gateway | Request validation, routing, rate limiting | Managed entry point — JWT validation and usage plans without custom middleware |
| Lambda | Business logic execution | Pay-per-invocation, scales to zero, no server management |
| Bedrock | AI text summarization | Managed foundation model access, no model hosting/GPU management |
| DynamoDB | Data storage | Serverless NoSQL, HTTP-based API fits Lambda's stateless invocation model, on-demand billing |
| S3 | Static frontend hosting, report storage | Durable object storage, near-zero cost for small static assets |
| CloudFront | CDN for frontend | HTTPS termination, edge caching, required for a public-facing S3 site |
| EventBridge | Scheduled trigger for weekly reports | Managed cron, no server needed to keep a schedule running |
| CloudWatch + SNS | Metrics, dashboards, alerting | Native integration with every other AWS service used here |
| CloudTrail + Config | Audit logging, compliance rules | Required for the CIS Foundations Benchmark conformance pack |
| CodePipeline / CodeBuild | CI/CD automation | Native AWS integration with CodeStar/GitHub, no separate CI service to manage |
| Terraform | Infrastructure as Code | Declarative, plan-before-apply workflow, multi-cloud portable skill |

#### Security and IAM Fundamentals

**Least privilege — the Lambda execution role.** The Lambda function's IAM role grants only the specific actions it needs:

```hcl
resource "aws_iam_role_policy" "lambda_permissions" {
  policy = jsonencode({
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["dynamodb:PutItem", "dynamodb:Query"]
        Resource = aws_dynamodb_table.summarizer.arn
      },
      {
        Effect   = "Allow"
        Action   = ["bedrock:InvokeModel"]
        Resource = "arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0"
      }
    ]
  })
}
```

No dynamodb: wildcard, no bedrock: wildcard, no access to any other table or model. If this role's credentials were ever leaked, the blast radius is limited to writing/reading one DynamoDB table and invoking one Bedrock model.

**IAM roles vs. users vs. hardcoded keys.** Lambda uses an IAM **role** — temporary credentials assumed at invocation time, never stored anywhere. IAM **users** with long-lived access keys are used only for human CLI access (personal profile). No AWS credentials are hardcoded in application code or committed to git at any point in this project.

**No public S3 buckets.** The frontend bucket is not publicly readable. CloudFront reaches it through an **Origin Access Control (OAC)**, so the only path to the bucket's contents is through CloudFront's HTTPS endpoint — direct S3 URLs are blocked.

**Cognito security trade-offs.** Advanced security features are enabled (compromised credential detection, risk-based sign-in). The Hosted UI is used instead of a custom-built login form, trading some UI customization for a maintained, security-patched auth flow.

**Secrets handling today vs. upgrade path.** Currently, the API key used by the frontend is retrieved via terraform output and placed in environment configuration manually. A future upgrade path would move this into **AWS Secrets Manager** with automatic rotation, removing the manual step entirely.

**CIS AWS Foundations Benchmark v1.4 Level 1 — current status: 60% compliant.** Two gaps are currently accepted as deliberate trade-offs for a learning-budget project, not hidden oversights:

- CodeBuild's IAM role uses AdministratorAccess rather than a scoped policy — the exact set of permissions Terraform needs varies by which resources are being changed, and scoping it precisely was deprioritized against the $50/month budget and project timeline.
- No MFA is enforced at the IAM policy level for all users — MFA is enabled on the personal admin user (see Prerequisites), but not technically enforced account-wide via a conditional policy.

Both are documented here explicitly, with a plan to close them in Section 6.2 (Simple Security Hardening).

#### Scalability

Every compute choice in this architecture already scales to zero and back up automatically: Lambda scales per-request, DynamoDB on-demand billing scales with traffic, API Gateway and CloudFront handle traffic spikes natively. None of this requires manual intervention at the traffic levels this workshop targets.

At real production scale, the following would need to change — noted here as a "not needed now, here's what changes" reference, not as work to do in this workshop:

- **Provisioned concurrency** for Lambda, to eliminate cold-start latency under sustained high traffic
- **API Gateway response caching**, explicitly skipped here for cost (see Section 6.1) — would reduce redundant Bedrock calls for repeated inputs at higher traffic volumes
- **Multi-region deployment**, for latency and disaster recovery — this workshop runs single-region (ap-southeast-1) with Bedrock calls routed cross-region to us-east-1 only for model availability, not for redundancy