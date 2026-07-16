---
title : "Monitoring & Security Layer"
date : 2026-07-14
weight : 8
chapter : false
pre : " <b> 5.4.8. </b> "
---

#### Goal

Set up visibility into the running system (dashboard, alarms, custom metrics) and a security/compliance baseline (audit logging, configuration rules) — both defined as Terraform resources in terraform/modules/monitoring and terraform/modules/security, deployed the same way as every other part of this project.

#### Custom Metrics from the Lambda Code

API Gateway, Lambda, and DynamoDB emit standard AWS metrics automatically. Bedrock does not, through this integration path, so src/lambda_fn/lambda_function.py publishes custom metrics to a Custom/Bedrock namespace around every model call:

```python
cloudwatch_client.put_metric_data(
    Namespace='Custom/Bedrock',
    MetricData=[
        {'MetricName': 'Latency', 'Value': latency_ms, 'Unit': 'Milliseconds'},
        {'MetricName': 'SuccessCount', 'Value': 1, 'Unit': 'Count'}
    ]
)
```

On failure, a BedrockErrors metric is published instead, with an ErrorType dimension so a quota error can be distinguished from a timeout without reading logs. The Lambda's IAM role grants cloudwatch:PutMetricData explicitly, alongside its log-group and Bedrock permissions — see Section 5.3.

#### Step 1 — SNS Topic and Alarms (Terraform)

terraform/modules/monitoring/main.tf defines one SNS topic with an email subscription, and two alarms:

```hcl
resource "aws_sns_topic" "alarms" {
  name = "${var.project_name}-alarms"
}

resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.alarms.arn
  protocol  = "email"
  endpoint  = var.alarm_email
}

# API Gateway 5XX error rate > 5% over 5 consecutive minutes
resource "aws_cloudwatch_metric_alarm" "api_5xx" {
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 5
  threshold           = 5

  metric_query {
    id          = "error_rate"
    expression  = "errors / requests * 100"
    return_data = true
  }
  metric_query {
    id = "errors"
    metric {
      namespace   = "AWS/ApiGateway"
      metric_name = "5XXError"
      dimensions  = { ApiId = var.api_id }
      stat        = "Sum"
    }
  }
  metric_query {
    id = "requests"
    metric {
      namespace   = "AWS/ApiGateway"
      metric_name = "Count"
      dimensions  = { ApiId = var.api_id }
      stat        = "Sum"
    }
  }
  alarm_actions = [aws_sns_topic.alarms.arn]
}

# Any Bedrock error at all
resource "aws_cloudwatch_metric_alarm" "bedrock_errors" {
  namespace           = "Custom/Bedrock"
  metric_name         = "BedrockErrors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  threshold           = 0
  treat_missing_data  = "notBreaching"
  alarm_actions       = [aws_sns_topic.alarms.arn]
}
```

The API alarm uses a metric-math expression (errors / requests * 100) rather than a raw count, so it tracks *rate*, not volume — it won't fire just because traffic is high. The Bedrock alarm fires on a single error (threshold = 0), since any Bedrock failure is worth surfacing immediately at this traffic scale. treat_missing_data = "notBreaching" means periods with zero calls don't get treated as failures.

After terraform apply, check your email and confirm the SNS subscription — alarms will not deliver until this is confirmed.

**How to verify:** Both alarms show state **OK** (green) in the CloudWatch **Alarms** list once there's been at least one evaluation period with no errors.

#### Step 2 — Deliberately Trigger an Alarm

Confirm the alert path actually works end to end rather than assuming it does.

```bash
aws cloudwatch put-metric-data \
  --namespace Custom/Bedrock \
  --metric-name BedrockErrors \
  --value 1
```

Wait for the alarm's evaluation period to elapse, then confirm the alarm state changes to **In alarm** and the alert email arrives.

**How to verify:** An email from AWS Notifications arrives referencing the triggered alarm name.

#### Step 3 — The Dashboard (Terraform)

aws_cloudwatch_dashboard.main is one Terraform resource, built with jsonencode, with six widgets — Lambda invocations/errors, Lambda duration (p50/p95/p99), Bedrock latency (p50/p95/p99), Bedrock success vs. error count, API Gateway requests/4XX/5XX, and DynamoDB read/write/throttles. All dimensions (function name, API ID, table name) are passed in as module variables from the root main.tf, so the dashboard is wired to the live resources automatically rather than hardcoded:

```hcl
module "monitoring" {
  source                = "./modules/monitoring"
  lambda_function_name  = module.compute.lambda_function_name
  api_id                = module.api.api_id
  dynamodb_table_name   = module.data.table_name
  alarm_email            = var.alarm_email
}
```

**How to verify:** Generate a few real requests against /summarize and /history, then reload the dashboard — the graphs show real, non-zero traffic within a few minutes.

#### Step 4 — CloudTrail (Terraform)

terraform/modules/security/main.tf provisions a multi-region CloudTrail with log file validation, writing to a dedicated, encrypted, public-access-blocked S3 bucket, and also streaming to a CloudWatch Logs group:

```hcl
resource "aws_cloudtrail" "main" {
  s3_bucket_name                = aws_s3_bucket.security_logs.id
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_log_file_validation    = true
  cloud_watch_logs_group_arn    = "${aws_cloudwatch_log_group.cloudtrail.arn}:*"
}
```

Only management events are captured, not data events — the first copy of management events is included free per account regardless of region count, while data events are billed per event. CIS 3.1 requires multi-region management-event coverage, which this satisfies without extra cost.

**How to verify:** The trail shows **Logging: On** in the CloudTrail console. After a few minutes, the security-logs S3 bucket contains log files under a cloudtrail/AWSLogs/YOUR_ACCOUNT_ID/ prefix.

#### Step 5 — AWS Config and the CIS Conformance Pack (Terraform)

```hcl
resource "aws_config_configuration_recorder" "main" {
  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

resource "aws_config_conformance_pack" "cis_v1_4_level1" {
  template_body = file("${path.module}/templates/Operational-Best-Practices-for-CIS-AWS-v1.4-Level1.yaml")
}
```

The recorder tracks all supported resource types, including global ones like IAM — not a hand-picked subset. The conformance pack is AWS's own published CIS v1.4 Level 1 template (44 Config rules), deployed with default parameters, so the account is checked against the full benchmark.

**How to verify:** The **Conformance packs** page in the AWS Config console shows compliance status once evaluation completes (can take 15–20 minutes on first run). This project's current status is **60% compliant**, with two accepted gaps:

- CodeBuild's IAM role uses AdministratorAccess rather than a scoped least-privilege policy (see Section 5.4.7)
- MFA is not enforced account-wide via IAM policy (it is enabled on the individual admin user from Prerequisites, but not conditionally required for all users)

These are documented trade-offs for a learning-budget project, deliberately left visible in the compliance report rather than excluded from the conformance pack's scope — the point of running Config here is an accurate compliance picture, not a clean scorecard. Both are revisited in Section 6.2 (Simple Security Hardening).

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Alarm never leaves **Insufficient Data** state | No data points have been published to the metric yet | Generate real traffic, or manually publish a test metric (Step 2) |
| SNS email never arrives after an alarm fires | Subscription was never confirmed | Check spam/junk folder for the original confirmation email; re-subscribe if it expired |
| CloudTrail bucket shows no log files after 15+ minutes | Bucket policy is missing required CloudTrail permissions | Confirm **Logging: On** in the trail overview; verify the bucket policy statement for AWSCloudTrailWrite applied correctly |
| Conformance pack stuck at **Evaluating** for a long time | Normal for the first run — depends on number of resources in the account | Wait; check back after 20–30 minutes rather than re-deploying |
| Conformance pack compliance percentage lower than expected | Some CIS rules apply to resources or settings outside this project's scope (e.g. default VPC rules) | Review individual rule results before assuming a gap is project-related |