---
title : "Testing & Measurement"
date : 2026-07-14
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Logs

**Lambda logs.** Every invocation of doc-summarizer-fn and doc-summarizer-report-fn writes to CloudWatch Logs automatically (via the logs:CreateLogGroup/logs:PutLogEvents permissions granted in Sections 4.1 and 4.6).

1. **CloudWatch** console → **Log groups** → /aws/lambda/doc-summarizer-fn.
2. Click into the most recent log stream to see individual invocation output, including the print() statements in the Lambda code (e.g. Bedrock API Error: ... when a call fails).

**API Gateway execution logs.** Disabled by default — must be explicitly enabled per stage.

1. **API Gateway** console → doc-summarizer-api → **Stages** → v1 → **Logs and tracing** tab.
2. Enable **CloudWatch Logs**, log level **INFO** or **ERROR**, and (optionally) **Log full requests/responses data** for debugging — turn this off again afterward, since it can log sensitive request bodies.
3. This requires an IAM role with CloudWatch Logs permissions attached to API Gateway at the account level (**API Gateway** → **Settings** → **CloudWatch log role ARN**) — a common gap that silently prevents logs from appearing even when enabled per-stage.

**CodeBuild logs.** Each pipeline stage's CodeBuild project streams its build output to CloudWatch Logs automatically.

1. **CodePipeline** console → doc-summarizer-pipeline → click a stage → **Details** link on the relevant action.
2. This opens the CodeBuild build page, with the full command output (pytest results, bandit/tfsec findings, terraform plan output).

**Reading a failed pipeline stage:** scroll to the first line containing FAILED or a non-zero exit code — CodeBuild logs are chronological, and the actual failure is usually near the bottom, not the top, since later steps still attempt to run cleanup even after a failure earlier in the phase.

#### Metrics

**Built-in metrics** (no extra code required):

| Metric | Source | What it tells you |
|---|---|---|
| Duration, Errors, Throttles, Invocations | Lambda | Execution time, failure rate, concurrency limits being hit |
| 4XXError, 5XXError, Count, Latency | API Gateway | Client vs server error rates, overall traffic volume |
| ConsumedReadCapacityUnits, ConsumedWriteCapacityUnits | DynamoDB | Actual on-demand usage — useful for cost tracking |

**Custom metrics** — the Custom/Bedrock namespace, emitted explicitly by the Lambda code (Section 4.1):

| Metric | Meaning |
|---|---|
| Latency | Milliseconds for the Bedrock invoke_model call to complete |
| SuccessCount | Incremented on every successful Bedrock call |
| BedrockErrors | Incremented on any Bedrock failure, dimensioned by ErrorType (e.g. DailyQuotaExceeded, ThrottlingException) |

The ErrorType dimension is what makes it possible to distinguish "hit the daily quota" from "generic throttling" from "model timeout" on the dashboard, rather than lumping all Bedrock failures into one undifferentiated error count.

#### Alerts

Section 4.8 covered creating the alarms and confirming delivery. As a recap of the verification step: publish a test data point to Custom/Bedrock / BedrockErrors manually, wait for the alarm's evaluation period, and confirm both the CloudWatch alarm state changes to **In alarm** and the SNS email arrives. This is worth re-running here as part of the overall test pass, not just once during initial setup — alarm configurations can silently break (e.g. an expired SNS subscription) without any obvious symptom until the moment they're actually needed.

#### Manual Test Plan

Using curl against the deployed API (https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1):

**1. Valid request with auth:**
```bash
curl -X POST https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "Content-Type: application/json" \
  -H "Authorization: YOUR-ID-TOKEN" \
  -H "x-api-key: YOUR-API-KEY" \
  -d '{"text": "Amazon Web Services provides a broad set of global cloud services including compute, storage, databases, and AI tools."}'
```
Expected: 200 with a summary field (or 429 if the daily Bedrock quota is currently exhausted — see Section 4.4).

**2. Missing token:**
```bash
curl -X POST https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "x-api-key: YOUR-API-KEY" \
  -d '{"text": "test"}'
```
Expected: 401 Unauthorized — confirms the Cognito authorizer is enforced.

**3. Missing API key:**
```bash
curl -X POST https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "Authorization: YOUR-ID-TOKEN" \
  -d '{"text": "test"}'
```
Expected: 403 Forbidden — confirms the usage plan's API key requirement is enforced.

**4. GET /history:**
```bash
curl https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1/history \
  -H "Authorization: YOUR-ID-TOKEN" \
  -H "x-api-key: YOUR-API-KEY"
```
Expected: 200 with a `history` array containing prior entries for this user.

**5. Input validation boundaries:**
```bash
# Too short (under 20 chars)
curl -X POST .../summarize -H "..." -d '{"text": "short"}'
# Expected: 400

# Too long (over 5000 chars)
curl -X POST .../summarize -H "..." -d '{"text": "'"$(python3 -c 'print("a"*5001)')"'"}'
# Expected: 400
```

#### Automated Tests

The tests/ directory contains pytest tests that exercise the Lambda handler functions directly, using moto to mock DynamoDB rather than hitting a real AWS account.

1. Install dependencies:
   ```bash
   pip install -r requirements-test.txt --break-system-packages
   ```
2. Run the suite:
   ```bash
   pytest tests/ -v
   ```
3. Typical coverage: input validation boundaries (too short, too long, empty), the summary_date field being correctly derived and written on every PutItem, handle_history returning results sorted newest-first, and the DailyQuotaExceededError path returning a 429 rather than propagating as a 500.

moto intercepts boto3 calls and simulates DynamoDB in-memory, so these tests run in CI (Section 4.7's Test stage) without needing real AWS credentials or touching the actual SummarizerTable.

**How to verify:** pytest tests/ -v exits with code 0 and a summary line like X passed in Y s.

#### Load Testing

locustfile.py can run in two distinct modes, and it's important to know which one produced a given result before drawing conclusions from it:

- **MOCK_MODE=true** (the default) — auth is skipped in favor of a fixed placeholder Bearer token, and requests go wherever --host points. This is for exercising the request/response contract and concurrency handling without depending on Cognito or Bedrock being available.
- **MOCK_MODE=false** — the script calls cognito-idp:InitiateAuth with USER_PASSWORD_AUTH directly via boto3 (an unauthenticated/unsigned API call, not the Hosted UI's /oauth2/token endpoint, which only accepts authorization_code/refresh_token grants) to get a real JWT, and every request carries the real x-api-key header. This is the only mode that actually exercises Bedrock.

**Mock-mode baseline** (auth mocked; target is whatever --host is set to — the local Flask/gunicorn stand-in on localhost:8000, or the real API Gateway endpoint with MOCK_BEDROCK enabled server-side):

| Users | Total requests | Avg | p95 | p99 | Failures |
|---|---|---|---|---|---|
| 10 | 150 | 4 ms | 6 ms | 7 ms | 0% |
| 50 | 11,363 | 4 ms | 5 ms | 7 ms | 0% |

The 50-user run held steady at ~24 req/s with zero failures across /health, /history, and /summarize for the full ramp-and-hold duration — response times don't degrade with 5x the concurrent users, which indicates the bottleneck found below is specific to real Bedrock calls, not the request handling, auth path, or DynamoDB access pattern.

**Running it:**
```bash
pip install boto3 locust --break-system-packages
locust -f locustfile.py --headless -u 50 -r 1 --run-time 60s --host <target-host>
```
Set `--host` to `http://localhost:8000` for the mock server, or the real API Gateway invoke URL to test mock-mode auth against the live endpoint.

**Real-API setup** (MOCK_MODE=false), for exercising the actual Bedrock-backed path:

1. Create a test Cognito user for load testing:
   ```bash
   AWS_PROFILE=phatnguyen aws cognito-idp admin-create-user \
     --user-pool-id ap-southeast-1_Uo593E4hR \
     --username loadtest@example.com
   AWS_PROFILE=phatnguyen aws cognito-idp admin-set-user-password \
     --user-pool-id ap-southeast-1_Uo593E4hR \
     --username loadtest@example.com \
     --password YourStrongPass123! \
     --permanent
   ```
2. Set up .env.test:
   ```
   MOCK_MODE=false
   COGNITO_REGION=ap-southeast-1
   CLIENT_ID=7mfke3ntkous3rbvpbqpu7c2nb
   TEST_USERNAME=loadtest@example.com
   TEST_PASSWORD=YourStrongPass123!
   API_KEY=<value from: terraform output -raw api_key_value>
   ```
3. Run Locust against the real endpoint:
   ```bash
   locust -f locustfile.py --host https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1
   ```
4. Open the Locust web UI (default `http://localhost:8089`), set number of users and ramp-up rate, start the test.

**What "good" looks like:** GET /history should show 0 failures with a low median latency (sub-500ms is reasonable for a DynamoDB query behind API Gateway + Lambda). POST /summarize latency depends entirely on Bedrock — a healthy result is consistent 2xx responses in the low single-digit seconds.

**A real finding from this project's load testing:** a run of 5 concurrent users ramping at 1/sec in real-API mode showed GET /history performing correctly, while every POST /summarize request failed with 502/504 at almost exactly 29 seconds — API Gateway's hard integration timeout ceiling. Root cause: the Lambda's retry logic (Section 4.1) was retrying Bedrock throttling errors with exponential backoff, consuming the entire 30-second Lambda timeout before ever returning a response. The mock-mode results above rule out the request/auth path as the cause — 50 concurrent users produced zero failures under mock mode, so the 502/504s are specific to real Bedrock invocations, not a concurrency or code-path issue. The fix (fast-failing on daily-quota errors specifically, rather than retrying them) is the retry logic shown in Section 4.1.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| pytest fails with NoRegionError or real AWS calls being attempted | moto mock decorator missing on a test, or DynamoDB resource created outside the mocked context | Confirm the test function is wrapped with the moto mock (e.g. @mock_aws or equivalent for your moto version) before any boto3.resource('dynamodb') call |
| API Gateway execution logs stay empty even after enabling them per-stage | Missing account-level CloudWatch Logs role for API Gateway | Set **API Gateway → Settings → CloudWatch log role ARN** to a role with logs:* permissions |
| Locust InitiateAuth fails with NotAuthorizedException | Test user password wasn't set as permanent, or USER_PASSWORD_AUTH isn't enabled on the app client | Confirm admin-set-user-password --permanent was used, and that Section 4.2 Step 6 (auth flows) was completed |
| POST /summarize consistently times out around 29s under load | Bedrock retry logic burning the full Lambda timeout on quota errors | Confirm the daily-quota fast-fail path from Section 4.1 is deployed, not an older version without it |
| A locust run reports mode: MOCK in its startup log | MOCK_MODE env var (or its default) is true | Expected for a mock-path baseline; if a real-API result was intended, set MOCK_MODE=false and confirm .env.test is loaded before re-running |