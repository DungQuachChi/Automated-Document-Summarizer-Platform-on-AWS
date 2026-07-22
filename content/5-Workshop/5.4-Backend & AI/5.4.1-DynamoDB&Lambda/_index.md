---
title : "Backend Foundation — DynamoDB & Lambda"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.4.1. "
---

#### Goal

Build the storage layer (DynamoDB table with a GSI for date-based queries) and the compute layer (a Lambda function with a least-privilege IAM role) that everything else in this workshop connects to — both defined in Terraform (terraform/modules/data, terraform/modules/compute).

#### DynamoDB Table

terraform/modules/data/main.tf:

![overview](/images/5-Workshop/5.4-S3-onprem/The_DynamoDB_Table.jpeg)

- PAY_PER_REQUEST billing — no idle cost, no capacity planning.
- Encryption at rest, point-in-time recovery (35-day restore window).
- GSI on summary_date — needed by the weekly report Lambda (Section 5.6.1) to query "all summaries in the last 7 days across all users," which the base table's user_id partition key can't answer efficiently alone.

#### Lambda IAM Role

terraform/modules/compute/main.tf builds a role scoped to discrete, resource-level statements — no wildcard actions or resources beyond what cross-region Bedrock inference requires:

- DynamoDB: PutItem + Query only — no Scan, no DeleteItem — matching exactly what the two code paths (handle_summarize, handle_history) call.
- Bedrock: two specific ARNs (foundation model + inference profile), not a wildcard — detailed in Section 5.4.2.
- cloudwatch:PutMetricData uses a * resource — a CloudWatch API limitation (custom metrics don't support resource-level ARNs), not a scoping choice.

#### Lambda Function

The log group is created explicitly (aws_cloudwatch_log_group, 7-day retention) and depended on before the function, rather than relying on Lambda's auto-created group with no expiry. Timeout is 30 seconds — Bedrock calls plus retry backoff need headroom beyond the default 3s.

**Known dev-only shortcut, stated plainly rather than hidden:** if no Cognito claims are present on the request, user_id falls back to a hardcoded test value. This exists for local/manual testing and should be removed before any real production deployment.

#### Deploy and Test

```bash
cd terraform
terraform init
terraform apply
```

```bash
aws lambda invoke \
  --function-name doc-summarizer-summarizer \
  --payload '{"httpMethod":"POST","path":"/summarize","body":"{\"text\":\"Amazon Web Services offers cloud computing services worldwide. Businesses use AWS to reduce infrastructure costs and scale applications globally.\"}","requestContext":{"authorizer":{"claims":{"sub":"test-user-123"}}}}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json
```

**How to verify:** response.json shows "statusCode": 200 with summary, timestamp, and summary_date fields. A 502 here means Bedrock model access hasn't been granted yet — see Section 5.4.2.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException on DynamoDB | dynamodb_table_arn not passed correctly into the compute module | Confirm root main.tf wires module.data.table_arn into module.compute |
| ResourceNotFoundException | Lambda's DYNAMODB_TABLE env var doesn't match the actual table name | Check module.data.table_name output matches what's passed to module.compute |
| terraform apply fails creating the Lambda: "no such file" | var.lambda_zip_path doesn't point to a built deployment package | Zip src/lambda_fn/ before running terraform apply |