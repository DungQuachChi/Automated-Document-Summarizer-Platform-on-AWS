---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.4.2. "
---

#### Goal

Request access to Amazon Nova Lite in Bedrock, understand how the Lambda calls it across regions, and handle the account's on-demand quota running out.

#### Terraform Wiring: IAM and Model ID

- IAM policy scopes bedrock:InvokeModel to exactly two resources: the foundation model ARN and the cross-region inference profile ARN, not a wildcard.
- BEDROCK_MODEL_ID = "us.amazon.nova-lite-v1:0" is a Terraform-set Lambda environment variable, not hardcoded in Python.

#### Cross-Region Requirement

ap-southeast-1 is not in the AP inference pool for Nova Lite. The Lambda's Bedrock client is hardcoded to call us-east-1 instead:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

The Lambda itself keeps running in ap-southeast-1; only the Bedrock API call crosses regions.

#### The Quota Reality

New AWS accounts start with an on-demand quota of **zero requests/second** for a given Bedrock model, even after model access is granted. This is a genuine account-level restriction, not a bug in this project. Every **/summarize** call failed with ThrottlingException.

We filed an AWS Support case to raise the quota. AWS declined for now, explaining that model eligibility depends on account age, payment history, and usage, that it is reassessed automatically over time, and that it is not a permanent restriction. In other words, the blocker is billing history, not configuration.

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

Transient throttling retries with backoff; a daily-quota error fails fast instead, since retrying it just burns the Lambda's 30-second timeout for no benefit.

The deployed Lambda has **no mock path** of its own. It always calls real Bedrock and returns a genuine 429 to real users when quota is exhausted:

```json
{"message": "Summarization limit reached for today. Please try again after midnight UTC."}
```

To keep the frontend demo usable while access is gated, the static site has a `MOCK_SUMMARIZE` flag that serves a placeholder summary. Turning it off is a single flag with no code change, so real inference goes live the moment the account becomes eligible. The integration itself is verified by the pytest suite, which mocks Bedrock and asserts that the request payload, the retry path, and the 429 quota classification are all correct.

Check CloudWatch → Metrics → Custom/Bedrock → BedrockErrors, dimension ErrorType = DailyQuotaExceeded, to confirm the Lambda classified the error correctly.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException calling Bedrock | Model access not granted, or IAM ARN doesn't match the model/region | Grant access in **Bedrock → Model access**; confirm IAM ARNs match BEDROCK_MODEL_ID |
| ValidationException: model identifier is invalid | Wrong model ID format | Confirm BEDROCK_MODEL_ID is exactly us.amazon.nova-lite-v1:0 |
| 429 on every request, even right after requesting access | On-demand quota defaults to 0 on some new accounts | File an AWS Support case under Service Quotas |

---

### Section Summary

Well done on finishing the backend and the AI layer! Here, you designed a DynamoDB table with a GSI to run date-based queries, a Lambda execution role with the least amount of permissions, and connected the Lambda service to call Amazon Bedrock for text summarization across different regions. The success of this process can be attributed to every IAM permission granted matching exactly one code path; there was no use of wildcard actions, no use of wildcard resources beyond what was needed for cross-region inference. Moreover, quota exhaustion was treated as usual; thus, we experienced an expected 429 error rapidly instead of an unexpected one.