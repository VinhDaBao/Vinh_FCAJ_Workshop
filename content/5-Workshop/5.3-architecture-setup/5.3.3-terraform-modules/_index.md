---
title: "5.3.3. Terraform Modules Reference"
weight: 3
---

The PubliCast infrastructure is built using a highly modular Terraform architecture. Writing infrastructure as code is powerful, but placing all configurations into a single, monolithic `main.tf` file quickly becomes unmanageable and dangerous in a production environment. 

To ensure the system is maintainable, scalable, and secure, we have grouped the individual Terraform configurations into 5 core logical layers defined in our Architecture Layout.

## The Modular Approach

Why did we choose a modular architecture for this project?
1. **Separation of Concerns:** Networking logic shouldn't be tangled with database configurations. Modules allow us to isolate responsibilities.
2. **Blast Radius Reduction:** If a configuration error occurs in the `ci` (Continuous Integration) module, it won't accidentally destroy the `database` module because their definitions are logically separated.
3. **Reusability:** The `compute` module can be reused to spin up both the main API server and the background Workers simply by passing different variables, drastically reducing code duplication.

## Module Dependency Graph

In our setup, modules do not exist in isolation. They form a directed dependency graph where lower-level foundational modules provide necessary outputs (like IDs and ARNs) to higher-level application modules.
*   The **Network** module must be deployed first because all other resources need a VPC and Subnets to live in.
*   The **Database** and **Storage** modules are deployed next.
*   The **Compute** module is deployed last, as the ECS containers require both the Network (to know where to run) and the Database (to connect to PostgreSQL and Redis).

> [!TIP]
> **Source Code Reference**
> Because Terraform files can be quite long, we avoid putting massive code blocks directly into this documentation. Instead, for each module in the following pages, we provide a conceptual explanation of the key `aws_*` resource blocks used. Since you have already cloned the project, you can view the full, raw source code for all these modules by opening the `terraform/modules/` directory in your local IDE.

## How to Read This Reference

For each of the 5 layers detailed in the sidebar, you will find:
*   **Purpose:** A clear explanation of what the module achieves.
*   **Key Terraform Resources:** The specific `aws_*` resources provisioned (e.g., `aws_vpc`, `aws_ecs_cluster`).
*   **Visual Validation:** A screenshot of the resulting AWS Console to help you verify that the Terraform code executed correctly.
