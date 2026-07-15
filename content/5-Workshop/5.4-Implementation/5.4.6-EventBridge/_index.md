---
title : "Weekly Report Pipeline — EventBridge"
date : 2026-07-14
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

#### Goal

Automatically generate a weekly report of summarization activity, using EventBridge to trigger a dedicated Lambda function on a schedule, with output stored in S3 and transitioned to cheaper storage over time.

#### Step 1 — Create the S3 Reports Bucket

1. Open the **S3** console → **Create bucket**.
2. Bucket name: doc-summarizer-reports-YOUR_ACCOUNT_ID (globally unique, suffixed with your account ID).
3. Region: ap-southeast-1.
4. Leave **Block Public Access settings** fully checked — reports are accessed by the report Lambda and by direct download, never publicly.
5. Under **Default encryption**, select **Server-side encryption with Amazon S3 managed keys (SSE-S3)** — this is AES256 encryption.
6. Click **Create bucket**.

#### Step 2 — Add a Lifecycle Rule

Reports older than 30 days move to Glacier, where storage is significantly cheaper — appropriate for records that are rarely accessed after the first month but still worth retaining.

1. Open the bucket → **Management** tab → **Create lifecycle rule**.
2. Rule name: reports-glacier-transition.
3. Rule scope: **Apply to all objects in the bucket**.
4. Under **Lifecycle rule actions**, check **Move current versions of objects between storage classes**.
5. Add a transition: **30 days after object creation** → **Glacier Flexible Retrieval**.
6. Click **Create rule**.

**How to verify:** The **Management** tab lists reports-glacier-transition with status **Enabled**.

#### Step 3 — Create the IAM Role for the Report Lambda

1. **IAM** console → **Roles** → **Create role**.
2. Trusted entity: **AWS service** → **Lambda**. Click **Next**.
3. Skip attaching managed policies. Click **Next**.
4. Role name: doc-summarizer-report-lambda-role. Click **Create role**.
5. Click into the role → **Add permissions** → **Create inline policy** → **JSON** tab:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "logs:CreateLogGroup",
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:*"
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:log-group:/aws/lambda/doc-summarizer-report-fn:*"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:Query"],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:YOUR_ACCOUNT_ID:table/SummarizerTable",
        "arn:aws:dynamodb:ap-southeast-1:YOUR_ACCOUNT_ID:table/SummarizerTable/index/summary-date-index"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": "arn:aws:s3:::doc-summarizer-reports-YOUR_ACCOUNT_ID/*"
    }
  ]
}
```

6. Click **Next**, name it doc-summarizer-report-lambda-policy, click **Create policy**.

The summary-date-index GSI (built in Section 4.1) is what makes this efficient — the report Lambda queries by date range instead of scanning the entire table.

#### Step 4 — Create the Report Lambda Function

1. **Lambda** console → **Create function**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Function name | doc-summarizer-report-fn |
   | Runtime | Python 3.12 |
   | Architecture | x86_64 |

3. Under **Permissions**, select **Use an existing role** → doc-summarizer-report-lambda-role.
4. Click **Create function**.
5. **Configuration** → **General configuration** → **Edit** → set **Timeout** to 60 sec (queries + report generation can take longer than a single summarization call). Set **Memory** to 256 MB. Click **Save**.
6. **Configuration** → **Environment variables** → add:

   | Key | Value |
   |---|---|
   | DYNAMODB_TABLE | SummarizerTable |
   | REPORTS_BUCKET | doc-summarizer-reports-YOUR_ACCOUNT_ID |

7. **Code** tab → paste the report generation logic (queries summary-date-index for the past 7 days, aggregates counts/summary lengths, writes a report object to the reports bucket) → **Deploy**.

#### Step 5 — Create the EventBridge Schedule

1. Open the **EventBridge** console → **Rules** → **Create rule**.
2. Name: doc-summarizer-weekly-report.
3. Rule type: **Schedule**.
4. Click **Next**.
5. Schedule pattern: **A fine-grained schedule** → **Cron-based schedule**:
   ```
   cron(0 8 ? * MON *)
   ```
   This runs every Monday at 08:00 UTC.
6. Click **Next**.
7. Target: **AWS Lambda function** → doc-summarizer-report-fn.
8. Click **Next** → **Next** → **Create rule**.

**How to verify:** The rule shows **State: Enabled** and **Next scheduled run** with a Monday 08:00 UTC date.

#### Step 6 — Test Manually

Rather than waiting until the next Monday, invoke the report Lambda directly to confirm the pipeline works end to end.

1. **Lambda** console → doc-summarizer-report-fn → **Test** tab.
2. Create a test event with an empty JSON body {} (EventBridge schedule triggers pass no meaningful payload).
3. Click **Test**.
4. Open the **S3** reports bucket → confirm a new report object appears.

**How to verify:** Execution result: succeeded, and the reports bucket contains a newly created object with a recent timestamp in its key or metadata.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| EventBridge rule shows **Next scheduled run** but Lambda never executes | Missing lambda:InvokeFunction resource-based permission for EventBridge to invoke the function | Lambda console → function → **Configuration** → **Permissions** → confirm a resource-based policy statement exists for events.amazonaws.com; if not, recreate the rule target and accept the "Add permission" prompt |
| Report Lambda times out | Query pattern falls back to a full table scan instead of using the GSI | Confirm the code queries summary-date-index with a KeyConditionExpression, not table.scan() |
| AccessDenied writing to S3 | Bucket name in REPORTS_BUCKET doesn't match the IAM policy resource ARN | Confirm both use the exact same bucket name including the account ID suffix |
| Reports appear but never transition to Glacier | Lifecycle rule wasn't created, or objects are younger than 30 days | Confirm the lifecycle rule (Step 2) is **Enabled**; transitions only apply after the 30-day threshold |