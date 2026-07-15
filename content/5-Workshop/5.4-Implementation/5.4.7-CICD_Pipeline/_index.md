---
title : "CI/CD Pipeline — CodePipeline"
date : 2026-07-14
weight : 7
chapter : false
pre : " <b> 5.4.7. </b> "
---

#### Goal

Automate infrastructure changes through a pipeline that runs tests and security checks first, requires a human approval before touching production infrastructure, and only then applies the change via Terraform.

#### Pipeline Stages

    GitHub push (main branch)
         |
         v
    Source ──── CodeStar connection pulls the repo
         |
         v
    Test ──── CodeBuild: pytest
         |
         v
    Security Scan ──── CodeBuild: bandit (Python) + tfsec (Terraform)
         |
         v
    Terraform Plan ──── CodeBuild: terraform plan, output saved as artifact
         |
         v
    Manual Approval ──── a human reviews the plan output before proceeding
         |
         v
    Terraform Apply ──── CodeBuild: terraform apply, deploys the reviewed plan

#### Step 1 — Set Up Remote Terraform State

Before the pipeline can run `terraform plan`/`apply` non-interactively, Terraform's state needs to live somewhere the pipeline can reach — not on a developer's laptop.

1. **S3** console → **Create bucket** → name: `doc-summarizer-tfstate-YOUR_ACCOUNT_ID`. Enable **Bucket Versioning** (protects against a corrupted state file overwriting a good one).
2. **DynamoDB** console → **Create table** → name: `doc-summarizer-tfstate-lock` → partition key: `LockID` (String) → **On-demand** capacity. This table prevents two concurrent `terraform apply` runs from corrupting state.
3. In `terraform/backend.tf`:

```hcl
terraform {
  backend "s3" {
    bucket         = "doc-summarizer-tfstate-476433508694"
    key            = "terraform.tfstate"
    region         = "ap-southeast-1"
    dynamodb_table = "doc-summarizer-tfstate-lock"
    encrypt        = true
  }
}
```

4. Run `terraform init` locally once to migrate state into this backend.

#### Step 2 — Connect GitHub to CodePipeline

1. **CodePipeline** console → **Settings** → **Connections** → **Create connection**.
2. Provider: **GitHub**.
3. Follow the prompt to authorize AWS to access the repository.
4. Note the **Connection ARN**, e.g.:
   ```
   arn:aws:codestar-connections:ap-southeast-1:476433508694:connection/2e294d39-b237-459c-8960-4f27f2e50896
   ```

#### Step 3 — Define the Pipeline in Terraform

Rather than clicking through the CodePipeline console wizard, the pipeline itself is defined as code in `terraform/pipeline.tf`, so its stages are versioned alongside everything else:

```hcl
resource "aws_codepipeline" "this" {
  name     = "doc-summarizer-pipeline"
  role_arn = aws_iam_role.pipeline.arn

  artifact_store {
    location = aws_s3_bucket.pipeline_artifacts.bucket
    type     = "S3"
  }

  stage {
    name = "Source"
    action {
      name             = "Source"
      category         = "Source"
      owner            = "AWS"
      provider         = "CodeStarSourceConnection"
      version          = "1"
      output_artifacts = ["source_output"]
      configuration = {
        ConnectionArn    = var.codestar_connection_arn
        FullRepositoryId = "your-org/Project_Abstract"
        BranchName       = "main"
      }
    }
  }

  stage {
    name = "Test"
    action {
      name             = "RunTests"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["test_output"]
      configuration = {
        ProjectName = aws_codebuild_project.test.name
      }
    }
  }

  stage {
    name = "SecurityScan"
    action {
      name             = "SecurityScan"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["security_output"]
      configuration = {
        ProjectName = aws_codebuild_project.security_scan.name
      }
    }
  }

  stage {
    name = "TerraformPlan"
    action {
      name             = "Plan"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["plan_output"]
      configuration = {
        ProjectName = aws_codebuild_project.terraform_plan.name
      }
    }
  }

  stage {
    name = "ManualApproval"
    action {
      name     = "ApproveDeployment"
      category = "Approval"
      owner    = "AWS"
      provider = "Manual"
      version  = "1"
    }
  }

  stage {
    name = "TerraformApply"
    action {
      name            = "Apply"
      category        = "Build"
      owner           = "AWS"
      provider        = "CodeBuild"
      version         = "1"
      input_artifacts = ["plan_output"]
      configuration = {
        ProjectName = aws_codebuild_project.apply.name
      }
    }
  }
}
```

#### Step 4 — Define the CodeBuild Buildspecs

Each stage runs a `buildspec.yml`. Test stage:

```yaml
version: 0.2
phases:
  install:
    commands:
      - pip install -r requirements.txt --break-system-packages
  build:
    commands:
      - pytest tests/ -v
```

Security scan stage:

```yaml
version: 0.2
phases:
  build:
    commands:
      - bandit -r src/ -f json -o bandit-report.json
      - tfsec terraform/ --format json
```

Terraform plan stage:

```yaml
version: 0.2
phases:
  install:
    commands:
      - curl -o terraform.zip https://releases.hashicorp.com/terraform/1.9.0/terraform_1.9.0_linux_amd64.zip
      - unzip terraform.zip && mv terraform /usr/local/bin/
  build:
    commands:
      - cd terraform && terraform init
      - terraform plan -out=tfplan
artifacts:
  files:
    - terraform/tfplan
```

Terraform apply stage runs the same install phase, then `terraform apply tfplan` in the build phase.

#### Step 5 — Apply the Pipeline

```bash
cd terraform
AWS_PROFILE=phatnguyen terraform plan
AWS_PROFILE=phatnguyen terraform apply
```

Review the plan output before confirming — this creates the pipeline, CodeBuild projects, and their IAM roles.

#### Step 6 — Test the Full Pipeline

1. Push a small, safe change to `main` (e.g. a comment update).
2. **CodePipeline** console → `doc-summarizer-pipeline`.
3. Watch each stage turn green in sequence: Source → Test → Security Scan → Terraform Plan.
4. At **Manual Approval**, click **Review** → inspect the plan output artifact → **Approve**.
5. Confirm **Terraform Apply** completes successfully.

**How to verify:** All stages show a green checkmark, and the pipeline's execution history shows `Succeeded` as the final status.

#### Why the Manual Approval Gate Matters

Tests and security scans are safe to run unattended — they don't change anything. `terraform apply` can modify or destroy real infrastructure, so a human reviews the exact plan before it's allowed to proceed. This is a standard production safeguard: automate everything up to the point of irreversible change, then require a person to confirm it.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Source stage fails immediately | CodeStar connection status is **Pending** rather than **Available** | Return to **Settings → Connections** in the console and complete the GitHub authorization handshake |
| Terraform Plan stage fails with a state lock error | A previous run didn't release the DynamoDB lock (e.g. it was manually cancelled) | `AWS_PROFILE=yourname terraform force-unlock LOCK_ID` — get `LOCK_ID` from the error message |
| Pipeline stuck at Manual Approval with no reviewer able to approve | No SNS topic or IAM permission configured for the approval action | Confirm the `aws_codepipeline` approval action has an SNS topic ARN, and that your IAM user has `codepipeline:PutApprovalResult` |
| Terraform Apply stage fails with a permissions error mid-apply | CodeBuild's IAM role lacks a permission needed for a newly added resource type | This project currently accepts `AdministratorAccess` on the CodeBuild role as a known compliance gap — see Section 3 (Security and IAM Fundamentals) and Section 6.2 |