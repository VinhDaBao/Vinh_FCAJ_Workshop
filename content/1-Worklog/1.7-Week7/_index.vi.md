---
title: "Week 7 Worklog"
date: 2026-05-04
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Tìm hiểu các khái niệm Infrastructure as Code (IaC) trên AWS.
* Hiểu cách cung cấp và quản lý tài nguyên đám mây bằng AWS CloudFormation.
* Khám phá AWS Cloud Development Kit (CDK).
* Tìm hiểu cách AWS Systems Manager đơn giản hóa việc quản trị hạ tầng.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu Infrastructure as Code (IaC).<br>&emsp;+ Lợi ích của IaC.<br>&emsp;+ CloudFormation Templates.<br>&emsp;+ Stacks và StackSets.<br>&emsp;+ Change Sets. | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:**<br>&emsp;+ Tạo một CloudFormation template.<br>&emsp;+ Triển khai một EC2 instance bằng CloudFormation.<br>&emsp;+ Cập nhật và xóa CloudFormation stacks. | 05/06/2026 | 05/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu AWS Cloud Development Kit (CDK).<br>&emsp;+ CDK Applications.<br>&emsp;+ Stacks.<br>&emsp;+ Constructs.<br>&emsp;+ Quy trình Synth và Deploy. | 05/07/2026 | 05/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu AWS Systems Manager.<br>&emsp;+ Session Manager.<br>&emsp;+ Parameter Store.<br>&emsp;+ Run Command.<br>- Thực hành kết nối tới EC2 mà không cần SSH keys. | 05/08/2026 | 05/08/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Triển khai hạ tầng bằng AWS CDK.<br>&emsp;+ Lưu cấu hình ứng dụng trong Parameter Store.<br>&emsp;+ Thực thi các lệnh từ xa bằng Systems Manager.<br>&emsp;+ Xác thực triển khai hạ tầng. | 05/09/2026 | 05/09/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 7:

* Hiểu được nguyên tắc và lợi ích của Infrastructure as Code (IaC) để tự động hóa việc cung cấp tài nguyên AWS.

* Tìm hiểu các thành phần cốt lõi của AWS CloudFormation, bao gồm:
  * Templates
  * Stacks
  * Stack Updates
  * Change Sets

* Tạo và triển khai thành công các tài nguyên AWS bằng CloudFormation templates.

* Tìm hiểu cách AWS Cloud Development Kit (CDK) cho phép triển khai hạ tầng thông qua các ngôn ngữ lập trình.

* Hiểu quy trình làm việc của AWS CDK, bao gồm:
  * Tạo constructs
  * Tổng hợp stack
  * Triển khai
  * Quản lý tài nguyên

* Triển khai thành công các tài nguyên đám mây bằng AWS CDK và xác minh việc tạo hạ tầng.

* Tìm hiểu các khả năng của AWS Systems Manager, bao gồm:
  * Session Manager
  * Parameter Store
  * Run Command

* Kết nối thành công tới các EC2 instance bằng Session Manager mà không cần truy cập SSH.

* Lưu trữ cấu hình ứng dụng an toàn bằng Parameter Store và truy xuất giá trị cho quá trình triển khai ứng dụng.

* Thực thi các lệnh quản trị từ xa trên EC2 instances bằng Systems Manager Run Command.

* Có kinh nghiệm thực tế trong việc tự động hóa triển khai hạ tầng và quản trị hệ thống bằng các dịch vụ gốc của AWS.

* Xây dựng nền tảng vững chắc cho việc containerization và triển khai ứng dụng cloud-native trong các tuần sau.