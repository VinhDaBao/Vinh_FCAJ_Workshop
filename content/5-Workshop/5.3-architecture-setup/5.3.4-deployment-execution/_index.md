---
title: "5.3.4. Deployment Execution"
weight: 4
---

Now that you understand the infrastructure layout and the underlying Terraform modules, it's time to actually deploy the AWS resources!

## Step 1: Configure AWS Account (AWS Configure)

Before Terraform can interact with and provision actual resources on AWS, your computer (or command-line environment) must be authenticated using a set of security credentials. This configuration ensures proper authentication and authorization for the automated deployment process.

Here are the detailed steps:

1. **Access AWS IAM:** Log in to the [AWS Management Console](https://console.aws.amazon.com/). In the search bar at the top of the screen, type **IAM** (Identity and Access Management) and select this service.
   {{< img "images/Workshop/services/iam-search.png" "Search IAM" >}}
2. **Create User:** On the left menu, select **Users** and click the **Create user** button. Name the user (e.g., `terraform-admin`) and click Next.
   {{< img "images/Workshop/services/iam-create-user.png" "Create IAM User" >}}
3. **Grant Administrator Permissions:** At the Set permissions step, choose **Attach policies directly**. Search for and check the **AdministratorAccess** box (highest privilege) to ensure Terraform can create all necessary resources for PubliCast, then click Create user.
   {{< img "images/Workshop/services/iam-attach-policy.png" "Attach AdministratorAccess Policy" >}}
   > [!WARNING]
   > Granting `AdministratorAccess` here is solely for the convenience of this hands-on lab, ensuring Terraform can provision all resources without encountering permission denied errors. In a real-world production environment, you must adhere to the Principle of Least Privilege by configuring stricter policies that grant Terraform only the precise permissions it needs.
4. **Generate Access Key:** Click on the newly created `terraform-admin` username.
   {{< img "images/Workshop/services/iam-user-created.png" "User Created Successfully" >}}
   
   Navigate to the **Security credentials** tab. Scroll down to the Access keys section and click **Create access key**.
   {{< img "images/Workshop/services/iam-security-credentials.png" "Security Credentials Tab" >}}
   
   At Step 1 (Access key best practices & alternatives), select **Command Line Interface (CLI)** as the use case, acknowledge the warning at the bottom, and click Next.
   {{< img "images/Workshop/services/iam-access-key-cli.png" "Select CLI Use Case" >}}
   
   At Step 2 (Set description tag), you can optionally set a description (e.g., `terraform`) and click **Create access key**.
   {{< img "images/Workshop/services/iam-access-key-tag.png" "Set Access Key Tag" >}}
   
   At Step 3 (Retrieve access keys), the screen will display the `Access key` and `Secret access key`. This is the only time you can view the Secret Key, so copy it and click **Done** to finish.
   {{< img "images/Workshop/services/iam-access-key-retrieve.png" "Retrieve Access Keys" >}}
5. **Initialize AWS CLI Configuration:** Ensure you have the AWS CLI installed on your machine. Open your terminal (or Command Prompt/PowerShell) and run the initialization command:
   ```bash
   aws configure
   ```
   {{< img "images/Workshop/services/aws-configure-cmd.png" "Run aws configure" >}}
6. **Input Connection Details:** The terminal will prompt you to enter 4 parameters. Fill them in carefully:
   * **AWS Access Key ID**: Paste the Key ID you just generated in step 4 (e.g., `AKIAIOSFODNN7EXAMPLE`).
   * **AWS Secret Access Key**: Paste the corresponding Secret Key (this is only shown once).
   * **Default region name**: Enter the AWS region code where you want to deploy the project (e.g., `ap-southeast-1` for Singapore, or `us-east-1` for N. Virginia).
   * **Default output format**: Enter `json` to make the AWS CLI output more readable.
   
   {{< img "images/Workshop/services/aws-configure-inputs.png" "Enter AWS Credentials" >}}

## Step 2: Configure Environment Variables

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

## Step 3: Initialize Terraform

Initialize the Terraform working directory. This command downloads the necessary AWS provider plugins.

```bash
terraform init
```

## Step 4: Preview Changes

Generate an execution plan. This allows you to safely preview exactly what AWS resources Terraform will create before it actually creates them.

```bash
terraform plan
```

## Step 5: Apply Configuration

Once you are satisfied with the plan, execute it to provision the infrastructure.

```bash
terraform apply
```

At this point, Terraform will list all the changes one last time and pause to ask if you are sure you want to perform these actions. Type `yes` and press Enter to approve.

{{< img "images/Workshop/services/terraform-apply-yes.png" "Terminal - Terraform Apply Confirm" >}}

> [!NOTE]
> The deployment process takes approximately 10-15 minutes, primarily because provisioning the RDS Database and ElastiCache Redis cluster takes time. Grab a coffee and wait for the `Apply complete!` message in your terminal.

{{< img "images/Workshop/services/terraform-apply-complete.png" "Terminal - Terraform Apply Complete" >}}

## Step 6: Test & Validation

Once the system is successfully deployed, you need to validate that everything works as expected:

1. **Access the application:** Open your browser and navigate to your domain (e.g., `https://publicast.yourdomain.com`).
   {{< img "images/Workshop/services/test-access-app.png" "Access Application" >}}
2. **Send a Request:** Try logging in, creating a new Worklog post, or uploading an image.
   {{< img "images/Workshop/services/test-send-request.png" "Send Request" >}}
3. **View System Logs:** Go to the AWS Console > **CloudWatch** > **Log groups**. Locate `/ecs/publicast-staging` and check the log stream to verify that your requests are being recorded.
   {{< img "images/Workshop/services/test-cloudwatch-logs.png" "CloudWatch Logs" >}}
4. **Test CloudWatch Metrics & Alerts:** The system is configured to automatically scan logs for errors and trigger alerts. To test this flow, follow these steps:
   * **Trigger an Error:** Intentionally call a non-existent API (e.g., `https://api.publicast.yourdomain.com/v1/invalid-endpoint`) or enter incorrect login credentials multiple times to force the backend to generate logs containing the `ERROR` or `Exception` keywords.
   * **Check Metric Filter:** Go to **CloudWatch** > **Log groups** > select `/ecs/publicast-staging`. Switch to the **Metric filters** tab, where you will see the error keyword counter increase.
   * **Observe Alarm State:** Navigate to **All alarms** on the left menu. Locate the alarm named `publicast-staging-error-alarm`. Its state will change from green (OK) to red (In alarm) because the error count exceeded the configured threshold within the evaluation period.
   * **Verify SNS Email Alert:** Check the inbox of the email address you registered for notifications (confirmed during the previous setup step). You should receive an automated email from AWS Notifications detailing the incident, which allows the operations team to respond promptly.
   
   {{< img "images/Workshop/services/test-cloudwatch-alarm.png" "CloudWatch Alarm" >}}
