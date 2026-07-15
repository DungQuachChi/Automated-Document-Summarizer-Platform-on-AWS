---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

#### Goal

Request access to the Amazon Nova Lite model in Bedrock, understand how the Lambda function (built in Section 4.1) calls it, and cover what happens when the account's on-demand quota runs out.

#### Step 1 — Request Model Access

Bedrock requires explicit access approval per model before any Lambda can call it — this is separate from IAM permissions.

1. Open the AWS Console and search for **Amazon Bedrock**.
2. In the left menu, click **Model access**.
3. Click **Modify model access**.
4. Find **Amazon Nova Lite** and check the box next to it.
5. Click **Save changes**.
6. Wait 5–10 minutes. Status changes from **Available** to **Access granted**.

**How to verify:** The **Model access** page shows **Access granted** next to Amazon Nova Lite.

#### Step 2 — Understand the Cross-Region Requirement

ap-southeast-1 is not in the AP inference pool for Nova Lite. The Lambda function's Bedrock client is hardcoded to call us-east-1 instead, using the model ID us.amazon.nova-lite-v1:0 — the us. prefix identifies this as a cross-region inference profile rather than a single-region model ID.

Lambda itself continues to run in ap-southeast-1; only the Bedrock API call crosses regions. This is already wired into the Lambda code from Section 4.1:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

#### Step 3 — Test a Real Bedrock Call

1. Open the **Lambda** console → doc-summarizer-fn → **Test** tab.
2. Re-run the TestSummarize event created in Section 4.1, Step 8.
3. If model access was granted in Step 1 and the IAM policy from Section 4.1 includes bedrock:InvokeModel, this now returns a real AI-generated summary instead of failing.

**How to verify:** The response body contains a summary field with coherent, on-topic text — not a fixed placeholder string.

#### The Quota Story: When On-Demand Quota Hits Zero

New AWS accounts are sometimes provisioned with an on-demand quota of **zero requests per second** for a given Bedrock model, even after model access itself is granted. This is a separate limit from model access, and it's a genuine account-level restriction, not a bug anywhere in this project's code.

This is exactly what happened on this project's AWS account. Every call to /summarize failed with a ThrottlingException, and the Lambda's retry logic (Section 4.1) correctly detected the daily-quota pattern in the error message and fast-failed rather than burning the full 30-second timeout on pointless retries:

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

**Resolution path:** an AWS Support case was filed under **Service Quotas** requesting an increase for the Nova Lite on-demand quota in us-east-1. Quota increase requests are not instant — they can take from a few hours to a few days depending on account history and usage justification.

#### Working Around the Quota Gap: MOCK_SUMMARIZE

While waiting for the quota increase, development and testing needed to continue. The fix was a feature flag, MOCK_SUMMARIZE, on the **local FastAPI development app** — not on the deployed Lambda function. This is an important distinction:

- **Deployed Lambda (doc-summarizer-fn)** — always calls the real Bedrock endpoint. There is no mock path inside the Lambda itself. If the account has zero quota, real requests to the deployed API will genuinely fail with a 429, as shown to the end user.
- **Local FastAPI app** — used for local development and frontend integration testing without touching AWS at all. When MOCK_SUMMARIZE=true, it returns a canned summary instead of calling Bedrock, so a developer can build and test the rest of the flow (request validation, response shape, frontend rendering) independent of AWS quota, cost, or network access.

This split matters for the workshop's narrative: the deployed system is honest about the quota limitation — it returns a real 429 error to real users, rather than silently mocking data in production. The mock path only exists in the local dev loop, where hiding the AI call is a legitimate and common practice for fast iteration.

#### How to Verify the 429 Path

1. If your account currently has zero on-demand quota, call /summarize on the deployed API (see Section 5 for exact curl commands).
2. Confirm the response is:
   ```json
   {"message": "Summarization limit reached for today. Please try again after midnight UTC."}
   ```
   with HTTP status 429.
3. Check the **CloudWatch** console → **Metrics** → **Custom/Bedrock** namespace → BedrockErrors metric, dimension ErrorType = DailyQuotaExceeded. A data point here confirms the Lambda correctly classified and reported the quota error rather than treating it as a generic failure.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException calling Bedrock | Model access not yet granted, or IAM role missing bedrock:InvokeModel | Complete Step 1; confirm the IAM policy from Section 4.1 includes the Bedrock statement |
| ValidationException: model identifier is invalid | Wrong model ID format used | Confirm BEDROCK_MODEL_ID is exactly us.amazon.nova-lite-v1:0, including the us. prefix |
| 429 on every request, even right after requesting access | On-demand quota is separate from model access and defaults to 0 on some new accounts | File an AWS Support case under Service Quotas; use MOCK_SUMMARIZE locally in the meantime |
| Lambda call succeeds locally (mocked) but fails when deployed | Local FastAPI app and deployed Lambda are two different code paths — mocking one doesn't affect the other | Expected behavior; see the MOCK_SUMMARIZE distinction above |