---
title: "Week 8 Worklog"
date: 2026-05-11
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Hiểu các nền tảng cơ bản của containerization và Docker.
* Tìm hiểu cách triển khai các ứng dụng container bằng Amazon ECS.
* Khám phá quản lý image container bằng Amazon ECR.
* Có kinh nghiệm thực hành triển khai các ứng dụng có khả năng mở rộng bằng các dịch vụ container của AWS.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu các nền tảng của containerization.<br>&emsp;+ Containers vs Virtual Machines.<br>&emsp;+ Docker Architecture.<br>&emsp;+ Docker Images và Containers.<br>&emsp;+ Dockerfile basics. | 05/12/2026 | 05/12/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:**<br>&emsp;+ Cài đặt Docker.<br>&emsp;+ Xây dựng một Docker image.<br>&emsp;+ Chạy và quản lý Docker containers.<br>&emsp;+ Đẩy image lên Amazon Elastic Container Registry (ECR). | 05/13/2026 | 05/13/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu Amazon Elastic Container Service (ECS).<br>&emsp;+ ECS Clusters.<br>&emsp;+ Task Definitions.<br>&emsp;+ Services.<br>&emsp;+ Launch Types (EC2 & Fargate). | 05/14/2026 | 05/14/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Cấu hình một ECS cluster.<br>- Tạo Task Definitions và ECS Services.<br>- Triển khai một ứng dụng web container bằng Amazon ECS Fargate.<br>- Xác minh tính khả dụng của service. | 05/15/2026 | 05/15/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Cập nhật một container image.<br>&emsp;+ Thực hiện rolling deployment trên ECS.<br>&emsp;+ Giám sát ECS services bằng CloudWatch.<br>&emsp;+ Xác thực khả năng truy cập và khả năng mở rộng của ứng dụng. | 05/16/2026 | 05/16/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 8:

* Hiểu các khái niệm và lợi ích của containerization so với máy ảo truyền thống.

* Tìm hiểu các thành phần cốt lõi của Docker, bao gồm:
  * Docker Images
  * Docker Containers
  * Dockerfile
  * Docker Hub
  * Docker Engine

* Xây dựng, chạy và quản lý thành công Docker containers trong môi trường phát triển cục bộ.

* Tìm hiểu cách Amazon Elastic Container Registry (ECR) lưu trữ và quản lý container images một cách an toàn.

* Đẩy và quản lý thành công Docker images trong Amazon ECR.

* Hiểu kiến trúc của Amazon Elastic Container Service (ECS), bao gồm:
  * ECS Clusters
  * Task Definitions
  * ECS Services
  * Fargate Launch Type
  * EC2 Launch Type

* Triển khai thành công một ứng dụng container trên Amazon ECS bằng AWS Fargate.

* Tìm hiểu cách ECS tự động quản lý việc lập lịch, mở rộng và tính sẵn sàng của container.

* Cập nhật thành công các container ứng dụng thông qua rolling deployments với ít gián đoạn dịch vụ.

* Giám sát ECS tasks và services bằng Amazon CloudWatch để xác minh sức khỏe và hiệu năng ứng dụng.

* Có kinh nghiệm thực tế trong việc triển khai các ứng dụng cloud-native bằng Docker, Amazon ECR và Amazon ECS.

* Xây dựng nền tảng vững chắc cho việc triển khai CI/CD và quá trình triển khai ứng dụng tự động trong các tuần tiếp theo.