---
title : "5.4.3. GitHub Source Connection"
weight : 3
---

In order for AWS CodePipeline to automatically pull your latest code whenever you push to GitHub, AWS needs secure permission to access your GitHub repository.

We deploy this integration using the AWS CodeStar Connections service via Terraform:

```hcl
# ------------------------------------------------------------------------------
# 2. CODESTAR CONNECTION (GITHUB INTEGRATION)
# ------------------------------------------------------------------------------
resource "aws_codestarconnections_connection" "github" {
  name          = "${var.project}-${var.environment}-github-conn"
  provider_type = "GitHub"
}
```

## The "Pending" State Caveat

When you run `terraform apply`, Terraform successfully creates the connection resource in your AWS Account. **However, Terraform cannot automatically authorize AWS to read your personal GitHub account.** This requires an interactive OAuth login flow in your web browser.

Because of this security requirement, right after Terraform finishes deploying, the connection will be stuck in a **Pending** state. Your CodePipeline will fail to trigger until you manually approve this connection.

## How to Authorize the Connection

Follow these steps immediately after running `terraform apply`:

1.  Log in to the AWS Management Console and search for **Developer Tools**.
2.  On the left navigation pane, under **Settings**, click on **Connections**.
3.  You will see the connection created by Terraform (e.g., `publicast-workshop-github-conn`). The status will show as **Pending**.
    
    {{< img "images/Workshop/services/github-pending.png" "AWS Console - Pending Connection" >}}
    <p align="center"><i>Initial Pending Status</i></p>

4.  Select the connection and click the **Update pending connection** button.
5.  You will be redirected to the **Connect to GitHub** page. Under **App Installation**, select your existing AWS app installation (or click **Install a new app** if this is your first time). Finally, click the orange **Connect** button.
    
    {{< img "images/Workshop/services/github-update.png" "AWS Console - Connect to GitHub" >}}
    <p align="center"><i>Updating the Connection (Connect to GitHub)</i></p>

    > [!NOTE]
    > **GitHub Security (Sudo mode):** If you encounter a **"Confirm access"** screen, this is a standard GitHub security measure. Click **Verify via email**, retrieve the 6-digit code from your inbox, and enter it to proceed.

6.  Click **Authorize AWS Connector for GitHub**. You will be prompted to install the AWS app on your specific repository (ensure you select your cloned PubliCast monorepo).
    
    {{< img "images/Workshop/services/github-authorize.png" "GitHub - Authorize AWS Connector" >}}
    <p align="center"><i>Authorizing AWS Connector on GitHub</i></p>

7.  Once completed, the status in the AWS Console will change from *Pending* to **Available**.
    
    {{< img "images/Workshop/services/github-available.png" "AWS Console - Available Connection" >}}
    <p align="center"><i>Connection Successfully Available</i></p>

> [!IMPORTANT]
> Your CI/CD pipelines are now fully linked to your GitHub repository. Any future `git push` to your configured branch will automatically trigger the CodePipeline builds!
