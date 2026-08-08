---
title: "5.4. AWS-Native CI/CD Pipeline"
weight: 4
---

The Continuous Integration and Deployment (CI/CD) workflow for the PubliCast application is fully automated using AWS-native developer tools. Instead of relying on third-party CI/CD runners like GitHub Actions, we provisioned **AWS CodePipeline** and **AWS CodeBuild** directly through Terraform.

## Continuous Integration & Deployment Workflow

The workflow is triggered automatically whenever a developer pushes changes to the specified branch in our GitHub Monorepo. Because it is a monorepo, we have split the automation into two separate, independent pipelines: one for the Backend and one for the Frontend.

## Pipeline Architecture Breakdown

Our `ci/cd` Terraform module provisions the following core components to make this automation possible:

1.  **CodeStar Connection (`aws_codestarconnections_connection`)**: 
    This acts as the secure bridge between our AWS environment and our GitHub repository. It securely listens for webhook events (like code pushes) without needing to store long-lived personal access tokens in Terraform.

2.  **Artifact Store (`aws_s3_bucket`)**: 
    A dedicated S3 bucket used internally by CodePipeline to store temporary zip files (source artifacts) as they are passed from the GitHub Source stage to the Build stage.

3.  **Backend Pipeline (`aws_codepipeline.backend`)**:
    *   **Source Stage**: Pulls the latest code from GitHub via the CodeStar connection.
    *   **Build Stage (CodeBuild)**: Runs a Linux container configured with `privileged_mode = true` (which is strictly required to run Docker-in-Docker). It executes instructions from the `backend/buildspec.yml` file: building the 3 microservice Docker images (API, Worker Light, Worker Heavy), pushing them to Amazon ECR, and triggering a rolling update on the ECS Fargate services.

4.  **Frontend Pipeline (`aws_codepipeline.frontend`)**:
    *   **Source Stage**: Pulls the latest code from the same GitHub monorepo.
    *   **Build Stage (CodeBuild)**: Executes instructions from the `frontend/buildspec.yml` file. It compiles the frontend assets (e.g., React/Vite) and syncs the compiled static files directly into the frontend S3 bucket. It then automatically invalidates the CloudFront CDN cache so end-users see the updates immediately.

5.  **Strict IAM Policies**:
    To ensure security, Terraform provisions specific `aws_iam_role` and `aws_iam_role_policy` blocks. These guarantee that the Backend CodeBuild runner only has permissions to touch ECR and ECS, while the Frontend CodeBuild runner only has permissions to upload to S3 and invalidate CloudFront.
