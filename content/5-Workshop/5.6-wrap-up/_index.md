---
title: "5.6. Wrap-up"
weight: 6
---

Congratulations on completing the workshop! Here are the core achievements and key lessons you've learned through deploying the PubliCast project.

## Summary of achievements

*   **Production-grade Multi-AZ AWS Infrastructure**: You successfully built a secure, highly available network architecture with Public and Private Subnets spanning multiple Availability Zones.
*   **Zero-NAT cost for S3 storage**: By successfully implementing a VPC Gateway Endpoint for S3, the project entirely eliminated data processing costs through the NAT Gateway for large media upload/download tasks, saving significant budget.
*   **Microservice isolation**: Decoupling the background processing (Worker Light/Heavy) from the main API application keeps the system stable and flexible in scaling compute resources based on workload characteristics.

## Key lessons learned

*   **Infrastructure as Code (IaC) with Terraform**: Understanding the power of consistently managing, automating, and reusing infrastructure configurations through code.
*   **Cloud Cost Optimization Strategy**: Costs are not just about what services you use, but how those services communicate with each other (like the NAT Gateway vs VPC Endpoint lesson).
*   **Architectural Design Thinking**: Applying the Queue-Worker pattern is key to building high-load web applications involving heavy media processing.

## Personal Contributions & Customizations (Custom Features)

In the PubliCast project, to meet the real-world demands of a social media content management platform, I went beyond the standard architectural template by proactively designing and developing the following advanced features and services:

1. **Advanced Secrets Management**
   * **Context:** The application interacts with numerous third-party API Keys (Meta, Google, TikTok, Resend) and database credentials.
   * **Implementation:** Instead of using traditional Environment Variables, which are vulnerable to memory dumps or accidental `.env` leaks, I integrated **AWS Secrets Manager**. This service encrypts sensitive data in a centralized repository. ECS containers are only authorized to fetch tokens at runtime via a strict IAM Role.
   * **Result:** Significantly elevated the security posture of the entire system, meeting safety standards for handling OAuth credentials.

2. **High-Performance Message Broker for Queue Optimization**
   * **Context:** A core functionality of PubliCast is uploading, processing, and publishing large video/image files to social networks. Synchronous API processing would cause severe latency for end users.
   * **Implementation:** I engineered a robust Queue-Worker system by introducing **Amazon ElastiCache (Redis)** as the message broker, combined with the BullMQ library. Workloads are decoupled into "Worker Light" (handling emails/notifications) and "Worker Heavy" (dedicated to video streaming).
   * **Result:** Ensures the main API system remains entirely non-blocking, effortlessly scaling even when thousands of scheduled posts are triggered simultaneously.

3. **Automated Log Monitoring & Alerting**
   * **Context:** Detecting anomalies in a microservices environment is highly inefficient if done manually across containers.
   * **Implementation:** I proactively configured **CloudWatch Metric Filters** to perform real-time scanning of ECS Log streams. Whenever the system outputs keywords like "ERROR" or "Exception", a Metric Alarm is instantly triggered.
   * **Result:** The system automatically pushes alerts via email using Amazon SNS, empowering the operations team to react almost immediately before users even notice a failure.

4. **Hybrid Architecture for Scheduled Posts**
   * **Context:** Scheduling thousands of posts by soaking Delayed Jobs in Redis RAM wastes resources and risks failure if RAM overloads. Furthermore, aggressively calling social media APIs simultaneously can lead to app suspension due to Rate Limiting violations.
   * **Implementation:** I designed a hybrid solution combining **Amazon EventBridge Scheduler** and **Amazon SQS**. EventBridge serves as an ultra-low-cost "Alarm Clock". At the scheduled time, it triggers an event and pushes a task into the SQS queue. The Worker then pulls tasks from SQS and utilizes BullMQ as a "Job Manager" (Rate Limiter) to strictly control the pace of publishing API calls (e.g., 5 requests/second).
   * **Result:** Radically optimized server costs, completely freed up Redis RAM, and guaranteed absolute safety when interacting with third-party social media APIs.

## Reflection

### Challenges Faced
*   **Complex IAM Rights Management:** Setting up security policies following the Least Privilege principle for the CI/CD CodePipeline and ECS containers posed a significant challenge. Initially, the system frequently encountered "Access Denied" errors due to missing permissions for underlying services, particularly when containers attempted to fetch API keys from AWS Secrets Manager during startup.
*   **Internal Network Routing:** Integrating the VPC Gateway Endpoint for S3 to save bandwidth costs requires extremely precise Route Table configurations. During the first deployment, incorrect routing rules caused containers in the Private Subnet to be isolated, unable to push video/image files to S3, leading to timeouts across the entire publishing process.

### Solutions
*   **Advanced Debugging via CloudWatch:** Instead of taking the easy route and granting `AdministratorAccess` to everything, I honed my system log reading skills. By actively monitoring logs from CodeBuild and ECS CloudWatch Log streams, I could precisely trace the exact action denied by AWS, and then manually and accurately add those permissions into the Terraform configuration files. This ensured the project strictly adhered to enterprise security standards.
*   **Reviewing and Visualizing Network Topology:** I thoroughly rechecked the association between the Private Subnet's Route Table and the S3 Gateway Endpoint. By adjusting the routing rules, I ensured that all internal traffic calling to S3 was correctly and securely routed through the internal Endpoint rather than getting stuck at the NAT Gateway.

### Future Work
*   **Dynamic Auto-scaling:** Currently, the system only scales resources based on a static %CPU threshold. In the future, I plan to configure Application Auto Scaling to automatically increase the number of "Worker Heavy" containers based on the number of pending tasks (messages) in the SQS queue. This will help the system react more intelligently during peak traffic hours.
*   **Advanced CDN Deployment:** I aim to optimize Amazon CloudFront caching with more complex Behaviors, specifically designing dedicated caching rules for heavy video streams to smoothly serve users across various geographical regions.

