---
title : "5.4.2. Pipeline Artifacts (S3)"
weight : 2
---

## S3 Bucket for Pipeline Artifacts

When AWS CodePipeline runs, it moves data between different stages (e.g., from the **Source** stage to the **Build** stage). To do this securely and reliably, CodePipeline requires a temporary storage location.

In our Terraform configuration (`ci/cd/main.tf`), we provision a dedicated Amazon S3 bucket specifically for this purpose:

```hcl
# ------------------------------------------------------------------------------
# 1. S3 BUCKET TO STORE CODEPIPELINE ARTIFACTS
# ------------------------------------------------------------------------------
resource "aws_s3_bucket" "pipeline_artifacts" {
  bucket        = "${var.project}-${var.environment}-pipeline-artifacts"
  force_destroy = true
}
```

### Shared Architecture
It is important to note that this single S3 bucket is **shared by both pipelines**. Both the Backend CodePipeline and the Frontend CodePipeline use this exact same bucket as their `artifact_store`.

### How it works:
1. When a developer pushes code to GitHub, the **Source** stage of both pipelines zips the repository code and uploads it to this S3 bucket.
2. The **Build** stage (AWS CodeBuild) downloads this zip file from the S3 bucket, extracts it, and begins executing the respective `buildspec.yml` instructions for either frontend or backend.

> [!NOTE]
> We set `force_destroy = true` on this bucket. This is because pipeline artifacts are temporary and regenerate on every run. If we ever need to destroy the Terraform environment (like at the end of this workshop), we want Terraform to automatically delete this bucket even if it contains leftover artifact zip files.
