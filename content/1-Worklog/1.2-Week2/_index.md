---
title: "Week 2 Worklog"
date: 2026-03-30
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---



### Week 2 Objectives:

* Understand the core AWS networking services.
* Learn how to build a secure network architecture using Amazon VPC.
* Configure network access for EC2 instances.
* Practice deploying and testing network connectivity.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Learn Amazon Virtual Private Cloud (VPC).<br>&emsp;+ VPC architecture.<br>&emsp;+ CIDR Blocks.<br>&emsp;+ Public and Private Subnets.<br>&emsp;+ Route Tables. | 03/31/2026 | 03/31/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Learn Internet Gateway and Route Table configuration.<br>- Create a custom VPC.<br>- Configure Public and Private Subnets.<br>- Verify Internet connectivity for public resources. | 04/01/2026 | 04/01/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Learn Security Groups and Network ACLs.<br>- Compare stateful and stateless firewall mechanisms.<br>- Configure inbound and outbound traffic rules for EC2. | 04/02/2026 | 04/02/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Learn about Elastic IP addresses.<br>- Associate and disassociate Elastic IP with EC2 instances.<br>- Test remote SSH connectivity after IP reassignment. | 04/03/2026 | 04/03/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Practice:**<br>&emsp;+ Deploy an EC2 instance inside a custom VPC.<br>&emsp;+ Configure Security Groups and Route Tables.<br>&emsp;+ Validate Internet access.<br>&emsp;+ Troubleshoot networking issues. | 04/04/2026 | 04/04/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 2 Achievements:

* Understood the architecture and components of Amazon Virtual Private Cloud (VPC).

* Successfully designed and deployed a custom VPC including:
  * Public Subnet
  * Private Subnet
  * Route Tables
  * Internet Gateway

* Learned how CIDR blocks define IP address allocation within a VPC.

* Understood the differences between:
  * Public Subnets
  * Private Subnets
  * Internet-facing resources
  * Internal resources

* Successfully configured routing rules to allow Internet access for resources deployed in public subnets.

* Learned the functionality of Security Groups as stateful virtual firewalls.

* Understood how Network ACLs provide subnet-level traffic filtering, including:
  * Stateless packet filtering
  * Rule evaluation order
  * Inbound and Outbound rules

* Successfully configured Security Groups to permit secure SSH access while restricting unnecessary inbound traffic.

* Learned how Elastic IP addresses provide persistent public IP addresses for EC2 instances.

* Successfully associated an Elastic IP with an EC2 instance and verified remote connectivity.

* Practiced troubleshooting common networking issues such as:
  * Missing Internet Gateway
  * Incorrect Route Table configuration
  * Misconfigured Security Group rules
  * Network ACL restrictions

* Built a strong networking foundation for deploying scalable and secure AWS applications in subsequent weeks.