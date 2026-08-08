---
title: "5.2. Prerequisites"
weight: 2
---

Before deploying the infrastructure for the PubliCast project, ensure that the following tools are properly installed and configured on your machine:

## Pre-required tools

1.  **AWS CLI**: Installed and configured with your AWS credentials (Access Key ID & Secret Access Key). Ensure the IAM user has sufficient permissions to create resources like VPC, ECS, RDS, S3, etc.
2.  **Terraform**: Version >= 1.5.0. Used to deploy Infrastructure as Code (IaC).
3.  **Docker**: To build and test local images if necessary.
4.  **Git**: For source code management.

## Instructions to clone the project repository

Clone the source code containing the project's Terraform configuration to your local machine using the following commands:

```bash
git clone https://github.com/Nguyen-Thanh-Huy-io/FCAJ-AWS-Project.git
cd publicast-terraform/terraform/envs/staging
```

*Note: Replace `https://github.com/your-username/publicast-terraform.git` with your actual repository URL.*