---
title: "5.5. Cleanup"
weight: 5
---

After completing the workshop or when you no longer need this practice environment, it is crucial to clean up (destroy) the AWS resources to avoid unwanted charges.

## Step-by-step cleanup guide

We will use Terraform to tear down the entire infrastructure we created. The main command is:

```bash
terraform destroy --auto-approve
```

## Crucial Warning Notice

> [!WARNING]
> **Empty S3 Buckets before running Terraform Destroy**
> 
> By default, Terraform **cannot delete** an Amazon S3 Bucket if it still contains data (objects). If you run `terraform destroy` when an S3 bucket is not empty, the process will fail with a `BucketNotEmpty` error.
> 
> **How to fix:** 
> Before running `terraform destroy`, you must use the AWS Console or AWS CLI to empty all contents in the PubliCast S3 Media Bucket.
> 
> *Using AWS CLI to empty the bucket:*
> ```bash
> aws s3 rm s3://your-publicast-media-bucket-name --recursive
> ```
> *(Replace `your-publicast-media-bucket-name` with your actual bucket name).* Once the bucket is empty, the `terraform destroy` command will execute smoothly.
