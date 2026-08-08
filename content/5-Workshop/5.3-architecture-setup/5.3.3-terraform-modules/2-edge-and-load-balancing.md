---
title: "5.3.3.2. Edge & Load Balancing Layer"
weight: 2
---

This layer acts as the front door to the PubliCast application. It is responsible for receiving all global traffic from the internet, filtering it, and routing it efficiently to our hidden backend containers. Think of this layer as the receptionist desk and the high-speed elevators of our virtual building.

## Route53 (`route53`)
**Purpose:** Acts as a highly available Domain Name System (DNS) service to translate human-readable names (like `api.publicast.app`) into machine IP addresses or AWS endpoints. Without Route53, users would have to type complex AWS URLs to reach our app.

> [!IMPORTANT]
> **Use Your Own Domain**
> If you are following this workshop, make sure to replace values like `api.publicast.app` with your own registered domain name in your Terraform variables.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_route53_zone`: Manages the "Hosted Zone" for our custom domain. A Hosted Zone is essentially a container that holds all the DNS records (routing rules) for a specific domain name.
    *   **Configured Attributes:** 
        *   `name`: Your root domain name (e.g., `publicast.app`).
    
    {{< img "images/Workshop/services/route53-zone.png" "AWS Console - Route53 Hosted Zone" >}}
    <p align="center"><i>AWS Console - Route53 Hosted Zone</i></p>
    
*   `aws_route53_record`: Route53 is not just for our internal AWS routing; it is also crucial for verifying our domain with third-party social media and email platforms. As seen in the console, we define multiple types of records:
    *   **A Records (Alias):** Used to map our domain directly to the dynamic DNS names of our Application Load Balancer and CloudFront CDN. We use a special AWS `alias` block instead of hardcoding IP addresses, ensuring that if our Load Balancer automatically scales and changes its IP, the DNS routing won't break.
    *   **TXT Records (Social Media Integration):** Used for domain ownership verification. For example, we add a specific TXT record containing a `tiktok-developers-site-verification=...` string to prove to TikTok that we own this domain before they grant us access to their OAuth APIs.
    *   **MX & TXT Records (Email Security):** To ensure emails sent from our application (via Resend or AWS SES) don't go to user spam folders, we configure `MX` records for mail routing, and specific `TXT` records (like `_domainkey` and `_dmarc`) for strict SPF, DKIM, and DMARC email authentication policies.
    
    {{< img "images/Workshop/services/route53-record.png" "AWS Console - Route53 Record" >}}
    <p align="center"><i>AWS Console - Route53 Record</i></p>

📁 **Source Code:** Open `terraform/modules/route53/main.tf` in your local IDE to view the full configuration.

## Application Load Balancer (`alb`)
**Purpose:** Distributes incoming application traffic across multiple targets (our ECS containers) spreading across multiple Availability Zones. If one container is overwhelmed with requests or crashes, the ALB seamlessly redirects new traffic to healthy containers.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_lb`: This is the load balancer itself. It is explicitly designed to face the internet.
    *   **Configured Attributes:** 
        *   `load_balancer_type = "application"`: Tells AWS it needs to understand HTTP/HTTPS traffic natively (Layer 7).
        *   `internal = false`: Ensures the ALB has public IP addresses.
        *   `subnets`: Binds the ALB to our Public Subnets so it can be reached from the outside world.
        *   `security_groups`: Attaches the specific `alb-sg` firewall.
    
    {{< img "images/Workshop/services/alb.png" "AWS Console - Application Load Balancer" >}}
    <p align="center"><i>AWS Console - Application Load Balancer</i></p>
    
*   `aws_lb_target_group`: A Target Group tells the load balancer *where* to send the traffic. Because we are using AWS Fargate (serverless containers), there are no traditional EC2 instances to register. Fargate tasks are assigned Elastic Network Interfaces (ENIs) with private IP addresses.
    
    {{< img "images/Workshop/services/tg.png" "AWS Console - Target Groups List" >}}
    <p align="center"><i>AWS Console - Target Groups List</i></p>

    *   **Inside the Target Group Details:** As seen in the details image below, our target group actively registers two healthy targets on port 3000, safely spanning two different Availability Zones for high availability.
    *   **Configured Attributes:** 
        *   `target_type = "ip"`: Crucial setting for Fargate. As shown in the "Registered targets" tab, the ALB routes directly to the private container IPs (e.g., `10.0.x.x`), not instance IDs.
        *   `vpc_id`, `port`, `protocol`: Defines the base network configuration (HTTP on port 3000, where our Node.js app runs).
        *   `health_check` block: Configures the URL path (e.g., `/health`), interval, and timeout. The ALB constantly pings this path to verify if a container is healthy before sending it any traffic.
    
    {{< img "images/Workshop/services/tg-detail.png" "AWS Console - Target Group Details" >}}
    <p align="center"><i>AWS Console - Target Group Details</i></p>
    
*   `aws_lb_listener`: Listeners are processes that check for connection requests. As seen in the console, we create listeners for both HTTP (Port 80) and HTTPS (Port 443).
    *   **Inside the Listener Details & Rules:** 
    *   **Configured Attributes:** 
        *   `port` & `protocol`: Defines the listening ports (80 or 443).
        *   `certificate_arn` (for HTTPS): Connects the listener to an AWS Certificate Manager (ACM) SSL certificate (e.g., `*.domain.com`) for secure encryption.
        *   `default_action` block: For both listeners, we set `type = "forward"` and provide the `target_group_arn` to actively send incoming traffic to our ECS containers via the target group.
    
    {{< img "images/Workshop/services/listener.png" "AWS Console - ALB Listener Details & Rules" >}}
    <p align="center"><i>AWS Console - ALB Listener Details & Rules</i></p>

📁 **Source Code:** Open `terraform/modules/alb/main.tf` in your local IDE to view the full configuration.

## CloudFront CDN (`s3_cloudfront`)
**Purpose:** A Content Delivery Network (CDN) that dramatically speeds up the distribution of your static media (videos, images). It achieves this by caching copies of your files at edge locations globally. When a user in Europe requests an image, CloudFront serves it from a server in Europe, rather than fetching it all the way from our main server in the US.

**Key Terraform Resources:** 
**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_cloudfront_distribution`: This is the core CDN configuration. As seen in the console, our distribution is configured as a "unified gateway" handling different types of traffic through a single domain. We achieve this using multiple Origins and Behaviors.
    
    {{< img "images/Workshop/services/cloudfront-dist.png" "AWS Console - CloudFront Distribution" >}}
    <p align="center"><i>AWS Console - CloudFront Distribution</i></p>

    *   **Multiple `origin` blocks:** *(What is an Origin? An origin is the original source of your content, such as a backend server or an S3 bucket, where CloudFront fetches the data from).* We define distinct origins for our different architectural components. Let's analyze what each origin does:
        1.  `s3-origin`: Points to the S3 bucket hosting our Frontend compiled static assets (HTML, CSS, JS).
        2.  `alb-origin`: Points to our Application Load Balancer. This is the entry point for all dynamic backend API requests that need to be processed by our Node.js containers.
        3.  `backend-storage-origin`: Points to a separate S3 bucket dedicated to storing user-uploaded media (videos, profile pictures, thumbnails).
    
    {{< img "images/Workshop/services/cloudfront-origins.png" "AWS Console - CloudFront Origins" >}}
    <p align="center"><i>AWS Console - CloudFront Origins</i></p>

    *   **`default_cache_behavior` & `ordered_cache_behavior` blocks:** *(What is a Behavior? A cache behavior is a set of routing rules telling CloudFront exactly how to process requests based on the URL path).* These behaviors dictate exactly how the CDN routes traffic based on the URL path. As shown in the Behaviors tab:
        *   `/api/*` (Precedence 0): Routes backend API calls to the `alb-origin`.
        *   `/media/*` (Precedence 1): Routes media requests to the `backend-storage-origin`.
        *   `Default (*)` (Precedence 2): Routes all other frontend traffic to the `s3-origin`.
        *   We enforce `viewer_protocol_policy = "redirect-to-https"` across all behaviors for strict security.
        *   `viewer_certificate`: Configures the SSL certificate if using a custom CDN domain.
    
    {{< img "images/Workshop/services/cloudfront-behaviors.png" "AWS Console - CloudFront Behaviors" >}}
    <p align="center"><i>AWS Console - CloudFront Behaviors</i></p>
    
*   `aws_cloudfront_origin_access_control` (OAC): *(What is OAC? Origin Access Control is a security feature that ensures your S3 buckets only accept traffic coming directly from CloudFront).* This is a modern security best practice. It creates unique identities specifically for CloudFront. We then use these OAC identities in our S3 bucket policies. This creates an airtight seal: users are forced to access files through the CDN and are completely blocked from reading files directly from the S3 buckets. As seen in the image, we created dedicated OACs for our distinct S3 origins:
    1.  `publicast-staging-frontend-oac`: Secures the frontend static assets bucket.
    2.  `publicast-staging-backend-storage-oac`: Secures the user-uploaded media bucket.
    
    *   **Configured Attributes:** 
        *   `origin_access_control_origin_type = "s3"`: Specifies these identities are used for Amazon S3.
        *   `signing_behavior = "always"` & `signing_protocol = "sigv4"`: Ensures every request from CloudFront to S3 is securely signed and authenticated.
    
    {{< img "images/Workshop/services/cloudfront-oac.png" "AWS Console - CloudFront OAC" >}}
    <p align="center"><i>AWS Console - CloudFront OAC</i></p>

📁 **Source Code:** Open `terraform/modules/s3_cloudfront/main.tf` in your local IDE to view the full configuration.
