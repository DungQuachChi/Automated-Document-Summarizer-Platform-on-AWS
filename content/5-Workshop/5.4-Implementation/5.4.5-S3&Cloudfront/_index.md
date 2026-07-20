---
title : "Frontend Hosting — S3 & CloudFront"
date : 2026-07-14
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

#### Goal

Host the static frontend (frontend/index.html, frontend/script.js) on S3, serve it through CloudFront, and propagate the resulting domain to Cognito and API Gateway automatically via Terraform output wiring — no manual console updates to either service.

#### Terraform Resources

modules/frontend creates:

- aws_s3_bucket — private bucket, no static website hosting, no public access. All four aws_s3_bucket_public_access_block settings are enabled.
- aws_s3_bucket_server_side_encryption_configuration — AES256 at rest.
- aws_cloudfront_origin_access_control — OAC, the current recommended pattern (replaces the deprecated OAI). CloudFront authenticates to S3 with SigV4; the bucket stays fully private.
- aws_cloudfront_distribution — single origin pointed at the bucket via OAC, index.html as the default root object, HTTP redirected to HTTPS, PriceClass_100 (US/Canada/Europe edges only — cheapest tier, appropriate for a low-traffic learning project).
- aws_iam_policy_document + aws_s3_bucket_policy — grants cloudfront.amazonaws.com s3:GetObject, scoped with an AWS:SourceArn condition to this specific distribution's ARN, so no other CloudFront distribution in the account (or anyone else's) can read the bucket.

Three items are deliberately left out and marked with tfsec:ignore comments with a stated reason: bucket versioning/logging, CloudFront access logging, and a WAF Web ACL. Each carries an ongoing storage or per-request cost not justified at this project's traffic level. The TLS policy is also left at the CloudFront default, since a custom minimum TLS version requires an ACM certificate on a purchased domain — out of scope for a *.cloudfront.net deployment.

#### Cross-Module Wiring

This is the part that matters for the CI/CD story: the frontend's domain name isn't a value you copy-paste into other consoles. modules/frontend/outputs.tf exposes it —

```hcl
output "cloudfront_domain_name" {
  value = aws_cloudfront_distribution.frontend.domain_name
}
```

— and the root main.tf passes that output directly into two other modules:

```hcl
module "auth" {
  source = "./modules/auth"
  cloudfront_domain_name = module.frontend.cloudfront_domain_name
  # ...
}

module "api" {
  source = "./modules/api"
  cloudfront_domain_name = module.frontend.cloudfront_domain_name
  # ...
}
```

Inside modules/auth, that value builds the Cognito app client's callback and logout URLs:

```hcl
callback_urls = ["http://localhost:8000", "https://${var.cloudfront_domain_name}"]
logout_urls   = ["http://localhost:8000/", "https://${var.cloudfront_domain_name}/"]
```

Inside modules/api, the same value locks down CORS on both /summarize and /history:

```hcl
"method.response.header.Access-Control-Allow-Origin" = "'https://${var.cloudfront_domain_name}'"
```

localhost:8000 stays alongside the CloudFront domain in Cognito's callback list, so local development keeps working after the real domain goes live. CORS, by contrast, is locked to the CloudFront domain only — the API doesn't need to accept requests from arbitrary local origins in the same way auth redirects do.

Terraform resolves the dependency graph from these references alone — frontend doesn't need to appear earlier in the file for auth and api to wait for it. One terraform apply provisions the bucket and distribution, reads the resulting domain name, and pushes it into the Cognito app client and both API Gateway CORS configs in the same run. There is no separate manual step to "update Cognito callback URLs" or "lock CORS to the real domain" after the fact — that's the difference between wiring an output through the module graph and re-entering the same value by hand in two consoles.

#### Applying

```bash
terraform apply
```

Note the two relevant outputs:

```bash
terraform output cloudfront_domain_name
```

#### Deploying the Frontend Files

The bucket and distribution are infrastructure; the HTML/JS files themselves are application content and aren't Terraform-managed resources here, so they're uploaded directly:

1. Set the real API endpoint in frontend/script.js:
   ```js
   const API_BASE_URL = "https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1";
   ```
2. Upload both files to the bucket root:
   ```bash
   aws s3 cp frontend/index.html s3://$(terraform output -raw frontend_bucket_name)/
   aws s3 cp frontend/script.js  s3://$(terraform output -raw frontend_bucket_name)/
   ```
3. Invalidate the CloudFront cache so the new files serve immediately instead of a stale cached version:
   ```bash
   aws cloudfront create-invalidation \
     --distribution-id $(terraform output -raw cloudfront_distribution_id) \
     --paths "/*"
   ```

**How to verify:** open the CloudFront distribution domain in a browser — the frontend loads and sign-in redirects back correctly. A direct request to the S3 object URL (bypassing CloudFront) returns AccessDenied, confirming the bucket is unreachable except through the OAC-authenticated distribution.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDenied when loading the CloudFront URL | Bucket policy's AWS:SourceArn condition doesn't match the actual distribution ARN — usually from applying the frontend module before it exists yet, or a stale plan | terraform apply again; the policy is generated from aws_cloudfront_distribution.frontend.arn, so it self-corrects once the distribution exists |
| Frontend loads but API calls fail with a CORS error | Request origin doesn't match cloudfront_domain_name at apply time — happens if you're testing from localhost while CORS is locked to the CloudFront domain | Test from the CloudFront domain, or re-check what value cloudfront_domain_name resolved to at the last apply |
| Old script.js still served after re-upload | CloudFront caches objects at edge locations | Re-run the invalidation step after every frontend file update |
| Cognito Hosted UI redirects to a blank/error page after sign-in | terraform apply hasn't run since the frontend module was added, so Cognito's callback URL list doesn't include the CloudFront domain yet | terraform apply — no manual Cognito console edit needed |