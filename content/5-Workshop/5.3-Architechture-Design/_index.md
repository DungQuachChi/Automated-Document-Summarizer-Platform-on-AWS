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

![overview](/images/5-Workshop/5.3-Architechture-Design/CI_CD_Pipeline.png)

#### Weekly Report Path

![overview](/images/5-Workshop/5.3-Architechture-Design/Weekly_Report_Path.png)

#### Services Table

| Service | Role | Why chosen |
|---|---|---|
| Cognito | User authentication, JWT issuance | Managed identity service (no custom auth code, built-in Hosted UI) |
| API Gateway | Request validation, routing, rate limiting | Managed entry point (JWT validation and usage plans without custom middleware) |
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

**IAM roles vs. users vs. hardcoded keys.** Lambda uses an IAM **role** — is being utilized by the Lambda function as it is being executed by temporary credentials granted at the time the function is invoked; there is no IAM users with long-lived access keys are used only for human CLI access (personal profile). Throughout the project, no values of AWS credentials were hard-coded into application code or committed to git.

**S3 buckets should not be accessible to the public.** The frontend bucket must not be publicly accessible. CloudFront accesses the bucket via the **Origin Access Control (OAC)** mechanism. Thus, the only way to access its contents is secure HTTPS protocol through CloudFront — direct URL access to S3 is disabled.

**Cognito security trade-offs.** Advanced security features are enabled (compromised credential detection, risk-based sign-in). The Hosted UI is used instead of a custom-built login form, trading some UI customization for a maintained, security-patched auth flow.

**Secrets handling today vs. upgrade path.** Currently, the API key used by the frontend is retrieved via terraform output and placed in environment configuration manually. A future upgrade path would move this into **AWS Secrets Manager** with automatic rotation, removing the manual step entirely.

**CIS AWS Foundations Benchmark v1.4 Level 1 - current status: 60% compliance.** There are two known gaps accepted as conscientious compromises in the context of a learning budget project:

- Given the fact that the IAM role of CodeBuild is providing AdministratorAccess instead of following the scoped policy, the permissions set required by Terraform depends on the resources being modified, while having perfectly scoped permissions was of a lesser priority in view of the project budget (50 per month) and timeline.
- Multifactor authentication is in place for an individual admin user (please refer to Prerequisites), but it is not implemented at the level of the IAM policy for all accounts.
Both are documented here explicitly, with a plan to close them in Section 6.2 (Simple Security Hardening).

#### Scalability

Every compute choice in this architecture already scales to zero and back up automatically: Lambda scales per-request, DynamoDB on-demand billing scales with traffic, API Gateway and CloudFront handle traffic spikes natively. None of this requires manual intervention at the traffic levels this workshop targets.
For real-scale production, the changes will have to be introduced — referred in the text as "unneeded now, here is what changes" and not as an action item for this workshop:
- **Provisioned concurrency** for the Lambda to remove cold-start latency during sustained high-load activities.
- **API Gateway response caching** which has been skipped here for financial reasons (explained in Section 6.1) means reducing Bedrock calls for inputs repeated under higher load of traffic.
- **Multi-region deployment** to help with the issues of latency and disaster recovery — this workshop does not require this due to operation in a single region (ap-southeast-1) with Bedrock calls redirected cross-region only for the sake of model availability.