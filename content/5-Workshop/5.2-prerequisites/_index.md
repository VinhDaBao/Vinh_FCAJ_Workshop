---
title: "5.2. Prerequisites"
weight: 2
---

Before deploying the infrastructure for the PubliCast project, ensure that the following tools are properly installed and configured on your machine:

## Pre-required tools

1.  **AWS CLI**: Installed and configured with your AWS credentials (Access Key ID & Secret Access Key). Ensure the IAM user has sufficient permissions to create resources like VPC, ECS, RDS, S3, etc.
2.  **Terraform**: Version >= 1.5.0. Used to deploy Infrastructure as Code (IaC).
3.  **Docker**: To build and test local images if necessary.
4.  **Git**: For source code management.

## Instructions to clone the project repository

Clone the source code containing the project's Terraform configuration to your local machine using the following commands:

```bash
git clone https://github.com/Nguyen-Thanh-Huy-io/FCAJ-AWS-Project.git
cd publicast-terraform/terraform/envs/staging
```

*Note: Replace `https://github.com/your-username/publicast-terraform.git` with your actual repository URL.*

## Generate SSL/TLS Certificate for CloudFront via ACM (Mandatory)

To enable HTTPS access via your custom domain `publicast.trinhquoccongvinh.id.vn`, the system relies on **AWS Certificate Manager (ACM)** to provision and manage SSL/TLS certificates. This is a mandatory prerequisite before running Terraform.

### Step 1: Identify Target Domain

> [!NOTE]
> In this guide, `trinhquoccongvinh.id.vn` is used as an example domain. When performing this lab, you **MUST** replace it with your own custom domain that you own or have purchased.

The root domain managed is: `trinhquoccongvinh.id.vn` *(replace with your domain)*
The system utilizes a subdomain for the application: `publicast.trinhquoccongvinh.id.vn` *(replace with your subdomain)*

Because CloudFront requires the certificate to match the Alternate Domain Name (CNAME) exactly, the certificate must be issued to: `publicast.trinhquoccongvinh.id.vn`

### Step 2: Request Certificate in AWS Certificate Manager

Access **AWS Certificate Manager (ACM)** and switch your region to: `US East (N. Virginia) – us-east-1`
> [!IMPORTANT]
> This is a strict requirement when using ACM certificates with Amazon CloudFront.

In ACM, select **Request a certificate** from the dashboard.
{{< img "images/Workshop/services/acm-dashboard.png" "ACM Dashboard" >}}

Choose the certificate type: **Request a public certificate**
{{< img "images/Workshop/services/acm-cert-type.png" "Request a public certificate" >}}

Then input the following details:
* Fully qualified domain name: `publicast.trinhquoccongvinh.id.vn`
* Validation method: DNS validation - recommended

Click **Request** to submit the certificate request.
{{< img "images/Workshop/services/acm-request-details.png" "ACM Request Details" >}}

### Step 3: DNS Validation

Upon requesting the certificate, ACM provides a DNS validation CNAME record.
From the Certificate details screen, you will see the status is **Pending validation** along with the CNAME information.
{{< img "images/Workshop/services/acm-pending-validation.png" "ACM Pending Validation" >}}

**How to automatically add it to Route 53:**
Since we are managing the domain using AWS Route 53, you simply need to click the **Create records in Route 53** button.
A confirmation screen will appear, click **Create records** again. AWS will automatically attach this CNAME record to your Hosted Zone to verify domain ownership.
{{< img "images/Workshop/services/acm-create-records.png" "Create Route 53 Records" >}}

To be absolutely sure, you can navigate to the **Route 53 > Hosted zones** service and view the list of Records. You will see the new CNAME record that was just automatically added.
{{< img "images/Workshop/services/route53-acm-record.png" "Route 53 ACM Record" >}}

### Step 4: Verify Certificate Status

The certificate can only be used for CloudFront once its status changes to **`ISSUED`** (Green color). The DNS validation process typically takes a few minutes. Please refresh the page to check the status.

### Step 5: Configure Certificate for CloudFront

In your Terraform variables, the certificate ARN is passed as:
```hcl
acm_certificate_arn = "arn:aws:acm:us-east-1:ACCOUNT_ID:certificate/..."
```

The CloudFront Distribution consumes this certificate via:
```hcl
viewer_certificate {
  acm_certificate_arn      = var.acm_certificate_arn
  ssl_support_method       = "sni-only"
  minimum_protocol_version = "TLSv1.2_2021"
}
```

The Alternate Domain Name is configured as:
```hcl
aliases = [
  var.domain_name
]
```
where `domain_name = "publicast.trinhquoccongvinh.id.vn"`

### Step 6: Outcome

Once the certificate is issued and CloudFront is fully provisioned, users can securely access the application via: `https://publicast.trinhquoccongvinh.id.vn`

> [!WARNING]
> **Important Note:** Certificates intended for CloudFront **must be requested in the `us-east-1` region**, even if all other backend resources (VPC, ECS, RDS, etc.) are deployed in a different region like `ap-southeast-2`.
> 
> Furthermore, the certificate must exactly cover the domain configured in CloudFront. A certificate for `trinhquoccongvinh.id.vn` cannot be used for `publicast.trinhquoccongvinh.id.vn` unless the subdomain is explicitly included in the Subject Alternative Names.