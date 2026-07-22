---
title : "Auth & API — Cognito & API Gateway"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.5.1. "
---

#### Goal

Set up Cognito to issue and validate JWTs, and API Gateway to enforce that token plus an API key and usage plan on every request — both declared in Terraform, with no post-deploy console configuration needed.

#### Cognito (`modules/auth`)

- aws_cognito_user_pool: email as the sign-in identifier, strong password policy, advanced_security_mode = "ENFORCED" (Cognito's risk-based adaptive auth — compromised-credential and unusual-activity detection, zero extra app code). Self-registration is on by default.
- aws_cognito_user_pool_client: public client. explicit_auth_flows declares both ALLOW_USER_PASSWORD_AUTH and ALLOW_REFRESH_TOKEN_AUTH from the start. The Hosted UI path and the direct load-test auth path work off the same client with no later toggle.
- aws_cognito_user_pool_domain: Hosted UI domain prefix, set via variable.

#### API Gateway (modules/api)

- aws_api_gateway_rest_api (REST, not HTTP — needed for usage plans/API keys) + aws_api_gateway_authorizer (COGNITO_USER_POOLS, wired to the user pool ARN above).
- /summarize (POST) and /history (GET), both AWS_PROXY to the same Lambda (Section 5.4.1). Both methods declare authorization = "COGNITO_USER_POOLS" and api_key_required = true directly in code.
- CORS is an explicit OPTIONS method + MOCK integration, not a checkbox — Access-Control-Allow-Origin locked to the CloudFront domain (Section 5.5.2), not *.
- aws_api_gateway_deployment with a triggers hash over every resource/method/integration/authorizer ID — Terraform redeploys automatically on any route change.
- Account-level CloudWatch logging role and stage access logging are both provisioned here.
- aws_api_gateway_usage_plan — rate_limit = 2/s, burst_limit = 20, quota = 5000/month, with an aws_api_gateway_api_key linked to it.

#### Applying

```bash
terraform apply
terraform output user_pool_id
terraform output client_id
terraform output api_invoke_url
terraform output -raw api_key_value
```

#### How JWT Auth Works

Browser → Hosted UI login → Cognito verifies (risk-evaluated) → redirect with auth code → app exchanges code for JWT at /oauth2/token → JWT sent as Authorization header on every API call → API Gateway's Cognito authorizer validates before invoking Lambda. The JWT's sub claim becomes user_id in DynamoDB, scoping each user to their own history.

For scripted/load-test access, InitiateAuth/USER_PASSWORD_AUTH is enabled on the same client via explicit_auth_flows — no separate setup needed.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| invalid_request on Hosted UI login | Redirect URL doesn't match callback_urls | Confirm exact match — no typos/extra paths |
| grant_type=password returns 400 | Hosted UI's token endpoint only supports authorization_code/refresh_token | Use InitiateAuth/USER_PASSWORD_AUTH instead |
| {"message": "Unauthorized"} / "Forbidden" from the API | Missing/expired JWT, or missing/invalid x-api-key | Re-authenticate via Hosted UI; confirm x-api-key matches terraform output -raw api_key_value |
| Route/CORS changes don't seem to apply | Confusing this with manual console redeploys | terraform plan — if aws_api_gateway_deployment shows pending replacement, terraform apply |