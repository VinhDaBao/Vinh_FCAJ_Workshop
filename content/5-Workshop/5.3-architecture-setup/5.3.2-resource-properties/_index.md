---
title: "5.3.2. Architecture Decisions"
weight: 2
---

This section explains the rationale behind the key technology choices made when designing the PubliCast AWS infrastructure. Understanding *why* a service was chosen in the context of our specific business requirements is just as important as knowing *how* to deploy it.

PubliCast is a cross-platform social media publishing tool. Our architecture must handle high-resolution media, secure third-party credentials, and process unpredictable workloads. Here is how AWS services map to those requirements.

## 1. Why Amazon S3 and CloudFront? (Media Storage & Delivery)
**The Business Need:** Users continuously upload large video files and high-resolution images destined for platforms like YouTube, Facebook, and Instagram.
**The Technical Choice:** 
*   **Amazon S3:** We chose S3 because it provides virtually infinite scalability for massive binary blobs. It is far more cost-effective than storing media on block storage (EBS) or inside a relational database.
*   **Amazon CloudFront:** By putting a CDN in front of S3, we ensure that users across different geographic regions experience ultra-low latency when previewing or uploading their content, significantly improving the user experience.

## 2. Why AWS Secrets Manager? (Secure Credentials)
**The Business Need:** To publish content on behalf of users, PubliCast must manage and utilize highly sensitive third-party API keys, OAuth tokens (Meta, Google), and database passwords.
**The Technical Choice:** 
Hardcoding these credentials in source code or standard environment variables is a major security risk. We integrated **AWS Secrets Manager** to encrypt and store these secrets centrally. At runtime, the ECS tasks fetch these secrets securely using IAM roles, ensuring strict compliance and zero credential leakage.

## 3. Why decouple into 3 Microservices? (Workload Isolation)
**The Business Need:** Uploading and streaming heavy video files to YouTube takes time and consumes massive CPU/RAM. If handled synchronously, it would freeze the web application for users trying to do simple tasks.
**The Technical Choice:** 
We decoupled the monolith into three specialized microservices:
*   **API Service:** Handles lightweight, real-time HTTP requests (e.g., loading the dashboard). Requires low CPU but fast response times.
*   **Worker Light:** Handles quick background tasks like sending email notifications.
*   **Worker Heavy:** Exclusively dedicated to resource-intensive tasks like video encoding and streaming payloads to social networks. 
*   **Result:** This isolation guarantees that a massive spike in video publishing jobs will never crash or slow down the main user interface.

## 4. Why ECS on AWS Fargate? (Serverless Compute)
**The Business Need:** Social media publishing traffic is often spiky (e.g., thousands of users scheduling posts for exactly 8:00 AM on Monday).
**The Technical Choice:** 
Instead of provisioning traditional EC2 virtual machines, we chose **ECS on AWS Fargate**. Fargate eliminates the need to patch or manage operating systems. More importantly, it allows our microservices to auto-scale rapidly and independently based on real-time demand, ensuring we only pay for the exact compute power we use.

## 5. Why Amazon RDS and ElastiCache? (Data & Job Queues)
**The Business Need:** The system needs to reliably store structured user data (Workspaces, Post Histories) and manage thousands of scheduled publishing jobs without dropping any.
**The Technical Choice:** 
*   **Amazon RDS (MySQL):** Provides highly available, relational storage with automated daily backups, ensuring core user data is never lost.
*   **Amazon ElastiCache (Redis):** Acts as a lightning-fast message broker powering our BullMQ job queues. It ensures that every scheduled post is reliably picked up by a Worker, processed in order, and never duplicated, while also caching heavy database queries to speed up the API.
