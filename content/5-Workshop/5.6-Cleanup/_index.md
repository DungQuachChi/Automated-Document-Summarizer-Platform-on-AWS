---
title : "Clean Up"
date : 2026-07-14
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Why Teardown Order Matters

Terraform generally handles dependency order automatically, but a few AWS resources block deletion until a manual precondition is met — regardless of what Terraform tries to do. Two matter most here:

- **S3 buckets must be empty before they can be destroyed.** A bucket containing objects (including old versions, if versioning is enabled) will fail to delete with `BucketNotEmpty`, even via Terraform.
- **CloudFront distributions must be disabled before the underlying origin (the S3 bucket) can be safely removed**, and a distribution itself must finish disabling (a distribution-level operation that takes several minutes) before it can be deleted.

Getting the order wrong doesn't corrupt anything — it just means `terraform destroy` will fail partway through, and needs to be re-run after resolving the blocker.

#### Step 1 — Empty S3 Buckets

Do this before running `terraform destroy`, for both the frontend bucket (Section 4.5) and the reports bucket (Section 4.6):

1. **S3** console → select the bucket → **Empty**.
2. Type `permanently delete` to confirm.
3. If **Bucket Versioning** was enabled (Section 4.7's state bucket, for example), emptying via the console also removes all previous object versions — confirm this is intended, since old report files or state history are not recoverable afterward.

#### Step 2 — Disable the CloudFront Distribution

1. **CloudFront** console → select the distribution (Section 4.5).
2. Click **Disable**.
3. Wait for **Status** to change from **Deploying** to **Disabled** — this can take 5–15 minutes.

#### Step 3 — Run Terraform Destroy

With buckets empty and CloudFront disabled, destroy the infrastructure:

```bash
cd terraform
AWS_PROFILE=phatnguyen terraform plan -destroy
```

Review the plan — confirm every resource listed is expected to be destroyed and nothing looks out of place (e.g. a resource from an unrelated project sharing the same state file).

```bash
AWS_PROFILE=phatnguyen terraform destroy
```

Type `yes` when prompted.

If this is run through the CI/CD pipeline (Section 4.7) rather than locally, the same manual approval gate applies — review the destroy plan before approving it, exactly as you would review an apply plan.

#### Step 4 — Manual Leftovers Terraform Won't Catch

A few things commonly survive `terraform destroy` because they aren't tracked as Terraform-managed resources, or because deleting them isn't part of the standard resource lifecycle:

- **CloudWatch Log groups** — Lambda automatically creates a log group on first invocation (`/aws/lambda/doc-summarizer-fn`, etc.), but Terraform doesn't always manage or destroy these unless explicitly defined as `aws_cloudwatch_log_group` resources. Check **CloudWatch → Log groups** and manually delete any leftover `/aws/lambda/doc-summarizer-*` or `/aws/codebuild/doc-summarizer-*` groups.
- **S3 object versions** — even after "emptying" a versioned bucket through the console (Step 1), check **Show versions** in the bucket's Objects tab to confirm no delete markers or old versions remain before the bucket itself is destroyed.
- **Cognito domain prefix** — the `pathbridger` domain prefix (Section 4.2) is released when the Cognito domain resource is destroyed, but can take a few minutes to become available again. If re-running this workshop from scratch, confirm the prefix is actually free before attempting to recreate it.
- **CloudTrail S3 bucket** — if CloudTrail (Section 4.8) wrote to a bucket outside the main Terraform-managed set, it needs the same "empty before delete" treatment as Step 1.

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

- [ ] All S3 buckets emptied and destroyed
- [ ] CloudFront distribution disabled and destroyed
- [ ] `terraform destroy` completed without errors
- [ ] No leftover CloudWatch log groups under `/aws/lambda/doc-summarizer-*` or `/aws/codebuild/doc-summarizer-*`
- [ ] Cognito domain prefix released
- [ ] Cost Explorer shows no new activity after teardown
- [ ] CLI list commands (Step 5) return empty for Lambda, DynamoDB, S3, and Cognito

#### Common Destroy Failures and Fixes

| Error | Cause | Fix |
|---|---|---|
| `BucketNotEmpty` | S3 bucket still contains objects or old versions | Repeat Step 1 — confirm **Show versions** is empty, not just the current object list |
| `DependencyViolation` on the CloudFront distribution's origin | Distribution wasn't fully disabled before destroy ran | Repeat Step 2, wait for status **Disabled** (not just clicking Disable) before retrying `terraform destroy` |
| DynamoDB table fails to delete | Deletion protection enabled on the table | **DynamoDB** console → table → **Actions** → **Turn off deletion protection**, then retry |
| Cognito user pool domain fails to delete | Domain still attached to the app client's Hosted UI configuration | Remove the domain association from the app client first, or destroy in this order: app client → domain → user pool |
| `terraform destroy` partially completes, then fails | One resource blocked deletion (any of the above), leaving dependent resources undestroyed | Fix the specific blocker, then re-run `terraform destroy` — it's safe to re-run, it only acts on resources still present in state |