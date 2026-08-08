---
title : "5.4.1. Pipeline Architecture Diagram"
weight : 1
---

## Visualizing the CI/CD Flow

Before diving into the specific Terraform resources that build our pipeline, take a moment to understand the high-level architecture of how code moves from a developer's local machine into the production AWS environment.

As shown below, we utilize a dual-pipeline architecture (one for the Backend ECS services, and one for the Frontend S3/CloudFront application) triggered by a single CodeStar connection to our GitHub monorepo.

{{< img "images/Workshop/services/ci_cd.drawio.png" "CI/CD Pipeline Architecture" >}}
