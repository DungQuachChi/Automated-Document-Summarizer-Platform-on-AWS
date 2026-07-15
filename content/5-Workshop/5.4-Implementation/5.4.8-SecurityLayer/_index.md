---
title : "Monitoring & Security Layer"
date : 2026-07-14
weight : 8
chapter : false
pre : " <b> 5.4.8. </b> "
---

#### Goal

Set up visibility into the running system (dashboard, alarms, custom metrics) and a security/compliance baseline (audit logging, configuration rules) so problems and drift are caught rather than discovered later.

#### Step 1 — Create an SNS Topic for Alerts

1. Open the **SNS** console → **Topics** → **Create topic**.
2. Type: **Standard**.
3. Name: doc-summarizer-alerts.
4. Click **Create topic**.
5. Click **Create subscription**.
6. Protocol: **Email**. Endpoint: your email address.
7. Click **Create subscription**.
8. Check your email and click **Confirm subscription** — alarms will not deliver until this is confirmed.

#### Step 2 — Create CloudWatch Alarms

**Lambda errors:**

1. **CloudWatch** console → **Alarms** → **Create alarm**.
2. Select metric: **Lambda** → **By Function Name** → doc-summarizer-fn → **Errors**.
3. Condition: **Greater than** 0 for 1 consecutive period of 5 minutes.
4. Notification: select doc-summarizer-alerts SNS topic.
5. Alarm name: doc-summarizer-lambda-errors.
6. Click **Create alarm**.

**API Gateway 5xx errors:**

1. **Create alarm** → select metric: **ApiGateway** → doc-summarizer-api → **5XXError**.
2. Condition: **Greater than** 0 for 1 consecutive period of 5 minutes.
3. Notification: doc-summarizer-alerts.
4. Alarm name: doc-summarizer-api-5xx.
5. Click **Create alarm**.

**Bedrock custom metric errors:**

1. **Create alarm** → select metric: **Custom/Bedrock** namespace → BedrockErrors.
2. Condition: **Greater than** 0 for 1 consecutive period of 5 minutes.
3. Notification: doc-summarizer-alerts.
4. Alarm name: doc-summarizer-bedrock-errors.
5. Click **Create alarm**.

**How to verify:** All three alarms show state **OK** (green) in the **Alarms** list once there's been at least one evaluation period with no errors.

#### Step 3 — Deliberately Trigger an Alarm

Confirm the alert path actually works end to end rather than assuming it does.

1. Temporarily set the Lambda's memory (Section 4.1) very low, or send a malformed request repeatedly to force errors — or, simpler, manually publish a test data point:
   ```bash
   aws cloudwatch put-metric-data \
     --namespace Custom/Bedrock \
     --metric-name BedrockErrors \
     --value 1 \
     --profile phatnguyen
   ```
2. Wait for the alarm's evaluation period to elapse.
3. Confirm the alarm state changes to **In alarm** in the CloudWatch console.
4. Confirm the alert email arrives.

**How to verify:** An email from AWS Notifications arrives referencing the triggered alarm name.

#### Step 4 — Build a CloudWatch Dashboard

1. **CloudWatch** console → **Dashboards** → **Create dashboard**.
2. Name: doc-summarizer-dashboard.
3. Add widgets:
   - **Line graph**: Lambda Duration, Invocations, Errors for doc-summarizer-fn
   - **Line graph**: API Gateway Count, 4XXError, 5XXError for doc-summarizer-api
   - **Line graph**: Custom/Bedrock Latency, SuccessCount, BedrockErrors
   - **Number widget**: DynamoDB ConsumedWriteCapacityUnits for SummarizerTable
4. Click **Save dashboard**.

**How to verify:** Generate a few real requests against /summarize and /history (Section 5), then reload the dashboard — the graphs show real, non-zero traffic within a few minutes.

#### Step 5 — Enable CloudTrail

1. **CloudTrail** console → **Trails** → **Create trail**.
2. Trail name: doc-summarizer-trail.
3. Check **Enable for all accounts in my organization** only if applicable — for a single-account project, leave unchecked.
4. Under **Storage location**, create a new S3 bucket for logs, or use an existing one.
5. Under **Multi-Region trail**, ensure this is enabled — this is not a toggle you set separately, it's implied by trail configuration in the current console; confirm the region dropdown shows **All regions**.
6. Click **Next** → leave management/data event defaults → **Next** → **Create trail**.

**How to verify:** The trail shows **Logging: On**. After a few minutes, the designated S3 bucket contains log files under a AWSLogs/YOUR_ACCOUNT_ID/CloudTrail/ prefix.

#### Step 6 — Enable AWS Config and the CIS Conformance Pack

1. **AWS Config** console → **Get started** (or **Settings** if already partially configured).
2. Under **Resource types to record**, select **Record all resources supported in this region**.
3. Under **Amazon S3 bucket**, select or create a bucket for configuration history.
4. Under **Amazon SNS topic**, optionally reuse doc-summarizer-alerts.
5. Click **Next** → **Confirm**.
6. Once recording is active, go to **Conformance packs** → **Deploy conformance pack**.
7. Select the sample template **Operational Best Practices for CIS AWS Foundations Benchmark v1.4 Level 1**.
8. Click **Deploy conformance pack**. This can take 15–20 minutes to fully evaluate all rules.

**How to verify:** The **Conformance packs** page shows compliance status once evaluation completes. This project's current status is **60% compliant**, with two accepted gaps:

- CodeBuild's IAM role uses AdministratorAccess rather than a scoped least-privilege policy
- MFA is not enforced account-wide via IAM policy (it is enabled on the individual admin user from Prerequisites, but not conditionally required for all users)

These are documented trade-offs for a learning-budget project — not unnoticed failures — and are revisited in Section 6.2 (Simple Security Hardening).

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Alarm never leaves **Insufficient Data** state | No data points have been published to the metric yet | Generate real traffic (Section 5) or manually publish a test metric (Step 3) |
| SNS email never arrives after an alarm fires | Subscription was never confirmed | Check spam/junk folder for the original confirmation email; re-subscribe if it expired |
| CloudTrail bucket shows no log files after 15+ minutes | Trail wasn't actually enabled, or bucket policy is missing required CloudTrail permissions | Confirm **Logging: On** in the trail overview; if a custom bucket was used instead of a CloudTrail-created one, verify its bucket policy grants CloudTrail write access |
| Conformance pack stuck at **Evaluating** for a long time | Normal for the first run — depends on number of resources in the account | Wait; check back after 20–30 minutes rather than re-deploying |
| Conformance pack compliance percentage lower than expected | Some CIS rules apply to resources or settings outside this project's scope (e.g. default VPC rules) | Review individual rule results before assuming a gap is project-related — some flagged items may be pre-existing account defaults unrelated to this workshop |