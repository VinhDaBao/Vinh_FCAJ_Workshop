---
title: "5.3.3.4. Storage & Database Layer"
weight: 4
---

This layer provides the persistent memory of our application. While our ECS containers are stateless (meaning if they crash, any data inside them is lost), this layer provides persistent data storage, rapid caching, and secure management of sensitive credentials.

## Amazon S3 Media Bucket (`s3_cloudfront`)
**Purpose:** Highly durable object storage used to indefinitely store large volumes of user-uploaded videos and images. Unlike a traditional hard drive that can fill up, S3 provides virtually infinite storage capacity.

> [!IMPORTANT]
> **Use a Globally Unique Bucket Name**
> Amazon S3 bucket names must be globally unique across all AWS accounts. When deploying this Terraform code, ensure you change the default bucket name (e.g., `publicast-media-storage`) to something unique for your own environment.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_s3_bucket`: Creates the base storage buckets for our application. As seen in the console, our project utilizes three distinct buckets, each with a specific purpose:
    1.  **`frontend` bucket**: Hosted by CloudFront, this bucket stores the compiled static assets of our React Single Page Application (HTML, CSS, JS).
    2.  **`backend-storage` bucket**: Also served securely via CloudFront, this is the massive "hard drive" dedicated to storing user-uploaded media files (videos, thumbnails, profile pictures).
    3.  **`pipeline-artifacts` bucket**: (Created by our CI/CD module) This is used internally by AWS CodePipeline to temporarily store intermediate build files during deployments.
    
    {{< img "images/Workshop/services/s3-bucket.png" "AWS Console - S3 Buckets List" >}}
    <p align="center"><i>AWS Console - S3 Buckets List</i></p>
    
*   `aws_s3_bucket_public_access_block`: A crucial security measure that forcibly blocks all public Access Control Lists (ACLs) and bucket policies. This ensures the bucket is locked down at the account level so no one can accidentally make the entire bucket public.
    *   **Configured Attributes:** 
        *   `block_public_acls = true` & `block_public_policy = true`: Hard-codes the lockdown settings.
*   `aws_s3_bucket_policy`: We apply a strict JSON IAM policy to the bucket. Looking at the JSON in the console, we can see exactly how this zero-trust mechanism works:
    *   **`Principal`**: Tells AWS that only the `cloudfront.amazonaws.com` service is allowed to ask for files.
    *   **`Action`**: We only grant the `s3:GetObject` (read) permission. No one can delete or overwrite files through this channel.
    *   **`Condition`**: The ultimate lock. It explicitly checks that the request is coming from the exact `AWS:SourceArn` of our specific CloudFront Distribution. 
    
    This ensures media cannot be accessed directly via the raw S3 URL, preventing bandwidth theft and forcing all traffic through our high-speed CDN.
    
    {{< img "images/Workshop/services/s3-policy.png" "AWS Console - S3 Bucket Policy" >}}
    <p align="center"><i>AWS Console - S3 Bucket Policy</i></p>

📁 **Source Code:** Open `terraform/modules/s3_cloudfront/main.tf` in your local IDE to view the full configuration.

## Relational Database Service (`rds`)
**Purpose:** A fully managed, highly available relational database service. We use it to store structured data that requires complex queries and relationships, such as Users, Posts, and Comments.

Before we can deploy the actual database engine, AWS requires us to define exactly *where* in our network it is allowed to live securely. Therefore, the Terraform configuration is split into two logical steps: defining the network boundary, and then deploying the database inside it.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_db_subnet_group`: Groups our Private Subnets together. This tells RDS exactly which isolated network zones it is allowed to deploy its database nodes into.
    *   **Configured Attributes:** 
        *   `subnet_ids`: An array of Private Subnet IDs retrieved from our VPC layer. As you can see in the screenshot, this successfully restricts the database deployment strictly to `publicast-staging-private-0` and `publicast-staging-private-1` across two different Availability Zones. This guarantees both extreme network security and high availability.
    
    {{< img "images/Workshop/services/rds-subnet.png" "AWS Console - RDS Subnet Group" >}}
    <p align="center"><i>AWS Console - RDS Subnet Group</i></p>
    
*   `aws_db_instance`: Defines the core database configuration. 
    *   **Configured Attributes:** 
        *   `engine = "mysql"` & `instance_class = "db.t3.micro"`: Defines what software and hardware the database runs on. We use a `t3.micro` to keep workshop costs low.
        *   `allocated_storage`: The starting disk size in Gigabytes.
        *   `vpc_security_group_ids`: Attaches the strict `rds-sg` firewall which only allows traffic from ECS.
        *   `multi_az = false`: For a real production environment, this should be `true` for high availability. However, to save costs during this workshop, we explicitly disable multi-AZ replicas.
        *   `skip_final_snapshot = true`: For workshop or dev environments, we set this so Terraform can destroy the DB quickly without waiting 15 minutes to take a final backup.
    
    {{< img "images/Workshop/services/rds-instance.png" "AWS Console - RDS Instance" >}}
    <p align="center"><i>AWS Console - RDS Instance</i></p>

📁 **Source Code:** Open `terraform/modules/rds/main.tf` in your local IDE to view the full configuration.

## Amazon ElastiCache (`elasticache`)
**Purpose:** A managed in-memory Redis data store. Because reading from memory (RAM) is vastly faster than reading from a traditional database disk, we use Redis for caching API responses and powering our BullMQ background job queues.

> [!TIP]
> **Finding your Redis Cluster**
> In the ElastiCache console, you may not see your cluster immediately. Look at the left navigation pane under **Resources** and click **Redis OSS caches** to find your running components.

{{< img "images/Workshop/services/elasticache-menu.png" "AWS Console - ElastiCache OSS Menu" >}}
<p align="center"><i>AWS Console - ElastiCache OSS Menu</i></p>

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_elasticache_subnet_group`: Places the Redis cluster securely in the Private Subnets alongside the RDS database.
    *   **Configured Attributes:** 
        *   `subnet_ids`: The same Private Subnet IDs used for RDS.
    
    {{< img "images/Workshop/services/elasticache-subnet.png" "AWS Console - ElastiCache Subnet Group" >}}
    <p align="center"><i>AWS Console - ElastiCache Subnet Group</i></p>
    
*   `aws_elasticache_replication_group`: Deploys the Redis engine. 
    *   **Configured Attributes:** 
        *   `engine = "redis"` & `node_type = "cache.t3.micro"`: Defines the cache software and hardware. Similar to RDS, we use a `micro` instance to keep workshop costs low.
        *   `security_group_ids`: Attaches the `redis-sg` firewall to block unauthorized access.
        *   `num_cache_clusters = 1` & `automatic_failover_enabled = false`: For a real production environment, we would use multiple clusters and enable failover so our background workers never drop a task if a node goes down. However, for this workshop, we explicitly limit it to 1 node without failover to save costs.
    
    {{< img "images/Workshop/services/elasticache-cluster.png" "AWS Console - ElastiCache Cluster" >}}
    <p align="center"><i>AWS Console - ElastiCache Cluster</i></p>

📁 **Source Code:** Open `terraform/modules/elasticache/main.tf` in your local IDE to view the full configuration.

## AWS Secrets Manager (`secrets_manager`)
**Purpose:** Securely stores, encrypts, and rotates sensitive credentials (like database passwords and third-party API keys) without exposing them in code or plain text.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_secretsmanager_secret`: Creates the logical container (the name and description) of the secret in AWS.
    *   **Configured Attributes:** 
        *   `name` & `description`: Labels the secret box so administrators know what it is for.
    
    {{< img "images/Workshop/services/secrets-list.png" "AWS Console - Secrets Manager List" >}}
    <p align="center"><i>AWS Console - Secrets Manager List</i></p>
    
*   `aws_secretsmanager_secret_version`: This resource holds the actual sensitive data. As you can see in the screenshot below, instead of creating a dozen different isolated secrets, we consolidate them all into one secure vault.
    *   **Configured Attributes:** 
        *   `secret_id`: Binds the data to the logical container.
        *   `secret_string`: We use Terraform's `jsonencode()` function to safely format a map of all our project credentials (like `DB_PASSWORD`, `REDIS_HOST`, and `YOUTUBE_CLIENT_ID`) into the single JSON string you see in the image. During deployment, Terraform pushes this string to AWS encrypted. Later, our ECS tasks will dynamically fetch and inject these exact key-value pairs into the application at runtime.
    
    {{< img "images/Workshop/services/secrets-value.png" "AWS Console - Secrets Manager Value" >}}
    <p align="center"><i>AWS Console - Secrets Manager Value</i></p>

📁 **Source Code:** Open `terraform/modules/secrets_manager/main.tf` in your local IDE to view the full configuration.
