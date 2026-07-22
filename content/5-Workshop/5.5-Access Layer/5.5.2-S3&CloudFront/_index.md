---
title : "Frontend Hosting — S3 & CloudFront"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.5.2. "
---

#### Goal

Host the static frontend on S3, serve it through CloudFront, and propagate the resulting domain to Cognito and API Gateway (Section 5.5.1) automatically via Terraform output wiring.

#### Terraform Resources

modules/frontend:

- Private aws_s3_bucket (all public access blocked) + aws_cloudfront_distribution with Origin Access Control — CloudFront authenticates to S3 with SigV4, so the bucket stays fully private. HTTP redirected to HTTPS, PriceClass_100.
- Bucket policy grants cloudfront.amazonaws.com s3:GetObject, scoped via AWS:SourceArn to this distribution only.
- Versioning, CloudFront access logging, and a WAF Web ACL are deliberately left out — cost not justified at this project's scale.

#### Cross-Module Wiring

module.frontend.cloudfront_domain_name feeds directly into both other modules:

```hcl
module "auth" { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
module "api"  { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
```

Cognito's callback_urls/logout_urls and API Gateway's CORS origin both derive from this one output. One terraform apply provisions the bucket/distribution, reads the resulting domain, and pushes it into both — there's no separate step to update Cognito or CORS by hand afterward.

#### Applying and Deploying Files

```bash
terraform apply
terraform output cloudfront_domain_name
```

```bash
# set API_BASE_URL in frontend/script.js to the api_invoke_url output first
aws s3 cp frontend/index.html s3://$(terraform output -raw frontend_bucket_name)/
aws s3 cp frontend/script.js  s3://$(terraform output -raw frontend_bucket_name)/
aws cloudfront create-invalidation \
  --distribution-id $(terraform output -raw cloudfront_distribution_id) \
  --paths "/*"
```

**How to verify:** open the CloudFront domain — frontend loads, sign-in redirects correctly. A direct S3 object URL request returns **AccessDenied**.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDenied loading the CloudFront URL | Bucket policy's SourceArn doesn't yet match the distribution ARN | terraform apply again — policy is generated from the distribution resource and self-corrects |
| Old script.js still served after re-upload | CloudFront edge caching | Re-run the invalidation step after every frontend file update |
| Cognito redirects to a blank/error page after login | terraform apply hasn't run since the frontend module was added | terraform apply — no manual Cognito edit needed |

---

### Section Summary

Congratulations on completing the access layer. In this section, you set up Cognito to issue and validate JWTs, configured API Gateway to enforce that token plus an API key and usage plan on every request, and hosted the static frontend on S3 behind CloudFront using an Origin Access Control so the bucket itself stays private. The pieces connect automatically because the CloudFront domain name is passed as a Terraform output into both the Cognito app client's callback URLs and API Gateway's CORS configuration — one **terraform apply** keeps all three in sync, with no manual console cross-referencing required. This demonstrates defense in depth at the edge: authentication, authorization, and rate limiting are all enforced before a request ever reaches application code, and the frontend is reachable only through a CDN, never directly.