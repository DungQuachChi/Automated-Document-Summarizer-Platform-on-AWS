---
title : "Weekly Report Pipeline — EventBridge"
date : 2026-07-14
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

#### Goal

Generate a weekly report of summarization activity automatically: EventBridge triggers a dedicated Lambda on a schedule, the Lambda queries the last 7 days of activity via the summary-date-index GSI, and the result is written to S3 as a CSV, with older reports moved to cheaper storage over time.

#### Terraform Resources

The reports bucket is defined in modules/data, alongside the main table (Section 4.1) — not in the scheduling module itself, since it's data storage rather than a scheduling concern:

- aws_s3_bucket — ${project_name}-reports-${account_id}, private, SSE-S3 encryption, all public access blocked.
- aws_s3_bucket_lifecycle_configuration — one rule, archive-to-glacier, transitioning all objects to GLACIER storage class 30 days after creation.

modules/scheduling defines the compute and trigger side:

- aws_iam_role + aws_iam_role_policy for the report Lambda, scoped to exactly three things: writing its own CloudWatch logs, dynamodb:Query against the table and the GSI specifically (not the whole table, and not Scan), and s3:PutObject against the reports bucket. The s3:PutObject resource is ${reports_bucket_arn}/* rather than a specific key, since report keys are date-generated at runtime (reports/weekly-report-{date}.csv) and can't be known in advance — the ignored tfsec wildcard warning is scoped to this one bucket only, not account-wide.
- aws_lambda_function — Python 3.12, 60-second timeout, 256 MB memory, packaged from report_lambda/ via archive_file.
- aws_cloudwatch_event_rule — schedule_expression = "cron(0 8 ? * MON *)", state = "ENABLED".
- aws_cloudwatch_event_target — binds the rule to the Lambda function ARN.
- aws_lambda_permission — grants events.amazonaws.com lambda:InvokeFunction, scoped to this rule's ARN via source_arn. Without this, the rule fires but Lambda silently refuses the invocation.

Applying modules/scheduling and modules/data provisions all of this in one pass — the rule, the target, and the invoke permission are created together, so there's no separate step where the schedule exists but nothing is wired to it.

#### Report Lambda Logic

report_lambda/report_handler.py, in outline:

1. get_last_7_days_dates() builds seven YYYY-MM-DD strings.
2. For each date, query_dynamodb_by_date() queries summary-date-index with KeyConditionExpression: summary_date = :date, paginating on LastEvaluatedKey in case a single day's results exceed 1 MB.
3. aggregate_by_user() groups results by user_id, skipping any item missing user_id or original_text, and accumulates request count plus total original/summary text length per user.
4. generate_csv() writes one row per user: user_id, total_requests, avg_original_length, avg_summary_length, first_request_time, last_request_time.
5. The CSV is uploaded to reports/weekly-report-{today's date}.csv in the reports bucket.
6. If no items are found for the past 7 days, the function returns early with a 200 and a "no data" message rather than uploading an empty report.

Environment variables are DYNAMODB_TABLE and REPORT_BUCKET (singular — not REPORTS_BUCKET), set from Terraform outputs (module.data.table_name, module.data.reports_bucket_name), so there's no risk of the Lambda's env var drifting from the bucket Terraform actually created.

Logging is structured JSON (a small JSONLogger wrapper around logging), not print statements — each log line carries a level, message, and relevant context (date queried, item counts, error detail on failure), which is what CloudWatch Logs Insights queries against in Section 5.4.8.

#### Applying

```bash
terraform apply
```

#### Testing Manually

Rather than waiting for the next Monday 08:00 UTC run:

```bash
aws lambda invoke \
  --function-name $(terraform output -raw report_lambda_function_name) \
  --payload '{}' \
  response.json
cat response.json
```

Then confirm the object landed in S3:

```bash
aws s3 ls s3://$(terraform output -raw reports_bucket_name)/reports/
```

**How to verify:** response.json shows "statusCode": 200, and the reports bucket contains a new weekly-report-{date}.csv object. If the past 7 days have no real activity (for example, while running under MOCK_BEDROCK=true with no live summarize calls), the function still returns 200 with a "no data to report" body — that's expected behavior, not a failure.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| EventBridge rule shows a next scheduled run, but the Lambda never executes | aws_lambda_permission wasn't applied, or was later removed manually outside Terraform | terraform plan should show no drift; if it does, terraform apply to restore it — the permission is templated to this one rule's ARN, not a wildcard |
| Report Lambda times out | Query pattern isn't matching the GSI's key schema, and DynamoDB silently returns empty pages instead of the expected date's items until pagination logic loops longer than expected | Confirm the GSI's partition key is summary_date (Section 4.1) and that items are actually being written with that attribute set |
| AccessDenied writing to S3 | REPORT_BUCKET env var doesn't match the bucket the IAM policy scopes to | Both come from the same Terraform output (module.data.reports_bucket_name) — if they've drifted, something was changed manually outside Terraform; re-apply |
| Reports never transition to Glacier | Objects are younger than the 30-day threshold, or the lifecycle rule isn't enabled | aws s3api get-bucket-lifecycle-configuration --bucket <reports-bucket> — confirm archive-to-glacier shows "Status": "Enabled" |