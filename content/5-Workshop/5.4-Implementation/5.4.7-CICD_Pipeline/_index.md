---
title : "CI/CD Pipeline — CodePipeline"
date : 2026-07-14
weight : 7
chapter : false
pre : " <b> 5.4.7. </b> "
---

#### Goal

Automate infrastructure changes through a pipeline that tests and scans the code first, requires a human approval before any change touches real infrastructure, and applies exactly the plan that was reviewed — nothing re-planned at apply time.

#### Pipeline Stages

    GitHub push (main branch)
         |
         v
    Source ──── CodeStar connection pulls the repo
         |
         v
    Test ──── one CodeBuild project runs pytest, bandit, tfsec,
         |     terraform fmt -check, and terraform plan
         v
    Approve ──── a human reviews the plan output before proceeding
         |
         v
    Apply ──── a second CodeBuild project runs terraform apply
                against the exact plan artifact from Test

The pipeline itself is defined in Terraform (terraform/modules/pipeline/main.tf), so its stages are versioned the same way as the infrastructure it deploys.

#### Step 1 — Set Up Remote Terraform State

Before the pipeline can run terraform plan/apply non-interactively, Terraform's state needs to live somewhere the pipeline can reach — not on a developer's laptop.

1. **S3** console → **Create bucket** → name: doc-summarizer-tfstate-YOUR_ACCOUNT_ID. Enable **Bucket Versioning** (protects against a corrupted state file overwriting a good one).
2. **DynamoDB** console → **Create table** → name: doc-summarizer-tfstate-lock → partition key: LockID (String) → **On-demand** capacity. This table prevents two concurrent terraform apply runs from corrupting state.
3. In terraform/backend.tf:

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

4. Run terraform init locally once to migrate state into this backend.

#### Step 2 — Connect GitHub to CodePipeline

The GitHub connection itself (aws_codestarconnections_connection) is created by Terraform, but AWS requires a one-time manual handshake to authorize it — this step can't be automated away.

1. Apply the pipeline module once (Step 3 below creates the connection resource in a **Pending** state).
2. **CodePipeline** console → **Settings** → **Connections** → find the connection → **Update pending connection**.
3. Follow the prompt to authorize AWS to access the repository.
4. Confirm the connection status changes to **Available**.

#### Step 3 — The Pipeline as Code

terraform/modules/pipeline/main.tf defines everything: the GitHub connection, an encrypted+access-blocked S3 artifact bucket, two CodeBuild projects, IAM roles scoped to what each stage needs, and the four-stage aws_codepipeline resource itself.

```hcl
resource "aws_codepipeline" "main" {
  name     = "${var.project_name}-pipeline"
  role_arn = aws_iam_role.pipeline.arn

  artifact_store {
    location = aws_s3_bucket.artifacts.bucket
    type     = "S3"
  }

  stage {
    name = "Source"
    action {
      name             = "GitHub"
      category         = "Source"
      owner            = "AWS"
      provider         = "CodeStarSourceConnection"
      output_artifacts = ["source"]
      configuration = {
        ConnectionArn    = aws_codestarconnections_connection.github.arn
        FullRepositoryId = var.github_repo
        BranchName       = var.github_branch
      }
    }
  }

  stage {
    name = "Test"
    action {
      name             = "TestAndPlan"
      category         = "Build"
      provider         = "CodeBuild"
      input_artifacts  = ["source"]
      output_artifacts = ["tfplan"]
      configuration = { ProjectName = aws_codebuild_project.test.name }
    }
  }

  stage {
    name = "Approve"
    action {
      name     = "ManualApproval"
      category = "Approval"
      provider = "Manual"
      configuration = {
        CustomData = "Review the Terraform plan in the Test stage logs before approving."
      }
    }
  }

  stage {
    name = "Apply"
    action {
      name            = "TerraformApply"
      category        = "Build"
      provider        = "CodeBuild"
      input_artifacts = ["tfplan"]
      configuration   = { ProjectName = aws_codebuild_project.apply.name }
    }
  }
}
```

Passing the tfplan artifact straight from Test into Apply — instead of re-running terraform plan after approval — means what gets approved is exactly what gets applied. There's no window where the plan could drift between review and execution.

#### Step 4 — The Buildspecs

**Test stage** (buildspec-test.yml) runs every check before touching infrastructure, and fails the build if any of them fail:

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - # install terraform, tfsec
      - pip install -q -r requirements.txt
      - pip install -q -r tests/requirements-test.txt
      - pip install -q bandit
  build:
    commands:
      - PYTHONPATH=. pytest tests/ -v
      - bandit -r src/lambda_fn/ app/ -ll
      - tfsec terraform/
      - cd terraform
      - terraform init
      - terraform fmt -check -recursive
      - terraform plan -out=tfplan.binary
      - cd ..
artifacts:
  files:
    - terraform/tfplan.binary
    - terraform/**/*
    - src/**/*
    - buildspec-apply.yml
  name: tfplan
```

Unit tests, a Python security scan, a Terraform security scan, and a formatting check all gate the plan — none of them modify anything. The tfplan.binary produced here is the one artifact that carries forward to Apply.

**Apply stage** (buildspec-apply.yml) is deliberately minimal — its only job is to apply the artifact that already passed Test and Approve:

```yaml
version: 0.2
phases:
  install:
    commands:
      - # install terraform
  build:
    commands:
      - cd terraform
      - terraform init
      - terraform apply -auto-approve tfplan.binary
```

#### Step 5 — Apply the Pipeline

```bash
cd terraform
terraform plan
terraform apply
```

Review the plan output before confirming — this creates the pipeline, both CodeBuild projects, and their IAM roles.

#### Step 6 — Test the Full Pipeline

1. Push a small, safe change to main (e.g. a comment update).
2. **CodePipeline** console → doc-summarizer-pipeline.
3. Watch each stage turn green in sequence: Source → Test.
4. At **Approve**, click **Review** → inspect the Test stage logs for the plan output → **Approve**.
5. Confirm **Apply** completes successfully.

**How to verify:** All stages show a green checkmark, and the pipeline's execution history shows Succeeded as the final status.

#### Why the Manual Approval Gate Matters

Tests, scans, and terraform plan are safe to run unattended — none of them change anything. terraform apply can modify or destroy real infrastructure, so a human reviews the exact plan before it's allowed to proceed. This is a standard production safeguard: automate everything up to the point of irreversible change, then require a person to confirm it.

#### IAM Boundaries in the Pipeline

CodePipeline's own role is scoped to only what orchestration needs — not broad account access:

```hcl
resource "aws_iam_role_policy" "pipeline" {
  policy = jsonencode({
    Statement = [
      { Action = ["s3:GetObject", "s3:PutObject", "s3:GetBucketVersioning"] },
      { Action = ["codebuild:StartBuild", "codebuild:BatchGetBuilds"] },
      { Action = ["codestar-connections:UseConnection"] },
    ]
  })
}
```

CodeBuild's role currently uses AdministratorAccess, since it needs to create or modify every resource type Terraform manages in this project. This is flagged directly in the module as a known scope-down item for any real production use — see Section 5.4.8 and Section 6.2.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Source stage fails immediately | CodeStar connection status is **Pending** rather than **Available** | Return to **Settings → Connections** in the console and complete the GitHub authorization handshake |
| Test stage fails with a state lock error | A previous run didn't release the DynamoDB lock (e.g. it was manually cancelled) | terraform force-unlock LOCK_ID — get LOCK_ID from the error message |
| Pipeline stuck at Approve with no reviewer able to approve | IAM permission missing for the approval action | Confirm your IAM user has codepipeline:PutApprovalResult |
| Apply stage fails with a permissions error mid-apply | CodeBuild's IAM role lacks a permission needed for a newly added resource type | This project currently accepts AdministratorAccess on the CodeBuild role as a known compliance gap — see Section 5.4.8 and Section 6.2 |