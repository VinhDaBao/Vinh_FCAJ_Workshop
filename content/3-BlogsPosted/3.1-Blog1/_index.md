---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# ROUTE 53 IN THE TEAM'S MULTI-PLATFORM CONTENT MANAGEMENT PROJECT

# Route 53 in the Team's Content Management Project

Our project is a content management platform that allows users to create a post once and publish it across multiple social media platforms such as Facebook, LinkedIn, and X (Twitter) through their official APIs. To achieve this, the system needs to integrate the OAuth mechanism of each platform so that users can securely connect their accounts.

Each platform requires a fixed **Redirect URI (Callback URL)** to be registered before the application can be used. Therefore, having a stable domain name is very important. Instead of using an IP address or a temporary URL during development, we configured a subdomain such as `api.example.com` through Route 53 as the callback address for the entire system.

As a result, when users click the **"Connect Account"** button, the platforms redirect them to the correct backend endpoint after authentication is completed. In the future, even if the team changes the infrastructure, switches to another Load Balancer, or redeploys ECS, the Callback URL can remain unchanged. The team only needs to update the corresponding Route 53 record, and the entire system can continue working without reconfiguring every external platform.

This was when we realized that a domain name is not only used to access a website. It is also an important component that allows external services to identify and trust our application. Managing DNS through Route 53 has made the integration with multiple social media platforms much more stable and easier to maintain.

## Starting with Domain Management

Initially, the team purchased a domain name for the project. Instead of using the DNS service provided by the domain registrar, we decided to manage the entire DNS configuration through Amazon Route 53. The reason was quite simple: AWS services integrate very well with Route 53, making future configuration much more convenient.

After creating a Hosted Zone, the team received four Name Servers (NS) provided by AWS. The next step was to update these Name Servers at the domain registrar. This is an easy step to overlook because if the Name Servers are not updated, changes made in Route 53 will have no effect.

After waiting several minutes to several hours for the DNS changes to propagate, the team started creating the first records:

* **A or Alias Record** to point the main domain to the Application Load Balancer.
* **CNAME Record** for subdomains such as `api.example.com`.
* **MX Record** if email services are required.
* **TXT Record** for verification with various services.

At this point, we realized that Route 53 is not simply a place to "point a domain", but also a central place for managing almost all information related to the domain.

## Our First Domain Verification with a TXT Record

During the integration of third-party services such as Google OAuth, TikTok, Facebook, and some email platforms, the team repeatedly encountered the request:

> Please verify your domain.

At first, we thought domain verification would require uploading an HTML file to the website, similar to Google Search Console. However, after reading the documentation, we learned that many modern services use a **TXT Record** to verify domain ownership.

For example, a service may require a record such as:

```text
Type: TXT
Name: _verification.example.com
Value: x7d91abdf8c....
```

After adding the record to Route 53, the service checks the global DNS system. If the correct value is found, the service confirms that the team owns the domain.

This was an interesting experience because we realized that many systems on the Internet rely heavily on DNS, rather than using DNS only for accessing websites.

From that point onward, almost every new service we integrated required at least one TXT Record for verification.

## Route 53 and AWS Certificate Manager

One of the moments that surprised us most was when we created an HTTPS certificate.

Initially, the team thought that enabling HTTPS would require purchasing an SSL Certificate as we had seen before.

However, **AWS Certificate Manager (ACM)** allows certificates to be issued at no additional charge for supported AWS resources.

The condition is that we must prove ownership of the domain.

Once again, DNS became an important part of the process.

After requesting a certificate, ACM generated a CNAME Record such as:

```text
Name:
_abcd123.example.com

Value:
_xyz.acm-validations.aws
```

We only needed to add the record to Route 53.

After a few minutes, the certificate status changed from:

```text
Pending Validation
```

to:

```text
Issued
```

The team barely needed to perform any additional operations.

When Route 53 and ACM are used within the same AWS account, AWS can also automatically create the validation record in many cases.

After attaching the certificate to the Application Load Balancer, the website had fully enabled HTTPS.

* No need to purchase an SSL certificate.
* No need to install Let's Encrypt.
* No need to renew the certificate manually.

ACM can automatically renew the certificate before it expires as long as the validation record remains available.

This is probably one of the features we liked most about using AWS.

## Learning How to Organize Subdomains

Initially, every component ran under the main domain.

After some time, the team started separating the system into multiple subdomains.

For example:

```text
www.example.com    → Landing website
api.example.com    → Backend
pub.example.com    → Static website
docs.example.com   → Documentation
cdn.example.com    → Static resources
```

With Route 53, managing these subdomains is very straightforward.

If the architecture changes later, we only need to update the corresponding Record without affecting the other services.

## Alias Records Make Things Much Simpler

One useful thing we learned is that AWS usually does not require a CNAME for the root domain.

Instead, we can use an **Alias Record**.

An Alias can directly point to:

* Application Load Balancer
* CloudFront
* S3 Static Website
* API Gateway

The main advantage is that we do not need to know the actual IP address of the service.

If AWS changes the infrastructure behind the service, Route 53 can handle the change automatically.

This is something traditional DNS configuration cannot provide in the same way.

## Some Expensive Lessons

During our experience with Route 53, the team also encountered several errors.

At one point, we created a Hosted Zone for:

```text
pub.example.com
```

when it should have been:

```text
example.com
```

As a result, the DNS records did not work as expected.

On another occasion, the team added the correct TXT Record value, but domain verification still failed.

After investigating, we discovered that the domain's Name Servers were still pointing to the old DNS provider instead of Route 53.

There were also times when the team thought Route 53 was broken because a newly created record could not immediately be found by the verification service.

We later learned that DNS changes can take time to propagate and can also be affected by TTL and DNS caches maintained by different network providers.

Although these were relatively small mistakes, they helped us understand how DNS actually works instead of simply following instructions.

## Looking Back

After several months of using it, we realized that Amazon Route 53 is more than just a DNS management service. It acts as an important layer connecting many components of the system. From pointing the domain to an Application Load Balancer, verifying domain ownership through TXT Records, issuing HTTPS certificates through AWS Certificate Manager, to configuring Callback URLs for OAuth, payment services, and webhooks, Route 53 is involved in many parts of the deployment process.

What the team learned was not only how to create DNS records, but also how to manage domains in a structured way. When DNS configurations are organized clearly, expanding the system, changing infrastructure, or integrating new services becomes much easier.

For a group of students building a cloud-based product for the first time, Route 53 was one of the AWS services that provided the most practical lessons. It helped us understand that a domain name is not only used to access a website, but can also act as a "key" for verifying system ownership, establishing secure connections, and integrating with many modern services on the Internet.

Key points to know:

* Understand how to use Amazon Route 53 to manage Hosted Zones and DNS Records.
* Understand the importance of a stable domain for OAuth Redirect URIs and Callback URLs.
* Understand practical use cases for A, Alias, CNAME, MX, and TXT Records.
* Learn how TXT Records can be used to verify domain ownership.
* Understand how Route 53 works with AWS Certificate Manager for HTTPS certificate validation.
* Recognize how stable DNS endpoints reduce dependency on specific backend infrastructure.
* Understand how organizing domains and subdomains improves system management, scalability, and maintainability.

[Link Blog 1](https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2230396147725345)