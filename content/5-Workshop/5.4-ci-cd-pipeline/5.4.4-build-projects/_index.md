---
title : "5.4.4. CodeBuild Projects"
weight : 4
---

## AWS CodeBuild Integration

**Purpose:** AWS CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces ready-to-deploy software packages (like Docker images or compiled React static files).

**Key Terraform Resources:** 
In our Terraform code, we provision two distinct `aws_codebuild_project` resources to handle the entirely different requirements of our frontend and backend:

*   **Backend Build Project:** 
    *   We configure it to use a standard Amazon Linux 2 image.
    *   Crucially, we set `privileged_mode = true` in the environment block. This is strictly required to run the Docker daemon inside the CodeBuild container, enabling us to build our microservice Docker images.
    *   We inject runtime environment variables (like `IMAGE_REPO_NAME` and `ECS_CLUSTER_NAME`) so the `backend/buildspec.yml` script knows exactly where to push the images and update the services.
*   **Frontend Build Project:** 
    *   We set `privileged_mode = false` because we only need to run standard Node.js `npm run build` commands, not Docker.
    *   We inject `S3_BUCKET_NAME` and `CLOUDFRONT_DIST_ID` so the `frontend/buildspec.yml` script can upload the static HTML/JS/CSS files and invalidate the CDN cache.
*   **IAM Roles:** Both projects share a strict `aws_iam_role` that restricts them to only interacting with ECR, ECS, S3, and CloudFront.

📁 **Source Code:** Open `terraform/modules/ci/cd/main.tf` in your local IDE to view the CodeBuild configuration.

**Result:** After running `terraform apply` and waiting for the provisioning to finish, the result in the AWS Console will look like this:

{{< img "images/Workshop/services/codebuild.png" "AWS Console - CodeBuild Projects" >}}

---

### Exploring the Build Process

To truly see the power of AWS CodeBuild in action, click on the name of one of the projects (e.g., `publiast-staging-backend-build`) and navigate to the **Build logs** (or **Phase details**) tab of any recent Build run.

Here, you will see a real terminal interface displaying detailed line-by-line execution of your automated commands (such as `docker build`, `docker push`, or `npm run build`). This is an extremely useful place to debug if your Pipeline encounters an error.

{{< img "images/Workshop/services/codebuild-logs.png" "AWS Console - CodeBuild Logs" >}}
<p align="center"><i>Detailed execution logs inside CodeBuild</i></p>
