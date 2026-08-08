---
title: "5.3.3.1. Network & Security Layer"
weight: 1
---

This layer establishes the secure foundation of the PubliCast platform. It isolates our resources from the public internet and strictly controls access permissions. Think of this layer as the walls, doors, and security guards of your virtual data center.

## Virtual Private Cloud (`vpc`)
**Purpose:** Provides an isolated, private network environment on AWS. It acts as the absolute perimeter for all our infrastructure. Without a VPC, you cannot deploy modern AWS resources securely.

**Key Terraform Resources:** 
To build a highly available and cost-effective network, the module orchestrates several key blocks. 

**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_vpc`: Think of the VPC as the main container for your network. This resource defines the primary network boundary and assigns the main IP address range (known as the IPv4 CIDR block). Everything we build will live inside this IP range.
    *   **Configured Attributes:** 
        *   `cidr_block`: Driven by our variables (e.g., `10.0.0.0/16`), dictating the size of our network.
        *   `tags`: Dynamically merged with `${var.project}-${var.environment}-vpc` to easily identify it in the console.
    
    {{< img "images/Workshop/services/vpc.png" "AWS Console - Main VPC" >}}
    <p align="center"><i>AWS Console - Main VPC</i></p>
    
*   `aws_subnet`: Subnets are smaller, manageable chunks of your VPC's network. As seen in the image below, Terraform creates exactly **4 subnets** for our project. Why 4? 
    *   First, we divide the network by **Accessibility**: we need **Public Subnets** (for resources that must talk to the internet, like Load Balancers) and **Private Subnets** (to securely hide our databases and app containers from the outside world).
    *   Second, we divide by **High Availability**: AWS groups its physical data centers into Availability Zones (AZs). If an entire data center loses power, we don't want our app to go offline. By using Terraform's `count` loops to deploy across **2 different AZs**, we end up with a matrix: (1 Public + 1 Private) × 2 AZs = 4 Subnets. This guarantees fault tolerance.
    *   **Configured Attributes:** 
        *   `availability_zone`: Dynamically assigned using `data.aws_availability_zones` to spread them across multiple physical data centers.
        *   `map_public_ip_on_launch = true`: Applied *only* to the Public subnets so resources inside them automatically get a public IP.
    
    {{< img "images/Workshop/services/subnets.png" "AWS Console - Subnets" >}}
    <p align="center"><i>AWS Console - Subnets</i></p>
    
*   `aws_internet_gateway`: This is the literal "door" to the internet. Attached to the VPC and routed via the Public Route Table, the Internet Gateway allows resources living in our Public Subnets (like our Load Balancer) to receive incoming web traffic from your users.
    *   **Configured Attributes:** 
        *   `vpc_id`: Explicitly links this gateway to our newly created VPC.
    
    {{< img "images/Workshop/services/igw.png" "AWS Console - Internet Gateway" >}}
    <p align="center"><i>AWS Console - Internet Gateway</i></p>
    
*   `aws_nat_gateway` & `aws_eip`: What if a server in our Private Subnet needs to download a software update or talk to a third-party API (like Stripe or Meta)? Because it's private, it can't use the Internet Gateway directly. We place a NAT Gateway in the Public Subnet to act as a secure middleman. It takes requests from the Private Subnet, fetches the data from the internet, and returns it, all without ever exposing the private server to incoming internet attacks.
    *   **Configured Attributes:** 
        *   `allocation_id`: Connects the NAT Gateway to an Elastic IP (`aws_eip`) so it has a static public address.
        *   `subnet_id`: Instructs AWS to specifically place this gateway in the *Public* subnet.
    
    {{< img "images/Workshop/services/natgw.png" "AWS Console - NAT Gateway" >}}
    <p align="center"><i>AWS Console - NAT Gateway</i></p>
    
*   `aws_vpc_endpoint`: Configured as a `Gateway` type for Amazon S3. Normally, downloading large media files from S3 would route traffic out to the public internet and back in, incurring heavy NAT Gateway data processing fees. The VPC Endpoint creates a private shortcut directly to S3. This modifies the Private Route Table so that any traffic destined for S3 stays entirely within the AWS internal network, saving us a lot of money and improving speed.
    *   **Configured Attributes:** 
        *   `vpc_endpoint_type = "Gateway"`: Specifies we want a Gateway endpoint (which is free) rather than an Interface endpoint.
        *   `service_name`: Dynamically points to `com.amazonaws.${region}.s3`.
        *   `route_table_ids`: Automatically injects this shortcut into our Private Route Tables.
    
    {{< img "images/Workshop/services/endpoint.png" "AWS Console - VPC Endpoints" >}}
    <p align="center"><i>AWS Console - VPC Endpoints</i></p>

📁 **Source Code:** Open `terraform/modules/vpc/main.tf` in your local IDE to view the full configuration.

## Security Groups (`security_group`)
**Purpose:** Functions as virtual network firewalls to allow or deny traffic at the individual instance level. If the VPC is the wall around the building, Security Groups are the locked doors to each specific room.

**Key Terraform Resources:** 
We implement a "Zero Trust" architecture within the VPC. This means that even if a resource is inside our private network, it is not trusted by default.

**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_security_group`: Creates the logical firewall boundary. We create distinct security groups for different layers of our app.
    *   **Configured Attributes:** 
        *   `name` & `description`: Clearly labels the firewall's purpose.
        *   `vpc_id`: Binds the security group to our custom VPC, rather than the default VPC.
    
    {{< img "images/Workshop/services/sg.png" "AWS Console - Security Groups List" >}}
    <p align="center"><i>AWS Console - Security Groups List</i></p>

Below we split the `security_group` topic into the four concrete groups used in PubliCast.

### 1) ALB Security Group (`alb-sg`)
**Purpose:** Responsible for receiving traffic from the Internet and forwarding it to our ECS Tasks via the Application Load Balancer.
*   **Ingress (Inbound):** Allows HTTP (80) and HTTPS (443) from anywhere (`0.0.0.0/0` and IPv6 `::/0`).
*   **Egress (Outbound):** Allows outbound traffic to the `ecs-tasks-sg` on the application port to forward user requests, and perform health checks.

{{< img "images/Workshop/services/alb-sg.png" "AWS Console - ALB Security Group" >}}
<p align="center"><i>AWS Console - ALB Security Group</i></p>

### 2) ECS Tasks Security Group (`ecs-tasks-sg`)
**Purpose:** Attached directly to our running backend containers (Node.js API, Workers). It strictly hides the application from the public internet.
*   **Ingress:** Allows traffic *only* from the `alb-sg` on the application port.
*   **Egress:** Allows outbound traffic to connect to S3 (via VPC endpoints), Secrets Manager, RDS, and Redis.

{{< img "images/Workshop/services/ecs-sg.png" "AWS Console - ECS Tasks Security Group" >}}
<p align="center"><i>AWS Console - ECS Tasks Security Group</i></p>

### 3) Redis Security Group (`redis-sg`)
**Purpose:** Protects the ElastiCache Redis cluster, ensuring only authorized internal application containers can access the cache.
*   **Ingress:** Allows traffic *only* from the `ecs-tasks-sg` on port `6379`.
*   **Egress:** Restricted to internal network communication.

{{< img "images/Workshop/services/redis-sg.png" "AWS Console - Redis Security Group" >}}
<p align="center"><i>AWS Console - Redis Security Group</i></p>

### 4) RDS Security Group (`rds-sg`)
**Purpose:** Secures the relational database (MySQL). This is the most heavily guarded component holding our persistent data.
*   **Ingress:** Allows traffic *only* from the `ecs-tasks-sg` on the database port (e.g., `3306`).
*   **Egress:** Restricted minimal egress.

{{< img "images/Workshop/services/rds-sg.png" "AWS Console - RDS Security Group" >}}
<p align="center"><i>AWS Console - RDS Security Group</i></p>

> [!TIP]
> **Security Group Referencing:** This is our most powerful security feature! Notice how for RDS and Redis, we do *not* allow broad IP ranges like `10.0.0.0/16`. Instead, we set the `source_security_group_id` attribute to point exactly to the ID of the ECS Tasks Security Group. Even if a hacker compromises another server in the same subnet, they cannot access the database because they aren't wearing the `ecs-tasks-sg` badge.

📁 **Source Code:** Open `terraform/modules/security_group/main.tf` in your local IDE to view the full configuration.

## Identity and Access Management (`iam`)
**Purpose:** Controls which identities (users, services, or roles) can access specific AWS APIs and resources. While Security Groups control *network* access, IAM controls *API* access.

**Key Terraform Resources:** 
We adhere to the principle of least privilege for our containerized applications, meaning we only give them the exact permissions they need to do their job, and nothing more.

**Result:** After running `terraform apply`, you can verify these individual components in the AWS Console:

*   `aws_iam_role`: Creates the roles. Think of a role as a "hat" that a service can wear to gain temporary permissions. 
    *   **Configured Attributes:** 
        *   `assume_role_policy`: We define a JSON block here stating that only the AWS ECS service (`ecs-tasks.amazonaws.com`) is allowed to put on this hat.
    
    {{< img "images/Workshop/services/ecs-iam-role.png" "AWS Console - IAM Role" >}}
    <p align="center"><i>AWS Console - IAM Role</i></p>
    
*   **Task Execution Role (`aws_iam_role_policy_attachment`):** We use this resource to attach the AWS-managed `AmazonECSTaskExecutionRolePolicy`. This role is for the invisible ECS agent running in the background. 
    *   **Configured Attributes:** 
        *   `policy_arn`: Points to the official AWS policy ARN, giving the agent permission to do basic operational tasks like pulling Docker images securely from ECR and pushing `console.log` output to CloudWatch.
*   **Task Role (`aws_iam_role_policy`):** We craft custom, inline JSON policies that are meant for your actual Node.js application containers. As seen in the console, we attach specific inline policies:
    *   **`backend-s3-access-policy`**: Grants the application the `s3:PutObject`, `s3:GetObject`, and `s3:DeleteObject` permissions so users can upload and manage media files in our specific S3 bucket.
    *   **`[project]-ecs-exec-policy`**: Grants permissions like `secretsmanager:GetSecretValue` (so the app can securely fetch its database passwords) and `ssmmessages:CreateControlChannel` (allowing administrators to securely open a terminal inside the container for debugging using ECS Exec).
    *   **Configured Attributes:** 
        *   `policy`: Uses `jsonencode()` to safely inject these strict JSON policy documents (Effect, Action, Resource). If your app is ever compromised, the hacker can only perform these exact actions, limiting the blast radius.
    
    {{< img "images/Workshop/services/iam-policy.png" "AWS Console - IAM Policy" >}}
    <p align="center"><i>AWS Console - IAM Policy</i></p>

📁 **Source Code:** Open `terraform/modules/iam/main.tf` in your local IDE to view the full configuration.
