---
title : "Clean Up"
date : 2026-07-14
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

#### Teardown Order 

Terraform generally handles dependency order automatically, but a few AWS resources block deletion until a manual precondition is met — regardless of what Terraform tries to do. Two matter most here:

- **S3 buckets must be empty before they can be destroyed**, unless the bucket resource sets force_destroy = true. A bucket containing objects will fail to delete with BucketNotEmpty otherwise.
- **CloudFront distributions must be disabled before the underlying origin (the S3 bucket) can be safely removed**, and a distribution itself must finish disabling (a distribution-level operation that takes several minutes) before it can be deleted.

Getting the order wrong doesn't corrupt anything — it just means terraform destroy will fail partway through, and needs to be re-run after resolving the blocker.

#### Which Buckets Actually Need Manual Emptying

Only one of the four S3 buckets in this project sets force_destroy = true: the CI/CD artifacts bucket (modules/pipeline), since build artifacts are disposable by design. The other three do not, and will fail terraform destroy if anything is inside them:

| Bucket | Module | force_destroy | Manual empty required? |
|---|---|---|---|
| Frontend static files | modules/frontend | not set | Yes |
| Weekly reports | modules/data | not set | Yes |
| CloudTrail + Config logs | modules/security | not set | Yes |
| Pipeline artifacts | modules/pipeline | true | No — destroys cleanly even with objects inside |

None of the four buckets have S3 versioning enabled (deliberately, for a project at this scale — see the tfsec:ignore comments on each bucket resource), so there are no old object versions or delete markers to worry about when emptying them. The remote state bucket (doc-summarizer-tfstate-...) is a separate, manually-bootstrapped bucket outside this Terraform configuration entirely — it isn't part of terraform destroy and shouldn't be torn down as part of this workshop's cleanup.

#### Step 1 — Empty the Three Non-Force-Destroy Buckets

Do this before running terraform destroy, for the frontend bucket (Section 4.5), the reports bucket (Section 4.6), and the security logs bucket (Section 4.8):

1. **S3** console → select the bucket → **Empty**.
2. Type permanently delete to confirm.
3. Repeat for all three buckets in the table above. The pipeline artifacts bucket can be skipped — force_destroy handles it automatically.

#### Step 2 — Disable the CloudFront Distribution

1. **CloudFront** console → select the distribution (Section 4.5).
2. Click **Disable**.
3. Wait for **Status** to change from **Deploying** to **Disabled** — this can take 5–15 minutes.

#### Step 3 — Run Terraform Destroy

With the three buckets empty and CloudFront disabled, destroy the infrastructure:

```bash
cd terraform
AWS_PROFILE=phatnguyen terraform plan -destroy
```

Review the plan — confirm every resource listed is expected to be destroyed and nothing looks out of place (e.g. a resource from an unrelated project sharing the same state file).

```bash
AWS_PROFILE=phatnguyen terraform destroy
```

Type yes when prompted.

If this is run through the CI/CD pipeline (Section 4.7) rather than locally, the same manual approval gate applies — review the destroy plan before approving it, exactly as you would review an apply plan.

#### Step 4 — Manual Leftovers Terraform Won't Catch

The summarizer and report Lambda log groups are explicitly defined as aws_cloudwatch_log_group resources in modules/compute and modules/scheduling, so terraform destroy removes /aws/lambda/doc-summarizer-fn and /aws/lambda/doc-summarizer-report along with everything else — no manual step needed for those two. What genuinely survives destroy:

- **CodeBuild log groups** — the aws_codebuild_project resources in modules/pipeline don't have a matching aws_cloudwatch_log_group resource, so CodeBuild creates its log group implicitly on first run and Terraform never tracks or destroys it. Check **CloudWatch → Log groups** and manually delete any leftover /aws/codebuild/doc-summarizer-* groups.
- **Cognito domain prefix** — the pathbridger domain prefix (Section 4.2) is released when aws_cognito_user_pool_domain is destroyed, but can take a few minutes to become available again. If re-running this workshop from scratch, confirm the prefix is actually free before attempting to recreate it.
- **CloudTrail multi-region trail state** — CloudTrail can take a short delay to fully stop logging after its Terraform resource is destroyed; a trailing event or two after teardown is normal.

#### Step 5 — Verify $0 Forward Run-Rate

1. Open **Billing and Cost Management** → **Cost Explorer**.
2. Filter to the last 24–48 hours, grouped by service.
3. Confirm no service shows an active, ongoing cost — a small trailing charge from the last few hours before teardown is expected, but nothing should show new activity after the destroy completed.
4. Cross-check against the resource list:

```bash
aws lambda list-functions --profile phatnguyen --query 'Functions[?starts_with(FunctionName, `doc-summarizer`)]'
aws dynamodb list-tables --profile phatnguyen --query 'TableNames[?starts_with(@, `SummarizerTable`)]'
aws s3 ls --profile phatnguyen | grep doc-summarizer
aws cognito-idp list-user-pools --max-results 20 --profile phatnguyen --query 'UserPools[?starts_with(Name, `doc-summarizer`)]'
```

All of these should return empty results (`[]` or no matching lines).

#### Final Verification Checklist

- [ ] Frontend, reports, and security-logs buckets emptied (pipeline artifacts bucket doesn't need it)
- [ ] CloudFront distribution disabled and destroyed
- [ ] terraform destroy completed without errors
- [ ] No leftover CloudWatch log groups under /aws/codebuild/doc-summarizer-* (Lambda log groups are destroyed automatically)
- [ ] Cognito domain prefix released
- [ ] Cost Explorer shows no new activity after teardown
- [ ] CLI list commands (Step 5) return empty for Lambda, DynamoDB, S3, and Cognito

#### Common Destroy Failures and Fixes

| Error | Cause | Fix |
|---|---|---|
| BucketNotEmpty | One of the three non-force-destroy buckets (frontend, reports, security logs) still contains objects | Repeat Step 1 for the specific bucket named in the error |
| DependencyViolation on the CloudFront distribution's origin | Distribution wasn't fully disabled before destroy ran | Repeat Step 2, wait for status **Disabled** (not just clicking Disable) before retrying terraform destroy |
| Cognito user pool domain fails to delete | Domain still attached to the app client's Hosted UI configuration | Remove the domain association from the app client first, or destroy in this order: app client → domain → user pool |
| terraform destroy partially completes, then fails | One resource blocked deletion (any of the above), leaving dependent resources undestroyed | Fix the specific blocker, then re-run terraform destroy — it's safe to re-run, it only acts on resources still present in state |