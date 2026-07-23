---
title: "Week 6 Worklog"
date: 2026-04-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:

* Hiểu về điện toán serverless và kiến trúc hướng sự kiện trên AWS.
* Tìm hiểu cách xây dựng ứng dụng serverless bằng AWS Lambda và Amazon API Gateway.
* Khám phá các dịch vụ truyền tin bất đồng bộ với Amazon SQS và Amazon SNS.
* Có kinh nghiệm thực hành phát triển các ứng dụng đám mây hướng sự kiện.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu các nền tảng của Serverless Computing.<br>&emsp;+ AWS Lambda.<br>&emsp;+ Kiến trúc hướng sự kiện.<br>&emsp;+ Lambda Functions.<br>&emsp;+ Execution Roles. | 04/28/2026 | 04/28/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:**<br>&emsp;+ Tạo một Lambda function.<br>&emsp;+ Cấu hình IAM execution roles.<br>&emsp;+ Kiểm tra việc thực thi function bằng các sample events.<br>&emsp;+ Giám sát execution logs bằng CloudWatch. | 04/29/2026 | 04/29/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu Amazon API Gateway.<br>&emsp;+ REST APIs.<br>&emsp;+ Resources và Methods.<br>&emsp;+ Stages.<br>&emsp;+ API Deployment.<br>- Tích hợp API Gateway với AWS Lambda. | 04/30/2026 | 04/30/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu Amazon SQS và Amazon SNS.<br>&emsp;+ Message Queues.<br>&emsp;+ Topics và Subscriptions.<br>&emsp;+ Message Delivery.<br>- Khám phá các mẫu giao tiếp hướng sự kiện. | 05/01/2026 | 05/01/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Xây dựng một serverless REST API.<br>&emsp;+ Kết nối API Gateway với Lambda.<br>&emsp;+ Publish và nhận messages bằng SNS và SQS.<br>&emsp;+ Xác thực quy trình làm việc hướng sự kiện. | 05/02/2026 | 05/02/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 6:

* Hiểu được các khái niệm và lợi ích của điện toán serverless trong việc xây dựng các ứng dụng đám mây có khả năng mở rộng.

* Tìm hiểu các tính năng cốt lõi của AWS Lambda, bao gồm:
  * Lambda Functions
  * Execution Environment
  * Event Sources
  * IAM Execution Roles

* Tạo và triển khai thành công các Lambda function để thực thi logic ứng dụng mà không cần cung cấp máy chủ.

* Cấu hình Amazon CloudWatch Logs để giám sát quá trình thực thi Lambda và khắc phục các vấn đề runtime.

* Tìm hiểu cách Amazon API Gateway mở ra các API RESTful và tích hợp liền mạch với AWS Lambda.

* Triển khai thành công một serverless REST API bằng API Gateway và Lambda.

* Hiểu các mẫu truyền tin bất đồng bộ bằng:
  * Amazon Simple Queue Service (SQS)
  * Amazon Simple Notification Service (SNS)

* Cấu hình thành công các SNS topics và SQS queues để trao đổi message giữa các thành phần ứng dụng phân tán.

* Xác minh quy trình xử lý sự kiện end-to-end bằng cách tích hợp API Gateway, Lambda, SNS và SQS vào một workflow serverless.

* Có kinh nghiệm thực tế trong việc thiết kế và triển khai các kiến trúc hướng sự kiện bằng các dịch vụ AWS được quản lý.

* Cải thiện hiểu biết về thiết kế ứng dụng có khả năng mở rộng, loosely coupled thông qua công nghệ serverless.

* Xây dựng nền tảng vững chắc cho việc học Infrastructure as Code (IaC) và tự động hóa đám mây trong tuần tiếp theo.