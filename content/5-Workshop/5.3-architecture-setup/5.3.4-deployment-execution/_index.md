---
title: "5.3.4. Deployment Execution"
weight: 4
---

Now that you understand the infrastructure layout and the underlying Terraform modules, it's time to actually deploy the AWS resources!

## Step 1: Configure Environment Variables

Before running any Terraform commands, you need to provide your own specific values. Because the PubliCast platform integrates with numerous third-party services, your `terraform.tfvars` file will act as the central registry for all your API keys and passwords. 

We must **never** hardcode these sensitive values into the version control system.

1. Navigate to the `terraform` directory in your terminal:
   ```bash
   cd terraform
   ```
2. Copy the example variables file to create your own local `terraform.tfvars` file:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```
3. Open `terraform.tfvars` in your code editor and fill in your actual values. The variables are grouped logically:
   
   *   **Core Security & Databases (`db_password`, `jwt_secret`, `encryption_key`, etc.)**: Generate strong, random strings for these values. They are used internally for database access and encrypting user sessions.
   *   **AWS Config (`acm_certificate_arn`, `domain_name`, `frontend_bucket_name`)**: Paste the ARN of your SSL certificate created in step 5.2, your registered domain name, and choose a globally unique name for your S3 bucket.
   *   **Social OAuth & Publishing APIs**: PubliCast acts as a social media hub. You need to register your app on these platforms to get client IDs and secrets:
       *   **Google/YouTube**: Go to the [Google Cloud Console](https://console.cloud.google.com/), enable the YouTube Data API v3, and generate OAuth 2.0 credentials.
       *   **Meta (Facebook, Instagram, Threads)**: Go to [Meta for Developers](https://developers.facebook.com/), create a Business App, and add the Facebook Login, Instagram Graph API, and Threads API products.
       *   **TikTok**: Register at the [TikTok for Developers](https://developers.tiktok.com/) portal.
       *   **Discord**: Create an application in the [Discord Developer Portal](https://discord.com/developers/applications) to get the Client ID and Bot Token.
   *   **Third-Party Integrations**:
       *   **Payments (VietQR/SePay)**: Enter your banking details (`vietqr_account_no`) and your [SePay API Key](https://sepay.vn/) for automated transaction verification.
       *   **Emails (Resend)**: Get your API key from [Resend](https://resend.com/).
       *   **AI Generators**: Obtain keys from [OpenAI](https://platform.openai.com/) (ChatGPT) and [Google AI Studio](https://aistudio.google.com/) (Gemini) for the AI content generation features.
   *   **CI/CD Pipeline (`github_monorepo`, `github_branch`)**: Enter your GitHub repository string (e.g., `username/publicast`) and the branch you want CodePipeline to watch (e.g., `main`).

> [!WARNING]
> **Keep your keys safe!** 
> The `terraform.tfvars` file is automatically ignored by `.gitignore`. **Never** commit your actual `terraform.tfvars` file to GitHub, as it contains highly sensitive credentials that hackers can exploit.

## Step 2: Initialize Terraform

Initialize the Terraform working directory. This command downloads the necessary AWS provider plugins.

```bash
terraform init
```

## Step 3: Preview Changes

Generate an execution plan. This allows you to safely preview exactly what AWS resources Terraform will create before it actually creates them.

```bash
terraform plan
```

## Step 4: Apply Configuration

Once you are satisfied with the plan, execute it to provision the infrastructure.

```bash
terraform apply
```

At this point, Terraform will list all the changes one last time and pause to ask if you are sure you want to perform these actions. Type `yes` and press Enter to approve.

{{< img "images/Workshop/services/terraform-apply-yes.png" "Terminal - Terraform Apply Confirm" >}}

> [!NOTE]
> The deployment process takes approximately 10-15 minutes, primarily because provisioning the RDS Database and ElastiCache Redis cluster takes time. Grab a coffee and wait for the `Apply complete!` message in your terminal.

{{< img "images/Workshop/services/terraform-apply-complete.png" "Terminal - Terraform Apply Complete" >}}
