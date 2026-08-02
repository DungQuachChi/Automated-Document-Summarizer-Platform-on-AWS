---
title : "Auth & API — Cognito & API Gateway"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.5.1. "
---

#### Goal

Set up Cognito to issue and validate JWTs, and API Gateway to enforce that token plus an API key and usage plan on every request. Both declared in Terraform, with no post-deploy console configuration needed.

#### Cognito (modules/auth)

- In the case of aws_cognito_user_pool, signup identifies users via email with the implementation of a strong password policy and advanced_security_mode set to "ENFORCED" thereby introducing Cognito's risk-based adaptive authentication in the form of compromised-credential-activity detection with no incremental coding requirements for the app itself. Moreover, self-registration is applied by default.
- As for aws_cognito_user_pool_client, it is a public client which allows explicit_auth_flows with ALLOW_USER_PASSWORD_AUTH and ALLOW_REFRESH_TOKEN_AUTH from the beginning. The hosted UI path (as well as the direct load-testing authentication path) uses the same client and does not require subsequent switching of the client itself.
- For aws_cognito_user_pool_domain, the hosted UI domain prefix has to be configured using variables.

#### API Gateway (modules/api)

- AWS API Gateway Rest API (REST, not HTTP) is required to make API keys and usage plans work. AWS API Gateway Authorizer (COGNITO_USER_POOLS) gets wired to the same user pool ARN above.
- Endpoints /summarize (POST) and /history (GET) are AWS_PROXY to the same Lambda (Section 5.4.1). The authorization in both methods is set to "COGNITO_USER_POOLS" and the api_key_required parameter in the methods is set to true in the code.
- Implementing CORS requires an explicit OPTIONS method plus a MOULK integration. The correct header Access-Control-Allow-Origin must be provided, which is locked to the CloudFront domain. 
- Using the aws_api_gateway_deployment resource, we create a trigger for each resource/method/integration/authorizer IDs. 
- Both account-level CloudWatch logging role and stage logging role must be created here.
- aws_api_gateway_usage_plan creates an environment with a rate limit, available burst, and monthly quota.

#### Applying

```bash
terraform apply
terraform output cognito_user_pool_id
terraform output cognito_app_client_id
terraform output api_invoke_url
terraform output -raw api_key_value
```

#### How JWT Auth Works

Browser → Hosted UI authentication → Cognito checks (evaluates risk) → sends back authorization code → the app receives code and exchanges it for JWT via /oauth2/token → for every API call, JWT is sent in HTTP Authorization header → API Gateway's Cognito authorizer verifies the JWT before calling the Lambda function. The JWT sub claim translates into user_id in DynamoDB; therefore, each user has their own history.

When scripted/load-test type of access is needed, InitiateAuth/USER_PASSWORD_AUTH is enabled in the same client because of explicit_auth_flows – no stand-alone setup is required for this.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| invalid_request on Hosted UI login | Redirect URL doesn't match callback_urls | Confirm exact match no typos/extra paths |
| grant_type=password returns 400 | Hosted UI's token endpoint only supports authorization_code/refresh_token | Use InitiateAuth/USER_PASSWORD_AUTH instead |
| {"message": "Unauthorized"} / "Forbidden" from the API | Missing/expired JWT, or missing/invalid x-api-key | Re-authenticate via Hosted UI; confirm x-api-key matches terraform output -raw api_key_value |
| Route/CORS changes don't seem to apply | Confusing this with manual console redeploys | terraform plan if aws_api_gateway_deployment shows pending replacement, terraform apply |