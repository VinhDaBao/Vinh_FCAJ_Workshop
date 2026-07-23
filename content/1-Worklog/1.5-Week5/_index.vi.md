---
title: "Week 5 Worklog"
date: 2026-04-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---


### Mục tiêu tuần 5:

* Hiểu các dịch vụ giám sát và high availability của AWS.
* Tìm hiểu cách giám sát tài nguyên AWS bằng Amazon CloudWatch.
* Khám phá Auto Scaling và Elastic Load Balancing.
* Xây dựng kiến trúc ứng dụng web có tính sẵn sàng cao và có khả năng mở rộng.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu Amazon CloudWatch.<br>&emsp;+ Metrics.<br>&emsp;+ Logs.<br>&emsp;+ Dashboards.<br>&emsp;+ CloudWatch Alarms. | 04/21/2026 | 04/21/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:**<br>&emsp;+ Tạo CloudWatch Alarms.<br>&emsp;+ Giám sát CPU utilization của EC2.<br>&emsp;+ Cấu hình SNS notifications cho alarm.<br>&emsp;+ Phân tích các metrics giám sát. | 04/22/2026 | 04/22/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu Elastic Load Balancing (ELB).<br>&emsp;+ Application Load Balancer (ALB).<br>&emsp;+ Target Groups.<br>&emsp;+ Health Checks.<br>- Cấu hình load balancing cho nhiều EC2 instance. | 04/23/2026 | 04/23/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu Amazon EC2 Auto Scaling.<br>&emsp;+ Launch Templates.<br>&emsp;+ Auto Scaling Groups.<br>&emsp;+ Scaling Policies.<br>- Cấu hình scaling tự động dựa trên các metric của CloudWatch. | 04/24/2026 | 04/24/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Triển khai một ứng dụng web có tính sẵn sàng cao.<br>&emsp;+ Cấu hình Application Load Balancer.<br>&emsp;+ Cấu hình Auto Scaling Group.<br>&emsp;+ Xác minh failover và hành vi tự động scaling. | 04/25/2026 | 04/25/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 5:

* Hiểu được tầm quan trọng của việc giám sát tài nguyên đám mây nhằm duy trì độ tin cậy và hiệu năng của hệ thống.

* Tìm hiểu các tính năng cốt lõi của Amazon CloudWatch, bao gồm:
  * Metrics
  * Logs
  * Dashboards
  * CloudWatch Alarms
  * Events

* Cấu hình thành công CloudWatch để giám sát mức sử dụng tài nguyên EC2 và hiệu năng ứng dụng.

* Tạo CloudWatch Alarms và tích hợp Amazon SNS để nhận email thông báo khi vượt ngưỡng đã định trước.

* Hiểu cách Elastic Load Balancing phân phối lưu lượng đầu vào giữa nhiều EC2 instance.

* Cấu hình thành công Application Load Balancer với:
  * Target Groups
  * Health Checks
  * Listener Rules

* Tìm hiểu cách Amazon EC2 Auto Scaling tự động điều chỉnh dung lượng tính toán theo nhu cầu tải.

* Tạo thành công Auto Scaling Group bằng Launch Template và cấu hình các chính sách scaling động.

* Xác minh rằng các EC2 instance mới được tự động khởi chạy khi tải tăng và bị xóa khi nhu cầu giảm.

* Có kinh nghiệm thực tế trong việc triển khai kiến trúc có tính sẵn sàng cao kết hợp:
  * Amazon EC2
  * Application Load Balancer
  * Auto Scaling
  * Amazon CloudWatch

* Hiểu cách các dịch vụ quản lý của AWS phối hợp với nhau để cải thiện khả năng mở rộng, độ chịu lỗi và hiệu quả vận hành.

* Xây dựng nền tảng vững chắc để học các kiến trúc serverless và dịch vụ hướng sự kiện trong các tuần tiếp theo.