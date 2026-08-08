---
title: "5.3.3.5. Monitoring & Automation Layer"
weight: 5
---

This layer acts as the brain and eyes of our infrastructure. It ensures the system can be continuously observed, alerted upon, and that deployment or maintenance tasks occur automatically without human intervention.

## CloudWatch (`cloudwatch`)
**Purpose:** A centralized monitoring and observability service. It acts as a giant funnel that collects console logs, performance metrics (like CPU usage), and events from all your AWS resources into one single dashboard.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_cloudwatch_log_group`: Think of a Log Group as a folder where a specific service writes its logs. We create one for each ECS microservice. 
    *   **Configured Attributes:** 
        *   `name`: Creates a distinct path (e.g., `/ecs/publiast-staging`).
        *   `retention_in_days`: We explicitly set this attribute (e.g., to 7 days - 1 week). Without this setting, AWS would keep your logs forever, and your CloudWatch storage costs would accumulate indefinitely.
    
    **💡 Console Analysis:**
    As shown in the image below, when you navigate to **CloudWatch > Log groups**, you will see a list of active Log Groups in your project:
    - **ECS Log group** (`/ecs/publiast-staging`): Configured with a Retention period of **1 week** (7 days) to save costs, and has **1 Metric filter** attached (which we will cover next).
    - **CodeBuild Log groups** (`/aws/codebuild/...`): These are the logs generated during the CI/CD pipeline when building the frontend and backend applications. By default, they are set to `Never expire`.

    {{< img "images/Workshop/services/cloudwatch-logs.png" "AWS Console - CloudWatch Logs" >}}
    <p align="center"><i>AWS Console - CloudWatch Logs</i></p>

    **🔍 Viewing Log Stream Details:**
    From the Log Groups screen, if you click on `/ecs/publiast-staging` and select an active **Log stream** (e.g., `ecs/publicast-api/...`), you will see all the raw log events printed by your application. 
    This confirms that your ECS application is successfully streaming logs to CloudWatch. This stream is exactly where the **Metric Filter** will scan for error keywords (like ERROR, Exception) to trigger alarms.

    {{< img "images/Workshop/services/cloudwatch-log-stream.png" "AWS Console - CloudWatch Log Stream" >}}
    <p align="center"><i>AWS Console - CloudWatch Log Stream</i></p>

*   `aws_cloudwatch_log_metric_filter` & `aws_cloudwatch_metric_alarm`: Creates a filter to scan for error keywords (like ERROR, Exception) in ECS logs. When errors are detected, a Metric Alarm is triggered to notify the system.
    *   **Configured Attributes:**
        *   `pattern`: Keywords to scan (e.g., `?ERROR ?Error ?Exception`).
        *   `threshold` & `evaluation_periods`: Configures the alarm threshold when errors exceed the limit.

    {{< img "images/Workshop/services/cloudwatch-alarm.png" "AWS Console - CloudWatch Alarm" >}}
    <p align="center"><i>AWS Console - CloudWatch Alarm</i></p>

*   `aws_sns_topic` & `aws_sns_topic_subscription`: Channel to receive and distribute alert messages from CloudWatch Alarms.
    *   **Configured Attributes:**
        *   `protocol`: Notification delivery protocol (we use `email`).
        *   `endpoint`: The email address receiving the alerts. You must confirm the subscription via email after running `terraform apply`.

    {{< img "images/Workshop/services/sns-topic.png" "AWS Console - SNS Topic" >}}
    <p align="center"><i>AWS Console - SNS Topic</i></p>
    


📁 **Source Code:** Open `terraform/modules/cloudwatch/main.tf` in your local IDE to view the full configuration.



## CI/CD Pipeline IAM (`ci`)
**Purpose:** Provisions the necessary permissions for the Continuous Integration and Continuous Deployment (CI/CD) system to safely automate deployments without giving it full administrator control.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_iam_user` & `aws_iam_access_key`: Provisions a dedicated, programmatic IAM user for the CI/CD bot.
    *   **Configured Attributes:** 
        *   `name`: Sets the bot's username (e.g., `publicast-ci-bot`).
        *   `aws_iam_access_key` generates an Access Key ID and Secret Access Key. *(Note: Though our project uses AWS Native CodePipeline which doesn't strictly need static keys, this resource acts as a fallback or is used if you prefer external CI runners like GitHub Actions).*
    
    {{< img "images/Workshop/services/iam-user.png" "AWS Console - CI IAM User" >}}
    <p align="center"><i>AWS Console - CI IAM User</i></p>
    
*   `aws_iam_policy`: We craft a strict JSON policy that *only* grants the permissions required for deployment. 
    *   **Configured Attributes:** 
        *   `policy`: Defines the allowed actions. The bot is only allowed to do things like `ecr:GetAuthorizationToken` (login to Docker), `ecr:BatchPushImage` (upload new code), and `ecs:UpdateService` (tell ECS to use the new code). By doing this, even if the CI/CD Keys are stolen by a hacker, the attacker cannot delete the database or modify the VPC, keeping the blast radius extremely small.
    
    {{< img "images/Workshop/services/iam-ci-policy.png" "AWS Console - CI IAM Policy" >}}
    <p align="center"><i>AWS Console - CI IAM Policy</i></p>

📁 **Source Code:** Open `terraform/modules/ci/main.tf` in your local IDE to view the full configuration.
