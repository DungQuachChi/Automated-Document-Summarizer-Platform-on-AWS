---
title : "Auth Layer — Cognito"
date : 2026-07-14
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

#### Goal

Set up Amazon Cognito to handle user sign-up, login, and JWT token issuance, so the API never has to handle passwords directly — and support two distinct ways of authenticating: the Hosted UI for real users, and a direct password-based flow for automated load testing.

#### Terraform Resources

modules/auth creates:

- aws_cognito_user_pool — username_attributes = ["email"] (sign in by email, no separate username), auto_verified_attributes = ["email"] (verification codes sent automatically on sign-up). A strong password policy is set explicitly: minimum length 8, requiring lowercase, uppercase, numbers, and symbols. Account recovery goes through verified_email only. advanced_security_mode = "ENFORCED" turns on Cognito's risk-based adaptive authentication — it evaluates sign-in attempts for compromised-credential and unusual-activity signals and can require additional challenges, without any extra code on the app side.
- aws_cognito_user_pool_client — a public client (generate_secret = false, appropriate since the frontend is static JS with no secure place to store a client secret). explicit_auth_flows includes both ALLOW_USER_PASSWORD_AUTH and ALLOW_REFRESH_TOKEN_AUTH — both are declared in code from the start, not added as a separate step later. allowed_oauth_flows = ["code"] restricts the Hosted UI to the authorization code grant (no implicit grant). Scopes are email, openid, profile. callback_urls and logout_urls include http://localhost:8000 for local development, plus the CloudFront domain wired in automatically from modules.frontend.cloudfront_domain_name (Section 5.4.5) — that domain isn't typed into the console by hand.
- aws_cognito_user_pool_domain — the Hosted UI domain prefix (pathbridger), set via a variable rather than hardcoded, so a fresh deployment under a different account or prefix doesn't require editing this module.

Self-registration isn't a checkbox to enable — a Cognito user pool allows public sign-up by default unless admin_create_user_config.allow_admin_create_user_only is explicitly set to true, which this module doesn't do. So "self-registration enabled" here just means the module doesn't turn it off.

Both authentication paths this app needs — Hosted UI for the frontend, InitiateAuth/USER_PASSWORD_AUTH for Locust (Section 5.5) — come from the same explicit_auth_flows list above. There's no separate console toggle to flip after the fact for load testing to work; it's already permitted by the pool's Terraform configuration.

#### Applying

```bash
terraform apply
```

Then pull the values everything downstream needs:

```bash
terraform output user_pool_id
terraform output client_id
terraform output hosted_ui_domain
```

`hosted_ui_domain` is computed in `outputs.tf` as `${domain}.auth.${aws_region}.amazoncognito.com` — not a value you need to piece together manually from two different console screens.

#### How JWT Authentication Works

1. Browser redirects to the Hosted UI at `pathbridger.auth.ap-southeast-1.amazoncognito.com`.
2. User signs in with email and password.
3. Cognito verifies credentials (and, since `advanced_security_mode` is `ENFORCED`, evaluates the sign-in for risk signals).
4. Cognito redirects back with an authorization code.
5. The app exchanges the code for a JWT (`id_token`) at the `/oauth2/token` endpoint.
6. The JWT is sent as the `Authorization` header on every API call.
7. API Gateway's Cognito authorizer validates the token before invoking Lambda.

The JWT's `sub` claim becomes the `user_id` used in DynamoDB (Section 4.1), ensuring each user only sees their own history.

For scripted/load-test access (Section 5.5), the same user pool supports a second path that skips the Hosted UI entirely: `InitiateAuth` with `USER_PASSWORD_AUTH`, called directly via boto3. This is a distinct, unauthenticated Cognito Identity Provider API call — not the Hosted UI's `/oauth2/token` endpoint, which only accepts `authorization_code`/`refresh_token` grants and rejects `grant_type=password`. Both paths are valid because `explicit_auth_flows` declares both flows on the same app client.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `invalid_request` on the Hosted UI login page | The URL being used to reach the Hosted UI doesn't match an entry in `callback_urls` | Confirm the app is redirecting to exactly `http://localhost:8000` or the CloudFront domain — both are in the Terraform-managed list, but a typo or extra path segment won't match |
| `grant_type=password` returns 400 from `/oauth2/token` | The Hosted UI's OAuth token endpoint only supports `authorization_code`/`refresh_token` | Use `InitiateAuth` with `USER_PASSWORD_AUTH` instead — a separate, unauthenticated Cognito Identity Provider API, not the Hosted UI endpoint |
| `NotAuthorizedException` from `InitiateAuth` | `ALLOW_USER_PASSWORD_AUTH` isn't actually present on the deployed app client — usually from applying an older version of this module before the flow was added | `terraform plan` should show no drift on `explicit_auth_flows`; re-apply if it does |
| Sign-in redirects to a blank/error page after Hosted UI login | The domain the request came from isn't yet in `callback_urls` — happens if `terraform apply` hasn't run since the frontend module (and its CloudFront domain output) was added | `terraform apply` — no manual Cognito console edit needed, the callback URL list is derived from `module.frontend.cloudfront_domain_name` |
| Domain prefix `pathbridger` fails to create | Prefix is already taken globally (Cognito domain prefixes are unique across all AWS accounts) or was recently released and hasn't fully freed up yet (Section 5.6) | Choose a different `cognito_domain_prefix` value, or wait a few minutes if this is a re-deploy after a recent `terraform destroy` |    