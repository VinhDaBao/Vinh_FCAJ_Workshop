---
title: "5.3.1. Infrastructure Layout"
weight: 1
---

This section provides a high-level overview of the entire network and system architecture for the PubliCast project. The infrastructure is composed of multiple AWS services working in tandem to deliver a highly available, secure, and scalable application.

## Overall System Architecture

The PubliCast system is logically divided into several functional layers, each utilizing specific AWS managed services and corresponding Terraform modules.

### 1. Network & Security Layer
The foundation of the system is built on a highly secure Virtual Private Cloud (VPC).
*   **VPC, Subnets & Gateways:** Isolates the infrastructure into Public Subnets (internet-facing) and Private Subnets (secure backend).
*   **VPC Endpoints:** Provides a direct, cost-free shortcut to Amazon S3 for resources in the private subnets.
*   **Security Groups:** Acts as virtual firewalls to strictly control traffic between components (e.g., ensuring the database only accepts connections from the application containers).
*   **IAM (Identity and Access Management):** Enforces the principle of least privilege, granting services only the permissions they explicitly need to function.

### 2. Edge & Load Balancing Layer
This layer handles traffic originating from the public internet and routes it to the correct backend services.
*   **Route53:** Manages DNS resolution, pointing custom domain names to our entry points.
*   **Application Load Balancer (ALB):** Receives incoming HTTP/HTTPS requests and distributes them evenly across the backend API container instances.
*   **CloudFront (CDN):** Caches public media globally, reducing latency for end-users and offloading heavy download traffic from the core backend.

### 3. Compute & Container Layer
This is where the actual application code executes.
*   **Elastic Container Registry (ECR):** Stores the built Docker images for our microservices (API Service, Worker Light, Worker Heavy).
*   **Elastic Container Service (ECS) on Fargate:** Runs the application containers in a serverless compute environment. We don't have to manage the underlying servers. Auto-scaling rules automatically adjust the number of running containers based on real-time CPU and Memory usage.

### 4. Storage & Database Layer
This layer provides persistent data storage and high-speed caching.
*   **Amazon RDS (MySQL):** The primary relational database holding core application data (Users, Workspaces, Posts, etc.).
*   **Amazon ElastiCache (Redis):** Serves as both an in-memory cache to speed up API responses and as a message broker for the BullMQ background job queues.
*   **Amazon S3:** Secure, highly scalable object storage for all user-uploaded media files (videos, images).
*   **AWS Secrets Manager:** Securely stores database passwords and third-party API credentials, injecting them into the ECS containers at runtime to avoid hardcoding secrets.

### 5. Monitoring & Automation Layer
Ensures the system is observable, healthy, and deployments are seamless.
*   **CloudWatch:** Centralizes container logs and system metrics. It triggers auto-scaling policies or alerts when performance degrades.
*   **EventBridge:** Handles scheduled tasks (cron jobs) to automate routine system maintenance.
*   **CI/CD Pipeline (CodePipeline):** Automates the testing, building, and deployment of Docker images directly to ECS without manual intervention.