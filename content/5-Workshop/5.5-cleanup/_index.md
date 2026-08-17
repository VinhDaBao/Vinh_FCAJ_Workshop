---
title: "5.5. Cleanup"
weight: 5
---

After completing the workshop or when you no longer need this practice environment, it is absolutely critical to clean up (destroy) all AWS resources. Failing to do so can result in unwanted, ongoing charges on your AWS bill, especially for resources like RDS, ElastiCache, and NAT Gateways.

## The Power of Terraform Destroy

One of the greatest benefits of Infrastructure as Code (IaC) is that tearing down an environment is just as easy as building it. Instead of hunting through the AWS Console to manually delete dozens of interconnected resources, Terraform tracks all state and dependencies to destroy them in the correct order.

The primary command to tear down the infrastructure is:

```bash
terraform destroy --auto-approve
```

However, before you run this command, there are a few manual prerequisites you **must** complete to ensure the process executes smoothly.

## Crucial Prerequisites Before Destruction

> [!WARNING]
> **Empty S3 Buckets before running Terraform Destroy**
> 
> By default, Terraform **cannot delete** an Amazon S3 Bucket if it still contains data (objects). This is a safety mechanism to prevent accidental data loss. If you run `terraform destroy` when an S3 bucket is not empty, the process will fail with a `BucketNotEmpty` error.
> 
> **How to fix:** 
> Before running `terraform destroy`, you must use the AWS Console or AWS CLI to empty all contents in the PubliCast S3 Media Bucket.
> 
> *Using AWS CLI to empty the bucket:*
> ```bash
> aws s3 rm s3://your-publicast-media-bucket-name --recursive
> ```
> *(Replace `your-publicast-media-bucket-name` with your actual bucket name).*

> [!IMPORTANT]
> **Clear ECR Image Repositories**
> 
> Similar to S3, Amazon Elastic Container Registry (ECR) repositories usually cannot be deleted if they still contain Docker images (unless the `force_delete` flag was explicitly hardcoded in Terraform). 
> It is highly recommended to navigate to the AWS ECR Console, select your backend and frontend repositories, select all images, and manually delete them before initiating the Terraform destroy command.

## Step-by-Step Cleanup Checklist

1.  **Empty S3 Buckets:** Ensure no user-uploaded media files are left inside the bucket.
2.  **Clear ECR Images:** Delete all Docker images pushed by your CI/CD pipeline.
3.  **Execute Terraform Destroy:** Run `terraform destroy --auto-approve` in your terminal. This process may take 10-15 minutes as AWS gracefully shuts down RDS databases, ElastiCache clusters, ECS tasks, and VPC networking components.
4.  **Verify via AWS Console:** Log into your AWS Console one last time. Quickly check the EC2 Dashboard (for lingering NAT Gateways or Load Balancers), RDS, and ElastiCache to confirm that **0 instances** are running.
5.  **Check Billing Dashboard:** Monitor your AWS Billing & Cost Management Dashboard over the next 24-48 hours to ensure that daily charges have dropped back to zero.

By rigorously following these steps, you maintain a clean AWS environment and avoid unexpected bill shock!
