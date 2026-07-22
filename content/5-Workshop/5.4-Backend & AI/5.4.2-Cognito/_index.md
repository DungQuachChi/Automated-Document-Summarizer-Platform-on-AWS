---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.4.2. "
---

#### Goal

Request access to Amazon Nova Lite in Bedrock, understand how the Lambda (Section 5.4.1) calls it across regions, and handle the account's on-demand quota running out.

#### Terraform Wiring: IAM and Model ID

- IAM policy scopes bedrock:InvokeModel to exactly two resources: the foundation model ARN and the cross-region inference profile ARN — not a wildcard.
- BEDROCK_MODEL_ID = "us.amazon.nova-lite-v1:0" is a Terraform-set Lambda environment variable, not hardcoded in Python.

#### Cross-Region Requirement

ap-southeast-1 is not in the AP inference pool for Nova Lite. The Lambda's Bedrock client is hardcoded to call us-east-1 instead:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

The Lambda itself keeps running in ap-southeast-1; only the Bedrock API call crosses regions.

#### The Quota Reality

New AWS accounts are given with an on-demand quota of **zero requests/second** for a given Bedrock model, even after model access is accepted a genuine account-level restriction, not a bug in this project. Every **/summarize** call failed with ThrottlingException until an AWS Support case  increased it.

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

Transient throttling retries with backoff; a daily-quota error fails fast instead, since retrying it just burns the Lambda's 30-second timeout for no benefit.

The deployed Lambda has **no mock path** — it always calls real Bedrock, and returns a genuine 429 to real users if quota is exhausted:

```json
{"message": "Summarization limit reached for today. Please try again after midnight UTC."}
```

Check CloudWatch → Metrics → Custom/Bedrock → BedrockErrors, dimension ErrorType = DailyQuotaExceeded, to confirm the Lambda classified the error correctly.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException calling Bedrock | Model access not granted, or IAM ARN doesn't match the model/region | Grant access in **Bedrock → Model access**; confirm IAM ARNs match BEDROCK_MODEL_ID |
| ValidationException: model identifier is invalid | Wrong model ID format | Confirm BEDROCK_MODEL_ID is exactly us.amazon.nova-lite-v1:0 |
| 429 on every request, even right after requesting access | On-demand quota defaults to 0 on some new accounts | File an AWS Support case under Service Quotas |

---

### Section Summary

Congratulations on completing the backend and AI layer. In this section, you built a DynamoDB table with a GSI for date-range queries, a least-privilege Lambda execution role, and wired the Lambda to call Amazon Bedrock across regions for text summarization. It worked because every IAM permission granted matched exactly one code path — no wildcard actions, no wildcard resources beyond what cross-region inference required — and because quota exhaustion was treated as an expected condition, a fast-failing 429, rather than an unhandled failure. This demonstrates least-privilege IAM and graceful degradation: a backend that fails predictably and cheaply when a managed AI service's quota runs out, instead of retrying blindly or crashing silently.