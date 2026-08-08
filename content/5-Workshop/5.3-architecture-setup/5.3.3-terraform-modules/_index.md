---
title: "Terraform Modules Reference"
weight: 3
---

The PubliCast infrastructure is built using a highly modular Terraform architecture. To understand how the system is deployed, we have grouped the individual Terraform modules into the 5 core logical layers defined in our Architecture Layout. 

> [!TIP]
> **Source Code Reference**
> Because Terraform files can be quite long, we avoid putting large code screenshots here. Instead, for each module below, we provide a conceptual explanation of the key `aws_*` resource blocks used. Since you have already cloned the project, you can view the full, raw source code for all these modules by opening the `terraform/modules/` directory in your local code editor.

For each service deployed, you will find an explanation of its purpose, an overview of the core Terraform code structure, and a screenshot of the resulting AWS Console to verify its creation.
