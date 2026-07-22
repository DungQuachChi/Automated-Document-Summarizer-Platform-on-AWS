---
title : "Automation — Weekly Reports & CI/CD"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.6.1. "
---

#### Goal

Two things that run on their own once deployed: a scheduled Lambda that generates weekly usage reports, and a CI/CD pipeline that tests and gates every infrastructure change.

### Weekly Report Pipeline (EventBridge)

**Reports bucket** (modules/data): private, SSE-S3 encrypted, lifecycle rule transitioning objects to Glacier 30 days after creation.

**Scheduling** (modules/scheduling):
- IAM role scoped to exactly three actions: its own CloudWatch logs, dynamodb:Query against the table + summary-date-index GSI specifically, and s3:PutObject against the reports bucket.
- aws_lambda_function — Python 3.12, 60s timeout, 256MB.
- aws_cloudwatch_event_rule (cron(0 8 ? * MON *), ENABLED) + aws_cloudwatch_event_target + aws_lambda_permission — all three created together in one apply, so the schedule is never left unwired.

**Report Lambda logic**: queries summary-date-index for each of the last 7 days, aggregates by user_id, writes one CSV row per user. Env vars come from Terraform outputs, not hand-typed. No activity in the window → returns 200 with a "no data" message rather than an empty file.

```bash
terraform apply
aws lambda invoke --function-name $(terraform output -raw report_lambda_function_name) --payload '{}' response.json
aws s3 ls s3://$(terraform output -raw reports_bucket_name)/reports/
```

### CI/CD Pipeline (CodePipeline)

```
GitHub push (main) → Source → Test (pytest, bandit, tfsec, terraform fmt, terraform plan)
                             → Approve (human reviews the plan) → Apply (terraform apply)
```

The pipeline itself is Terraform (modules/pipeline). The tfplan.binary artifact passes straight from Test into Apply — nothing is re-planned after approval, so what's reviewed is exactly what's applied.

**One-time setup:**
1. Bootstrap remote state — S3 bucket with versioning + DynamoDB lock table.
2. The GitHub connection needs a one-time manual authorization: **CodePipeline → Settings → Connections → Update pending connection**.

**IAM:** CodePipeline's own role is scoped narrowly. CodeBuild's role currently uses AdministratorAccess — flagged as a known scope-down item, not hidden.

```bash
cd terraform && terraform apply
```
Then push a small change to main and watch Source → Test → Approve → Apply go green in the CodePipeline console.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| EventBridge rule scheduled but Lambda never runs | aws_lambda_permission missing or removed outside Terraform | terraform plan should show no drift; apply to restore |
| Reports never reach Glacier | Objects younger than 30 days, or lifecycle rule disabled | aws s3api get-bucket-lifecycle-configuration — confirm Enabled |
| Source stage fails immediately | CodeStar connection stuck at **Pending** | Complete the GitHub authorization handshake |
| Test stage fails with a state lock error | Previous run didn't release the DynamoDB lock | terraform force-unlock LOCK_ID |
| Apply stage fails mid-apply on a permissions error | CodeBuild role lacks a permission for a newly added resource type | Known gap under AdministratorAccess — see Section 5.6.2 |