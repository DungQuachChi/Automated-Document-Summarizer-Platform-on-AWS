---
title : "Frontend Hosting — S3 & CloudFront"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.5.2. "
---

#### Goal

Host the static frontend on S3, serve it through CloudFront, and propagate the resulting domain to Cognito and API Gateway automatically using Terraform output wiring.

#### Terraform Resources

modules/frontend: 
- There is a private awsS3_bucket with no public access. The traffic is routed through cloudfront with Origin Access Control (OAC) enabled. Hence, the bucket is secured and HTTP is redirected to HTTPS with PriceClass_100.
- The bucket policy allows access to cloudfront.amazonaws.com for s3:GetObject, hence restricting it to this distribution alone through AWS:SourceArn.
- Versioning, CloudFront Access Logging, and WAF Web ACL are not implemented due to costs not justifying the necessary expenses for this project.
#### Cross-Module Wiring

module.frontend.cloudfront_domain_name feeds directly into both other modules:

```hcl
module "auth" { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
module "api"  { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
```

Cognito's callback_urls/logout_urls are derived from the same output as the API Gateway's CORS origin. With one terraform apply creating the bucket/distribution, the resultant domain is picked up and automatically populated into both; there is no need to manually update Cognito or CORS afterwards.


#### Applying and Deploying Files

```bash
terraform apply
terraform output cloudfront_domain_name
```

`frontend/script.js` is deliberately not in the repository. It holds live Cognito and API Gateway values, so committing it would publish one account's credentials to everyone who clones the project. What ships instead is a template. Copy it first:

```bash
cp frontend/script.js.example frontend/script.js
```

Now open the `CONFIG` object at the top of your new `frontend/script.js` and fill in each field from your own stack. Run these from the `terraform/` directory:

| `script.js` field | Where to get it |
|---|---|
| `API_URL` | `terraform output -raw api_invoke_url` |
| `CLIENT_ID` | `terraform output -raw cognito_app_client_id` |
| `API_KEY` | `terraform output -raw api_key_value` |
| `REDIRECT_URI` | `https://$(terraform output -raw cloudfront_domain_name)/` keep the trailing slash, it has to match a Cognito callback URL exactly |
| `COGNITO_DOMAIN` | `<your-domain-prefix>.auth.<your-region>.amazoncognito.com`, built from the `cognito_domain_prefix` you set in terraform.tfvars (there is no dedicated output for it) |

`script.js` is listed in .gitignore, so your filled-in copy stays local and never reaches the repository. Leave it that way.

```bash
aws s3 cp frontend/index.html s3://$(terraform output -raw frontend_bucket_name)/
aws s3 cp frontend/script.js  s3://$(terraform output -raw frontend_bucket_name)/
aws cloudfront create-invalidation \
  --distribution-id $(terraform output -raw cloudfront_distribution_id) \
  --paths "/*"
```

**How to verify:** open the CloudFront domain, frontend loads, sign-in redirects correctly. A direct S3 object URL request returns **AccessDenied**.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDenied loading the CloudFront URL | Bucket policy's SourceArn doesn't yet match the distribution ARN | terraform apply again, policy is generated from the distribution resource and self-corrects |
| Old script.js still served after re-upload | CloudFront edge caching | Re-run the invalidation step after every frontend file update |
| Cognito redirects to a blank/error page after login | terraform apply hasn't run since the frontend module was added | terraform apply no manual Cognito edit needed |

---

### Section Summary

Congratulations on completing the access layer. In this section, you set up Cognito to issue and validate JWTs, configured API Gateway to enforce that token plus an API key and usage plan on every request, and hosted the static frontend on S3 behind CloudFront using an Origin Access Control so the bucket itself stays private. The pieces connect automatically because the CloudFront domain name is passed as a Terraform output into both the Cognito app client's callback URLs and API Gateway's CORS configuration. One **terraform apply** keeps all three in sync, with no manual console cross-referencing required. This demonstrates defense in depth at the edge: authentication, authorization, and rate limiting are all enforced before a request ever reaches application code, and the frontend is reachable only through a CDN, never directly.