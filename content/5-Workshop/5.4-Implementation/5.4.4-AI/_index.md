---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

#### Goal

Request access to the Amazon Nova Lite model in Bedrock, understand how the deployed Lambda calls it, and cover what happens when the account's on-demand quota runs out.

#### Terraform Wiring: IAM and Model ID

Model access approval happens in the console (it's an account-level grant, not a Terraform resource), but everything downstream of it is code:

- `modules/compute`'s IAM policy scopes `bedrock:InvokeModel` to exactly two resources: the foundation model ARN (`arn:aws:bedrock:*::foundation-model/amazon.nova-lite-v1:0`) and the cross-region inference profile ARN (`arn:aws:bedrock:us-east-1:*:inference-profile/us.amazon.nova-lite-v1:0`) — not a wildcard across all Bedrock models.
- `BEDROCK_MODEL_ID = "us.amazon.nova-lite-v1:0"` is set as a Lambda environment variable in Terraform, not hardcoded in the Python file — the `us.` prefix identifies this as a cross-region inference profile rather than a single-region model ID, which matters for the next point.

#### Cross-Region Requirement

ap-southeast-1 is not in the AP inference pool for Nova Lite. The Lambda's Bedrock client is hardcoded to call `us-east-1` instead:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

Lambda itself continues to run in ap-southeast-1 (Section 4.1); only the Bedrock API call crosses regions.

#### Test a Real Bedrock Call

1. Open the **Lambda** console → `doc-summarizer-fn` → **Test** tab.
2. Re-run the `TestSummarize` event created in Section 4.1.
3. If model access was granted in Step 1, this now returns a real AI-generated summary instead of failing.

**How to verify:** The response body contains a `summary` field with coherent, on-topic text — not a fixed placeholder string.

#### The Quota Story: When On-Demand Quota Hits Zero

New AWS accounts are sometimes provisioned with an on-demand quota of **zero requests per second** for a given Bedrock model, even after model access itself is granted. This is a separate limit from model access, and it's a genuine account-level restriction, not a bug anywhere in this project's code.

This is exactly what happened on this project's AWS account. Every call to `/summarize` failed with a `ThrottlingException`, and the Lambda's retry logic correctly detected the daily-quota pattern in the error message and fast-failed rather than burning the full 30-second timeout on pointless retries:

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

For anything else classified as retriable (`ThrottlingException` without the daily-quota pattern, or `ModelTimeoutException`), the Lambda retries with backoff, up to `max_retries` attempts.

**Resolution path:** an AWS Support case was filed under **Service Quotas** requesting an increase for the Nova Lite on-demand quota in us-east-1. Quota increase requests are not instant — they can take from a few hours to a few days depending on account history and usage justification.

#### The Deployed Lambda Has No Mock Path

`doc-summarizer-fn` always calls the real Bedrock endpoint — there's no environment variable or code branch that substitutes a canned response. If the account has zero quota, real requests to the deployed API genuinely fail with a 429, exactly as shown to an end user. Nothing about the deployed system's behavior is faked to work around the quota gap.

**Correction needed on the local dev side:** an earlier version of this section described a `MOCK_SUMMARIZE` flag toggling a local FastAPI app between mock and real Bedrock calls. That flag doesn't exist in the current code. `app/bedrock_client.py`'s `get_summary()` is hardcoded to always return a mock string — there's no env var and no real-call branch — and that FastAPI app (SQLAlchemy, a `Document` model, no Cognito/JWT, no DynamoDB) looks like a separate early prototype rather than a mock layer sitting in front of the same system described elsewhere in this workshop. Before publishing this section, decide which of these is true and write to that:
- If the FastAPI app is meant to be a genuine local mock-toggle for the deployed system, it needs an actual `MOCK_SUMMARIZE` env var and a real-Bedrock code path added.
- If it's an earlier prototype that's no longer part of the current architecture, this section should say so plainly instead of describing a feature that isn't there.

#### How to Verify the 429 Path

1. If your account currently has zero on-demand quota, call `/summarize` on the deployed API (Section 5.5 has exact curl commands).
2. Confirm the response is:
   ```json
   {"message": "Summarization limit reached for today. Please try again after midnight UTC."}
   ```
   with HTTP status 429.
3. Check the **CloudWatch** console → **Metrics** → `Custom/Bedrock` namespace → `BedrockErrors` metric, dimension `ErrorType = DailyQuotaExceeded`. A data point here confirms the Lambda correctly classified and reported the quota error rather than treating it as a generic failure.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `AccessDeniedException` calling Bedrock | Model access not yet granted, or the IAM policy's resource ARNs don't match the model/profile actually being called | Complete Step 1; confirm `BEDROCK_MODEL_ID` matches one of the two ARNs scoped in the compute module's IAM policy |
| `ValidationException: model identifier is invalid` | Wrong model ID format used | Confirm `BEDROCK_MODEL_ID` is exactly `us.amazon.nova-lite-v1:0`, including the `us.` prefix |
| 429 on every request, even right after requesting access | On-demand quota is separate from model access and defaults to 0 on some new accounts | File an AWS Support case under Service Quotas |
| Local FastAPI app always returns a mock summary, even after setting an env var | No such env var exists in the current `app/bedrock_client.py` — the mock return is unconditional | Either add the toggle, or stop describing this app as configurable until it is |