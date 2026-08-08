---
title: "5.3.3.3. Compute & Container Layer"
weight: 3
---

This layer hosts the actual execution environment for our decoupled microservices (API, Worker Light, Worker Heavy). Instead of managing traditional virtual machines, we use serverless container technology. This means AWS handles the underlying server maintenance, and we only pay for the exact CPU and memory our containers consume while running.

## Elastic Container Registry (`ecr`)
**Purpose:** A fully managed Docker container registry. Think of it as a secure, private version of Docker Hub or GitHub, but specifically for storing your compiled application images.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_ecr_repository`: Creates a private repository for each of our microservices.
    *   **Configured Attributes:** 
        *   `name`: The name of the repository (e.g., `publicast-staging-backend`).
        *   `image_scanning_configuration`: Contains a `scan_on_push = true` block. Every time a developer pushes a new Docker image, AWS automatically scans it for known OS and library vulnerabilities, ensuring we don't deploy compromised code.
        *   `force_delete = true`: Allows Terraform to destroy the repository even if it contains images (useful for workshop environments).
    
    {{< img "images/Workshop/services/ecr.png" "AWS Console - ECR Repository" >}}
    <p align="center"><i>AWS Console - ECR Repository</i></p>
    
    {{< img "images/Workshop/services/ecr-repo.png" "AWS Console - ECR Repository" >}}
    <p align="center"><i>AWS Console - ECR Images</i></p>
*   `aws_ecr_lifecycle_policy`: Every time we deploy an update via CI/CD, a new image is stored in ECR. To prevent storage costs from ballooning over time, we define a JSON lifecycle policy.
    *   **Configured Attributes:** 
        *   `repository`: Binds the policy to the repository we just created.
        *   `policy`: A JSON string defining our retention rules. Specifically, we set `countType = "imageCountMoreThan"` and `countNumber = 10`. This instructs ECR to only keep the 10 most recent images and automatically delete the older ones, keeping the registry clean and cost-effective.
    
    {{< img "images/Workshop/services/ecr-lifecycle.png" "AWS Console - ECR Lifecycle Policy" >}}
    <p align="center"><i>AWS Console - ECR Lifecycle Policy</i></p>

📁 **Source Code:** Open `terraform/modules/ecr/main.tf` in your local IDE to view the full configuration.

## ECS Cluster (`ecs`)
**Purpose:** A logical grouping of your tasks or services. It acts as the management boundary for our containerized applications. If ECR is where the containers are stored, the ECS Cluster is the designated zone where they actually run.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_ecs_cluster`: Creates the namespace and management plane.
    *   **Configured Attributes:** 
        *   `name`: The name of your cluster (e.g., `publicast-staging-cluster`). This acts as the central hub for our serverless services.
    
    {{< img "images/Workshop/services/ecs-cluster-main.png" "AWS Console - ECS Cluster" >}}
    <p align="center"><i>AWS Console - ECS Cluster</i></p>

📁 **Source Code:** Open `terraform/modules/ecs/main.tf` in your local IDE to view the full configuration.

## ECS Task Definitions (`ecs_task_definition`)
**Purpose:** Acts as a blueprint describing exactly how a Docker container should launch. If you are familiar with Docker, a Task Definition is very similar to a `docker-compose.yml` file, but designed for AWS.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_ecs_task_definition`: The overarching blueprint.
    *   **Configured Attributes:** 
        *   `network_mode = "awsvpc"`: A strict requirement for Fargate so each container gets its own IP address.
        *   `requires_compatibilities = ["FARGATE"]`: Validates that this task is built for serverless environments.
        *   `cpu` & `memory`: Defines the total resource limits for the entire task to prevent infinite consumption (e.g., `256` CPU units, `512` MB memory).
        *   `execution_role_arn` & `task_role_arn`: Attaches the specific IAM roles we built in Layer 1.
    
    {{< img "images/Workshop/services/taskdef-main.png" "AWS Console - Task Definition" >}}
    <p align="center"><i>AWS Console - Task Definition</i></p>
    

    {{< img "images/Workshop/services/taskdef-config.png" "AWS Console - Task Definition" >}}
    <p align="center"><i>AWS Console - Task Definition Configure</i></p>

*   `container_definitions` (JSON format): This is the heart of the configuration.
    *   **Configured Attributes:** 
        *   `image`: The URL pointing to the ECR repository.
        *   `portMappings`: Tells AWS which port to open (e.g., `containerPort: 8080`) so the load balancer can connect.
        *   `logConfiguration`: Tells the container to route `console.log` output to CloudWatch Logs.
        *   `secrets` array: Within the container definition, we face a security challenge: how do we give the container database passwords without writing them in plain text? Rather than using standard `environment` variables, we use the `secrets` array to map environment variables directly to the ARNs of AWS Secrets Manager objects. AWS will securely inject the passwords at the exact moment the container starts.
    
    {{< img "images/Workshop/services/taskdef-json.png" "AWS Console - Task Def JSON" >}}
    <p align="center"><i>AWS Console - Task Def JSON</i></p>

📁 **Source Code:** Open `terraform/modules/ecs_task_definition/main.tf` in your local IDE to view the full configuration.

## ECS Services (`ecs_service`)
**Purpose:** Maintains a specified number of instances (copies) of a Task Definition running simultaneously. It acts as the supervisor—if a container crashes, the ECS Service automatically starts a new one to replace it. It also handles integrating the containers with the Load Balancer.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_ecs_service`: Connects your Task Definition blueprint to the Cluster. As seen in the cluster overview, we have decoupled our application into a trio of distinct services running simultaneously:
    
    {{< img "images/Workshop/services/ecs-service.png" "AWS Console - ECS Services Overview" >}}
    <p align="center"><i>AWS Console - ECS Services Overview</i></p>

    Let's analyze the specific roles of this trio:

    1.  **API Service (`api-service`)**: The core backend handling live HTTP requests from users. It is uniquely configured with a `load_balancer` block to automatically register its tasks with the ALB Target Group so it can receive web traffic.
        {{< img "images/Workshop/services/ecs-service-api.png" "AWS Console - ECS API Service" >}}
        <p align="center"><i>AWS Console - ECS API Service</i></p>

    2.  **Worker Light Service (`worker-light-service`)**: Runs lightweight background jobs (e.g., sending emails, database cleanups) using BullMQ. It operates independently and securely without needing to connect to the Load Balancer.
        {{< img "images/Workshop/services/ecs-service-light.png" "AWS Console - ECS Worker Light Service" >}}
        <p align="center"><i>AWS Console - ECS Worker Light Service</i></p>

    3.  **Worker Heavy Service (`worker-heavy-service`)**: Dedicated entirely to CPU and Memory intense processing tasks (like video transcoding and chunking). By isolating this into its own service, heavy jobs will never crash or slow down our main API.
        {{< img "images/Workshop/services/ecs-service-heavy.png" "AWS Console - ECS Worker Heavy Service" >}}
        <p align="center"><i>AWS Console - ECS Worker Heavy Service</i></p>

    *   **Common Configured Attributes:** 
        *   `launch_type = "FARGATE"`: Instructs the service to launch containers serverlessly.
        *   `desired_count`: The number of container replicas we want running.
        *   `network_configuration` block: Places the running containers into the `subnets` (Private Subnets) we created in Layer 1, attaches the `security_groups`, and sets `assign_public_ip = false` to ensure maximum security.

📁 **Source Code:** Open `terraform/modules/ecs_service/main.tf` in your local IDE to view the full configuration.
