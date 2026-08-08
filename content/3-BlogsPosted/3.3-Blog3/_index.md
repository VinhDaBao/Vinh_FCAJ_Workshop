---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---


# CI/CD ON AWS – WHEN DEPLOYMENT IS NO LONGER “LOG INTO THE SERVER AND COPY THE CODE”


During the development of our project, one of the tasks we encountered most often was not developing new features, but deploying the application. Every time there was a small change, the team had to rebuild the application, log into the server, update the source code, restart the service, and check whether everything was working properly. Forgetting a single step or having two team members work on the same environment could easily cause deployment issues.

After learning more about DevOps and AWS services, we decided to build a **CI/CD** process to automate the entire deployment workflow.

## Starting with GitHub and Monorepo

The starting point was moving the entire source code to GitHub using a **monorepo** structure, where the backend and frontend are managed in the same repository but have two independent build processes. This makes version management more convenient while reducing the effort required to synchronize changes between the two parts of the system.

Whenever a new commit is pushed to the deployment branch, **AWS CodeStar Connection** establishes a secure connection between GitHub and AWS to trigger the pipeline. This allows AWS to monitor changes in the repository without requiring a manually configured webhook.

## CodePipeline and Infrastructure as Code

After receiving the source code, **AWS CodePipeline** starts the workflow defined through Terraform. Instead of manually creating each service through the AWS Console, the entire infrastructure is described as code using **Infrastructure as Code**.

This provides several benefits, including easier version control, reusability, and especially the ability to recreate the entire infrastructure with only a few Terraform commands.

In our pipeline, the backend and frontend are separated into two independent pipelines. This allows each component to be deployed independently. If only the user interface changes, the frontend can be built and deployed without affecting the backend. Similarly, when the backend API or business logic changes, the frontend does not need to be rebuilt.

## Building the Backend with CodeBuild and ECR

During the Build stage, **AWS CodeBuild** is responsible for building the source code. Since the backend uses Docker, CodeBuild is configured with *privileged mode* to allow Docker image creation.

After the image is successfully built, it is pushed to **Amazon ECR**. Environment variables such as the repository name, AWS Account ID, ECS Cluster, and ECS Service names are provided through Terraform, so the `buildspec` does not need to contain fixed values.

## Deploying the Backend with Amazon ECS

After the new image is available in ECR, the pipeline continues by updating the services running on **Amazon ECS**.

In our project, the backend is divided into several services:

* API
* Worker Light
* Worker Heavy

This structure allows each component to scale independently according to the system workload. When a new version is deployed, ECS performs a **rolling update**, replacing the old containers with the new ones without stopping the entire system.

## Deploying the Frontend with S3 and CloudFront

For the frontend, the process is slightly simpler. After CodeBuild finishes building the application, all static files are uploaded to **Amazon S3**.

The pipeline then performs an **Amazon CloudFront invalidation** to clear the cache, allowing users to receive the latest version immediately instead of waiting for the existing cache to expire.

## Pipeline Artifacts

Another small but important detail is that CodePipeline needs a place to store temporary artifacts between stages.

Therefore, our team uses a dedicated **S3 Bucket for Pipeline Artifacts**. Although users rarely interact with this bucket directly, it serves as a storage location for source packages and build results that are required by subsequent pipeline stages.

## Managing IAM with Terraform

All **IAM Roles** and **IAM Policies** required by CodePipeline and CodeBuild are also defined using Terraform. This allows access permissions to be managed centrally instead of being configured manually through the AWS Console.

When deploying to another environment such as staging or production, we only need to change variables such as `project`, `environment`, or `branch` to reuse almost the entire module.

## The Automated Deployment Process

The most valuable result of building this system was not necessarily faster builds, but greater stability during deployment.

Previously, every release required checking many manual steps. Now, the process is almost reduced to:

1. Push code to GitHub.
2. CodePipeline automatically retrieves the source.
3. CodeBuild builds the application.
4. The Docker image is updated in ECR.
5. ECS deploys the new version.
6. The frontend is synchronized to S3 and CloudFront refreshes the cache.

As a result, deployment time is significantly reduced and all team members follow the same deployment process. Troubleshooting also becomes easier because each build has its own logs, status, and execution history stored on AWS.

## Lessons from Infrastructure as Code

Through building this system, the team also gained a better understanding of the role of **Infrastructure as Code**.

Instead of viewing Terraform simply as a tool for creating AWS resources, we began to treat the entire infrastructure as part of the source code. Every change can be committed, reviewed, and stored in the same way as software development changes.

For anyone learning about CI/CD on AWS, I believe this is a practical model to start with because it takes advantage of AWS managed services while also providing hands-on experience with pipeline organization in real-world projects.

Key points to know:

* Understand how to build a CI/CD pipeline on AWS.
* Learn how to connect GitHub with AWS CodePipeline.
* Understand the roles of CodeBuild, ECR, ECS, S3, and CloudFront in deployment.
* Learn how to separate backend and frontend pipelines for independent deployment.
* Understand how Terraform can be used for Infrastructure as Code.
* Reduce dependency on manual deployment processes.
* Improve deployment tracking through logs, execution history, and pipeline status.
