---
title : "Backend Foundation — DynamoDB & Lambda"
date : 2026-07-14
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### Goal

Create the storage layer (DynamoDB table with a GSI for date-based queries) and the compute layer (a Lambda function with a least-privilege IAM role) that everything else in this workshop connects to — both defined in Terraform (terraform/modules/data and terraform/modules/compute), not created by hand in the console.

#### Step 1 — The DynamoDB Table (Terraform)

terraform/modules/data/main.tf defines the table with on-demand billing, encryption at rest, point-in-time recovery, and a GSI for date-range queries:

![overview](/images/5-Workshop/5.4-S3-onprem/The_DynamoDB_Table.jpeg)

PAY_PER_REQUEST billing means no idle cost and no capacity planning — reads/writes are billed per request, which fits a project with unpredictable, low-volume traffic. Point-in-time recovery lets the table be restored to any point in the last 35 days if data is accidentally deleted.

The GSI on summary_date exists specifically for the weekly report Lambda (Section 5.4.6) — it needs to query "all summaries in the last 7 days across all users," which the base table's user_id partition key can't answer efficiently on its own.

#### Step 2 — The Lambda IAM Role (Terraform)

terraform/modules/compute/main.tf builds the role with an assume-role policy scoped to the Lambda service, then attaches an inline policy built from discrete, resource-scoped statements — no wildcard actions, no wildcard resources beyond what cross-region Bedrock inference requires:

![overview](/images/5-Workshop/5.4-S3-onprem/The_DynamoDB_Table.jpeg)

Three points worth calling out in the report: the DynamoDB statement only allows PutItem and Query — no Scan, no DeleteItem — because that's all the Lambda's two code paths (handle_summarize, handle_history) ever call. The Bedrock resource list is two ARNs, not a wildcard, matched to the exact model and inference profile in use. PutMetricData needs a * resource because CloudWatch custom metrics don't support resource-level ARNs — this is a CloudWatch API limitation, not a scoping choice.

#### Step 3 — The Lambda Function (Terraform)

![overview](images/5-Workshop/5.4-S3-onprem/CloudWatchCloudWatch.jpeg)

The log group is created explicitly and depended on before the function, rather than letting Lambda auto-create it — this fixes its retention at 7 days from the start instead of defaulting to "never expire."

Timeout is set to 30 seconds because Bedrock calls, plus retry backoff, can take several seconds; the function needs headroom beyond the default 3-second Lambda timeout.

#### Step 4 — The Application Code

src/lambda_fn/lambda_function.py is what actually runs inside the function above. A few implementation details worth including in the report:

```python
# ap-southeast-1 is not in the AP Nova inference pool; route cross-region to us-east-1
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

The Lambda itself runs in ap-southeast-1, but the Bedrock client is hardcoded to us-east-1 because Nova Lite's inference pool doesn't cover the AP region yet — this is also why the IAM policy above grants the us-east-1 inference-profile ARN specifically.

Error handling distinguishes daily-quota exhaustion from transient errors:

![overview](/images/5-Workshop/5.4-S3-onprem/daily_quotaquota.jpeg)

Transient throttling retries up to 3 times with exponential backoff (1s, 2s, 4s). A daily quota error is detected separately and fails fast instead of retrying — retrying a quota error just burns the Lambda's 30-second timeout window for no benefit, since the quota won't reset mid-request.

One item flagged directly in the code rather than hidden: if no Cognito claims are present on the request, user_id falls back to a hardcoded test value. This exists for local/manual testing and is explicitly a dev-only path — worth stating plainly in the report as a known item to remove before any real production deployment, not something to leave unmentioned.

#### Step 5 — Deploy and Test

```bash
cd terraform
terraform init
terraform apply
```

To verify the deployed function directly:

```bash
aws lambda invoke \
  --function-name doc-summarizer-summarizer \
  --payload '{"httpMethod":"POST","path":"/summarize","body":"{\"text\":\"Amazon Web Services offers cloud computing services worldwide. Businesses use AWS to reduce infrastructure costs and scale applications globally.\"}","requestContext":{"authorizer":{"claims":{"sub":"test-user-123"}}}}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json
```

**How to verify:** response.json shows "statusCode": 200 with summary, timestamp, and summary_date fields in the body. If Bedrock model access hasn't been granted yet (Section 5.4.4), this call fails with a 502 — expected at this point in the workshop.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException on DynamoDB | dynamodb_table_arn variable not passed correctly into the compute module | Confirm the root main.tf wires module.data.table_arn into module.compute |
| ResourceNotFoundException | Lambda's DYNAMODB_TABLE env var doesn't match the actual table name | Check module.data.table_name output matches what's passed to module.compute |
| 502 AI Summarization service is currently unavailable | Bedrock model access not yet granted, or IAM policy ARN doesn't match the model/region in use | Complete Section 5.4.4 (Bedrock model access request) |
| 429 Summarization limit reached for today | Daily on-demand quota for the model has been exhausted | Expected behavior — see Section 5.4.4 for the quota workaround |
| terraform apply fails creating the Lambda: "no such file" | var.lambda_zip_path doesn't point to a built deployment package | Zip src/lambda_fn/ before running terraform apply, or use the packaging step in the deployment script if one exists |