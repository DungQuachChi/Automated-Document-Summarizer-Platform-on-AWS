---
title : "API Layer — API Gateway"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

#### Goal

Set up API Gateway as the secure entry point for the platform: validate JWT tokens via a Cognito authorizer, enforce an API key and usage plan, and deploy the API to a live stage — all declared in Terraform, with no post-deploy console configuration needed.

#### Terraform Resources

`modules/api` creates:

- `aws_api_gateway_rest_api` — `doc-summarizer-api`, a REST API (not HTTP API — REST API is what supports usage plans and API keys, which this project needs).
- `aws_api_gateway_authorizer` — type `COGNITO_USER_POOLS`, wired to `var.cognito_user_pool_arn` (passed in from `modules.auth`, Section 5.4.2). One authorizer, referenced by both routes below.
- `aws_api_gateway_resource` + `aws_api_gateway_method` + `aws_api_gateway_integration` for `/summarize` (POST) and `/history` (GET), both `AWS_PROXY` integration type pointed at the same Lambda (`doc-summarizer-fn`). Both methods declare `authorization = "COGNITO_USER_POOLS"` and `api_key_required = true` directly in code — there's no separate console step to turn either of these on afterward.
- `aws_api_gateway_method` (OPTIONS, `authorization = "NONE"`) + a `MOCK` integration + explicit method/integration responses, on both resources — this is what "check CORS" means under the hood on a REST API. `Access-Control-Allow-Origin` is set to `'https://${var.cloudfront_domain_name}'`, not `*` (the same CloudFront-domain wiring from Section 5.4.5), and `Access-Control-Allow-Headers`/`-Methods` are set explicitly rather than left to defaults.
- `aws_lambda_permission` — grants `apigateway.amazonaws.com` `lambda:InvokeFunction`, scoped via `source_arn` to this specific API's execution ARN (`.../*/*/*`, meaning any stage/method/resource on this API, but not other APIs or Lambdas in the account).
- `aws_api_gateway_deployment` — its `triggers` block hashes the IDs of every resource, method, integration, and the authorizer. Terraform recomputes that hash on every `plan`; if any of those change, the deployment resource is recreated automatically. This is the Terraform equivalent of clicking **Deploy API** — it happens as part of `terraform apply`, not as a manual follow-up step after editing routes in a console.
- `aws_iam_role` (assumable by `apigateway.amazonaws.com`) + the AWS-managed `AmazonAPIGatewayPushToCloudWatchLogs` policy + `aws_api_gateway_account` — this is the account-level CloudWatch logging role API Gateway needs before *any* stage's logs can be delivered. It's created here, not left as a manual console gap.
- `aws_cloudwatch_log_group` (`/aws/api_gateway/doc-summarizer-api`, 7-day retention) + `access_log_settings` on the stage — access logging (who called what, when, response status) is on from the first deploy, structured as JSON with request ID, source IP, method, path, status, and response length.
- `aws_api_gateway_stage` — stage name `v1`.
- `aws_api_gateway_usage_plan` — `throttle_settings { rate_limit = 2, burst_limit = 20 }` (steady-state 2 req/s, allowing short bursts up to 20), `quota_settings { limit = 5000, period = "MONTH" }`.
- `aws_api_gateway_api_key` + `aws_api_gateway_usage_plan_key` — the key and its association to the usage plan, both created and linked in the same apply.

#### Applying

```bash
terraform apply
```

Then retrieve what you need to call the API:

```bash
terraform output api_invoke_url
terraform output -raw api_key_value   # marked sensitive; -raw prints it without quotes
```

`api_invoke_url` already includes the stage (`https://.../v1`) — no separate lookup for "which stage is this" is needed.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `{"message": "Missing Authentication Token"}` | Request sent to a path/method that doesn't exist on the deployed stage | Confirm the URL includes the stage name (`/v1/summarize`, not just `/summarize`) |
| `{"message": "Unauthorized"}` | No `Authorization` header, or an expired/invalid JWT | Sign in again via the Hosted UI to get a fresh token (Section 5.4.2) |
| `{"message": "Forbidden"}` | Missing or invalid `x-api-key` header | Confirm the request includes `x-api-key: <api_key_value>` — pull the current value with `terraform output -raw api_key_value` rather than a value copied from an earlier session |
| Changes to a route or authorizer don't seem to take effect | Confusing this with the console workflow — in Terraform, `terraform plan` shows the deployment resource will be replaced whenever the trigger hash changes; if `apply` wasn't actually re-run after the code change, nothing was deployed | Run `terraform plan` and check whether `aws_api_gateway_deployment.deployment` shows as needing replacement; if so, `terraform apply` |
| CORS error in the browser console despite the OPTIONS method existing | Request origin doesn't match `cloudfront_domain_name` at the time of the last apply (e.g. testing from `localhost` while CORS is locked to the CloudFront domain) | Test from the origin CORS is actually locked to, or confirm what `cloudfront_domain_name` resolved to at the last apply |
| 429 quota/throttle errors under normal single-user testing | `rate_limit = 2` req/s is intentionally tight for a project at this scale — a quick sequence of manual curl calls or a browser retry loop can hit it | Expected behavior, not a bug — space out requests, or note this in the load test discussion (Section 5.5) rather than treating it as an error to fix |