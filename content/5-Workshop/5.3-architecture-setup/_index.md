---
title: "5.3. Architecture and Setup"
weight: 3
---

In this chapter, we will dive deep into the AWS infrastructure that powers the PubliCast platform. We will explore how Terraform is used to provision and orchestrate various AWS services into a cohesive, production-ready environment.

You will learn about the different layers of our architecture, the specific configuration choices made for each service, and the step-by-step process to deploy the entire stack to your AWS account.

## Architectural Philosophy

Before diving into the code, it is important to understand the core principles that guided the system design of PubliCast:

1.  **Security by Design (Least Privilege):** Every component, from the CI/CD pipeline to the background workers, operates with the absolute minimum permissions required. Sensitive configurations and third-party API keys are strictly managed via AWS Secrets Manager.
2.  **High Availability & Fault Tolerance:** The infrastructure is distributed across multiple Availability Zones (Multi-AZ). This ensures that if one data center goes down, the application remains fully operational.
3.  **Cost Optimization (Serverless & Endpoints):** We heavily utilize serverless compute (AWS Fargate) to avoid paying for idle servers. Furthermore, we implemented VPC Gateway Endpoints to route S3 traffic internally, completely avoiding expensive NAT Gateway data processing fees.
4.  **Asynchronous Processing:** By decoupling the main API server from heavy media processing tasks using Amazon SQS and EventBridge, we guarantee a blazing-fast user experience even during peak traffic.

## The 5-Pillar Architecture Strategy

The infrastructure is logically divided into five distinct pillars, which you will explore in the upcoming sections:

*   **Pillar 1: Networking & Isolation.** The foundation of the system. We established a Virtual Private Cloud (VPC) with Public Subnets for external access (Load Balancers) and Private Subnets to securely hide our compute and database resources.
*   **Pillar 2: Compute & Orchestration.** Utilizing Amazon ECS (Elastic Container Service) with AWS Fargate to run our Docker containers without managing the underlying EC2 instances. 
*   **Pillar 3: State & Persistence.** A combination of Amazon RDS (PostgreSQL) for reliable relational data storage and Amazon ElastiCache (Redis) for high-speed caching and message brokering.
*   **Pillar 4: Storage.** Amazon S3 is used as a highly durable object store for user-uploaded media (images, videos) before they are published to social networks.
*   **Pillar 5: Monitoring & Automation.** Continuous observation via CloudWatch and automated alert routing via SNS to keep the operational team informed of the system's health.

## Why Infrastructure as Code (IaC)?

We chose **Terraform** as our IaC tool for this workshop because of its platform-agnostic nature and powerful state management. By defining our infrastructure in code, we achieve:
*   **Reproducibility:** You can deploy the exact same production environment in minutes, rather than spending hours clicking through the AWS Console.
*   **Disaster Recovery:** If the infrastructure is accidentally deleted, Terraform can rebuild the entire system from scratch with a single command.
*   **Auditability:** Every change to the infrastructure is tracked in Git, making it easy to review security rules and network changes.

### Topics Covered in this Chapter

*   [Infrastructure Layout](5.3.1-infrastructure-layout/)
*   [Architecture Decisions](5.3.2-resource-properties/)
*   [Terraform Modules Breakdown](5.3.3-terraform-modules/)
*   [Deployment Execution](5.3.4-deployment-execution/)