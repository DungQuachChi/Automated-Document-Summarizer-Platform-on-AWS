---
title : "Operations — Monitoring & Security"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.6.2. "
---

#### Goal

Visibility into the running system (dashboard, alarms, custom metrics) and a security/compliance baseline.

#### Custom Metrics

src/lambda_fn/lambda_function.py publishes to Custom/Bedrock around every model call: Latency, SuccessCount, and on failure BedrockErrors dimensioned by ErrorType.

#### Alarms (modules/monitoring)

One SNS topic + email subscription, two alarms:
- API Gateway 5xx **rate** (metric-math errors/requests*100) > 5% over 5 minutes — rate, not raw count.
- Any Custom/Bedrock/BedrockErrors data point at all (threshold = 0).

```bash
terraform apply   # confirm the SNS email subscription afterward, or alarms won't deliver
aws cloudwatch put-metric-data --namespace Custom/Bedrock --metric-name BedrockErrors --value 1
```

#### Dashboard

One aws_cloudwatch_dashboard resource, six widgets — Lambda invocations/errors/duration percentiles, Bedrock latency/success/error, API Gateway traffic/4xx/5xx, DynamoDB capacity/throttles. Dimensions passed in as module variables from root main.tf.

#### CloudTrail & AWS Config (modules/security)

- Multi-region CloudTrail, log file validation, dedicated encrypted/private S3 bucket, also streamed to CloudWatch Logs.
- AWS Config recorder tracks all supported resource types including global (IAM), with the CIS v1.4 Level 1 conformance pack.

**Current compliance: 60%**, with two accepted, documented gaps rather than hidden ones:
- CodeBuild's IAM role uses AdministratorAccess.
- MFA isn't enforced account-wide via IAM policy.

Both are revisited in Section 6.2 (Simple Security Hardening) — the point of running Config here is an accurate picture, not a clean scorecard.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Alarm stuck at **Insufficient Data** | No data points published yet | Generate real traffic, or manually publish a test metric |
| SNS email never arrives | Subscription never confirmed | Check spam folder; re-subscribe if expired |
| Conformance pack stuck at **Evaluating** | Normal on first run | Wait 15–30 minutes rather than re-deploying |

---

### Section Summary

Congratulations on completing the automation and operations layer. In this section, you set up a scheduled Lambda that generates weekly usage reports without any manual trigger, a CI/CD pipeline that tests, scans, and requires human approval before any infrastructure change is applied, and a monitoring and security baseline — dashboards, alarms, CloudTrail, and AWS Config — that watches the system once it's running. This worked because every stage that could modify real infrastructure sits behind an automated check or a manual approval gate, and every alert path was tested end-to-end rather than assumed to work. This demonstrates the core DevOps principle behind this whole project: infrastructure changes are reviewed before they happen, and the system's health stays observable after they do, instead of being discovered only when a user reports a problem.