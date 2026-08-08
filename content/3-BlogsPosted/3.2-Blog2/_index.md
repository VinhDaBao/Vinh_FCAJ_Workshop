---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---



# A $30 LESSON FROM IGNORING AN AWS EMAIL

## During the Project Deployment

During the deployment of our project on AWS, our team encountered a memorable issue involving Amazon RDS. Although the system continued to operate normally without any technical errors, we ended up paying approximately **$30** in additional charges simply because we overlooked an AWS notification. This was a cost that could have been completely avoided.

At that time, our project database was running on Amazon RDS MySQL 8.0. AWS had notified the account that MySQL 8.0 was approaching the end of its Standard Support period and recommended upgrading to a newer version before the deadline. According to the notification, after MySQL 8.0 reached the end of its community support lifecycle, Amazon RDS would also end Standard Support and move databases that had not been upgraded to Amazon RDS Extended Support. This mode would continue providing important security updates, but it would also introduce additional charges based on usage if the database was not upgraded to a supported version [1][2][7].

At the time we received the email, the team was focused on completing the final features of the system, so we only glanced at the subject and ignored it. Since the database was still operating normally, we assumed that the email was simply a reminder that could be handled later.

This assumption eventually led to the problem. Once the support deadline passed, AWS automatically moved our MySQL 8.0 database to Extended Support. Since the system continued operating normally, nobody noticed that additional charges were already being generated every hour.

## Discovering the Unexpected Charge

We only discovered the issue when checking the Billing dashboard to review the project's deployment costs. A new charge related to Amazon RDS Extended Support had appeared.

After checking the AWS Health Dashboard and reviewing the previous notification, we realized that the charge was caused by our failure to upgrade the database to MySQL 8.4 before the deadline.

The team immediately decided to upgrade the database according to AWS's recommended procedure. However, another mistake occurred during the upgrade configuration.

Instead of selecting **Upgrade immediately**, we kept the default option to **Apply during the next maintenance window**.

This meant that the upgrade request was only scheduled, while the database continued running on MySQL 8.0 under Extended Support until the maintenance window occurred. During this waiting period, the system continued to incur Extended Support charges as usual [6].

Once the upgrade was completed and the database was moved to MySQL 8.4, we checked the bill again. The total Extended Support charges had reached approximately **$30**.

Although this amount is relatively small for an enterprise system, it was a significant and unnecessary expense for a student project with a limited budget. More importantly, the cost was not caused by a technical failure, but by a mistake in service management and operations.

## Lessons About AWS Management

After this incident, we realized that using cloud services is not only about designing the right architecture or writing stable code. Administrators also need to regularly monitor notifications in the AWS Health Dashboard, carefully read emails from AWS, and check the Billing Dashboard to identify unexpected charges as early as possible.

Service lifecycle notifications may appear to be informational, but they can have a direct impact on the operating cost of a system.

We also learned that every option should be carefully reviewed when performing service upgrades. If the goal is to stop Extended Support charges as soon as possible, choosing an immediate upgrade may be more appropriate than waiting for the next maintenance window, provided that the potential service interruption is acceptable.

AWS also recommends creating a snapshot before an upgrade so that the database can be restored if necessary. For systems serving users, Blue/Green Deployment can also be used to minimize downtime [4][5].

## Looking Back

This experience taught us that in a cloud environment, even a small mistake such as ignoring an email or overlooking a default option can result in unexpected costs.

With only two small mistakes — failing to read the AWS notification and not performing the upgrade immediately — our team ended up paying approximately **$30** in additional charges.

Since then, we have developed a habit of regularly checking the AWS Health Dashboard, monitoring the Billing Dashboard, and responding to important notifications as soon as they are received. This helps us avoid similar situations in future projects.

Key points to know:

* Always read and respond to AWS notifications.
* Regularly monitor the AWS Health Dashboard and Billing Dashboard.
* Upgrade services before their support period ends.
* Carefully review upgrade options before applying changes.
* Be aware that default configurations can result in unexpected costs
