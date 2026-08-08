---
title : "5.4.5. CodePipelines"
weight : 5
---

## AWS CodePipeline Orchestration

**Purpose:** AWS CodePipeline orchestrates the entire release process. It acts as the "glue" that connects the GitHub repository (Source) and the CodeBuild projects (Build) into a unified, fully automated continuous delivery workflow.

**Key Terraform Resources:** 
We use the `aws_codepipeline` resource to define two parallel pipelines for the monorepo:

*   **Backend Pipeline & Frontend Pipeline:**
    *   **Git Triggers (Monorepo Strategy):** Because we use a Monorepo architecture, upgrading to `pipeline_type = "V2"` allows us to use the `trigger` block. This intelligently filters events by `file_paths`: the Backend Pipeline only triggers for changes in `backend/**`, and the Frontend Pipeline only triggers for changes in `frontend/**`. This saves significant time and build costs. *(Note: `DetectChanges = false` must be set in the Source action to let the trigger block take control).*
    *   **Artifact Store:** Both pipelines define an `artifact_store` block pointing to the S3 bucket we created in step 5.4.1. This is how files are securely passed between stages.
    *   **Stage 1 (Source):** Uses the `CodeStarSourceConnection` provider pointing to our GitHub monorepo. It outputs a zip file conceptually named `source_output`.
    *   **Stage 2 (Build):** Takes the `source_output` zip file as an `input_artifact` and passes it to the respective CodeBuild project.
*   **IAM Roles:** We attach a specific `aws_iam_role` to the pipelines that grants them permission to read/write to the S3 artifact bucket and trigger CodeBuild executions.

📁 **Source Code:** Open `terraform/modules/ci/cd/main.tf` in your local IDE to view the CodePipeline configuration.

> [!IMPORTANT]
> Once your Terraform deployment finishes and you authorize the CodeStar connection (as detailed in 5.4.3), you should see both pipelines automatically trigger and run successfully in the AWS Console.

**Result:** After running `terraform apply` and waiting for the provisioning to finish, the result in the AWS Console will look like this:

{{< img "images/Workshop/services/codepipeline.png" "AWS Console - CodePipeline" >}}

---

### Exploring the CodePipeline

Similar to CodeBuild, you can click directly on a pipeline's name (e.g., `publiast-staging-backend-pipeline`) to view its detailed execution flow.

Here, you will visually see the sequential Stages:
1. **Source:** Fetches the code from GitHub (you can even see the commit hash and commit message like `feat: change logo text to StreamHubbb for pipeline testing`).
2. **Build:** Triggers the corresponding CodeBuild project to compile the code and package it. If you click on the *AWS CodeBuild* link within this box, it will take you straight to the detailed Build logs screen.

This interface allows you to track exactly where your source code is in the deployment process and quickly identify which Stage is failing if an error occurs.

{{< img "images/Workshop/services/codepipeline-details.png" "AWS Console - CodePipeline Details" >}}
<p align="center"><i>Detailed stages inside CodePipeline</i></p>