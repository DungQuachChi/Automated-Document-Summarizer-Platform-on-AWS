---
title : "Backend Foundation — DynamoDB & Lambda"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.4.1. "
---

#### Goal

Build the storage layer and the compute layer  that everything else in this workshop connects to both defined in Terraform (terraform/modules/data, terraform/modules/compute).

#### DynamoDB Table

terraform/modules/data/main.tf:

```
data "aws_caller_identity" "current" {}

#tfsec:ignore:aws-dynamodb-table-customer-key
resource "aws_dynamodb_table" "summarizer" {
  name         = "${var.project_name}-${var.table_name}"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "user_id"
  range_key    = "timestamp"

  attribute {
    name = "user_id"
    type = "S"
  }

  attribute {
    name = "timestamp"
    type = "S"
  }

  # summary_date is written as YYYY-MM-DD by the summarizer Lambda.
  # The weekly report Lambda uses this GSI to fetch all records in a
  # date range across all users (one Query per day, up to 7 for a week).
  attribute {
    name = "summary_date"
    type = "S"
  }

  server_side_encryption {
    enabled = true
  }

  point_in_time_recovery {
    enabled = true
  }

  global_secondary_index {
    name            = "summary-date-index"
    hash_key        = "summary_date"
    range_key       = "timestamp"
    projection_type = "ALL"
  }
}

#tfsec:ignore:aws-s3-enable-bucket-logging
#tfsec:ignore:aws-s3-enable-versioning
resource "aws_s3_bucket" "reports" {
  bucket = "${var.project_name}-reports-${data.aws_caller_identity.current.account_id}"
}

#tfsec:ignore:aws-s3-encryption-customer-key
resource "aws_s3_bucket_server_side_encryption_configuration" "reports" {
  bucket = aws_s3_bucket.reports.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "reports" {
  bucket                  = aws_s3_bucket.reports.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "reports" {
  bucket = aws_s3_bucket.reports.id
  rule {
    id     = "archive-to-glacier"
    status = "Enabled"
    filter {}
    transition {
      days          = 30
      storage_class = "GLACIER"
    }
  }
}

```

- PAY_PER_REQUEST billing: no idle cost, no capacity planning.
- Encryption at rest, point-in-time recovery.
- GSI on summary_date: needed by the weekly report Lambda to query "all summaries in the last 7 days across all users," which the base table's user_id partition key can't answer efficiently alone.

#### Lambda IAM Role

The log group is generated manually (aws_cloudwatch_log_group, retaining logs for 7 days) before the actual function is able to begin executing the Lambda-based function, instead of utilizing the log group which is created by Lambda automatically without an expiration period. The timeout is set for 30 seconds, as Bedrock calls and retries also require the extra time on top of the default value of 3 seconds.

If no Cognrito claims are supplied in the request, the user_id is assigned a hard-coded value, which can be considered useful for local/manual testing and which must be deleted before launching the application in any production mode.

#### Lambda Function

The log group is built explicitly (aws_cloudwatch_log_group, seven days retention) and was needed before the function is called instead of relying on the automatically created log group without the expiration period. The timeout period is set to 30 seconds as Bedrock calls plus retry backoff need room greater than the default 3 seconds.

If the requests do not include any Cognito claims, the user_id is set to a specific test value. This feature is used for local/manual testing and should be removed before deploying to actual production.

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

**How to verify:** response.json shows "statusCode": 200 with summary, timestamp, and summary_date fields. A 502 here means Bedrock model access hasn't been granted yet (see Section 5.4.2).

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException on DynamoDB | dynamodb_table_arn not passed correctly into the compute module | Confirm root main.tf wires module.data.table_arn into module.compute |
| ResourceNotFoundException | Lambda's DYNAMODB_TABLE env var doesn't match the actual table name | Check module.data.table_name output matches what's passed to module.compute |
| terraform apply fails creating the Lambda: "no such file" | var.lambda_zip_path doesn't point to a built deployment package | Zip src/lambda_fn/ before running terraform apply |