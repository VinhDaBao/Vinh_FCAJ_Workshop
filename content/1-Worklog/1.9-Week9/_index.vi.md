---
title: "Week 9 Worklog"
date: 2026-05-18
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Hiểu các nguyên tắc Continuous Integration và Continuous Deployment (CI/CD).
* Tìm hiểu cách tự động hóa việc build, test và triển khai ứng dụng bằng AWS Developer Tools.
* Khám phá AWS CodeCommit, CodeBuild, CodeDeploy và CodePipeline.
* Có kinh nghiệm thực hành xây dựng một pipeline triển khai tự động.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu khái niệm CI/CD.<br>&emsp;+ Continuous Integration.<br>&emsp;+ Continuous Delivery.<br>&emsp;+ Continuous Deployment.<br>&emsp;+ Thực hành DevOps tốt nhất. | 05/19/2026 | 05/19/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu AWS Developer Tools.<br>&emsp;+ AWS CodeCommit.<br>&emsp;+ AWS CodeBuild.<br>&emsp;+ AWS CodeDeploy.<br>&emsp;+ AWS CodePipeline. | 05/20/2026 | 05/20/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **Thực hành:**<br>&emsp;+ Tạo một source code repository.<br>&emsp;+ Cấu hình một CodeBuild project.<br>&emsp;+ Tự động build một Docker image.<br>&emsp;+ Đẩy artifacts lên Amazon ECR. | 05/21/2026 | 05/21/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Cấu hình AWS CodePipeline.<br>- Kết nối source repository với CodeBuild và các giai đoạn deployment.<br>- Triển khai ứng dụng lên Amazon ECS.<br>- Xác minh việc thực thi pipeline tự động sau khi có thay đổi source code. | 05/22/2026 | 05/22/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Chỉnh sửa source code ứng dụng.<br>&emsp;+ Kích hoạt pipeline execution.<br>&emsp;+ Giám sát các giai đoạn pipeline.<br>&emsp;+ Xác thực deployment thành công và tính khả dụng của ứng dụng. | 05/23/2026 | 05/23/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 9:

* Hiểu vòng đời phát triển phần mềm và tầm quan trọng của CI/CD trong các ứng dụng đám mây hiện đại.

* Tìm hiểu các công cụ AWS Developer Tools cốt lõi, bao gồm:
  * AWS CodeCommit
  * AWS CodeBuild
  * AWS CodeDeploy
  * AWS CodePipeline

* Tạo thành công một source code repository và tích hợp nó vào quy trình triển khai tự động.

* Cấu hình AWS CodeBuild để:
  * Build source code ứng dụng
  * Build Docker images
  * Tạo deployment artifacts
  * Đẩy container images lên Amazon ECR

* Cấu hình thành công AWS CodePipeline để tự động hóa:
  * Truy xuất source
  * Build ứng dụng
  * Xuất bản container image
  * Triển khai lên Amazon ECS

* Xác minh rằng thay đổi mã nguồn tự động kích hoạt một lần chạy pipeline mới mà không cần can thiệp thủ công.

* Tìm hiểu cách các giai đoạn deployment, quy trình phê duyệt và build logs giúp tăng độ tin cậy của triển khai.

* Giám sát việc thực thi pipeline và phân tích build logs để khắc phục lỗi triển khai.

* Triển khai thành công các phiên bản ứng dụng mới lên Amazon ECS thông qua pipeline CI/CD tự động.

* Có kinh nghiệm thực tế trong việc tích hợp các dịch vụ container với AWS Developer Tools cho việc cung cấp phần mềm liên tục.

* Xây dựng nền tảng vững chắc cho việc bảo mật môi trường đám mây và áp dụng các best practices bảo mật AWS trong các tuần tiếp theo.