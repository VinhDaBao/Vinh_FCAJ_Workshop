---
title: "Week 3 Worklog"
date: 2026-04-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu các dịch vụ lưu trữ và DNS của AWS.
* Tìm hiểu cách lưu trữ website tĩnh bằng Amazon S3.
* Khám phá Amazon Route 53 cho quản trị tên miền và DNS.
* Tìm hiểu cách Amazon CloudFront cải thiện hiệu năng phân phối nội dung.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu Amazon Simple Storage Service (S3).<br>&emsp;+ Buckets.<br>&emsp;+ Objects.<br>&emsp;+ Storage Classes.<br>&emsp;+ Bucket Policies. | 04/07/2026 | 04/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:**<br>&emsp;+ Tạo một S3 Bucket.<br>&emsp;+ Tải lên các tệp website.<br>&emsp;+ Cấu hình Static Website Hosting.<br>&emsp;+ Kiểm tra khả năng truy cập công khai. | 04/08/2026 | 04/08/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu Amazon Route 53.<br>&emsp;+ Hosted Zones.<br>&emsp;+ DNS Records.<br>&emsp;+ Domain Registration.<br>&emsp;+ Routing Policies. | 04/09/2026 | 04/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu Amazon CloudFront.<br>- Tạo một CloudFront Distribution cho website tĩnh trên S3.<br>- Hiểu cơ chế caching và phân phối nội dung. | 04/10/2026 | 04/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Kết nối Route 53 với website S3.<br>&emsp;+ Cấu hình CloudFront.<br>&emsp;+ Xác minh khả năng truy cập website qua CDN.<br>&emsp;+ Kiểm tra cập nhật cache và invalidation. | 04/11/2026 | 04/11/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 3:

* Hiểu được kiến trúc và các trường hợp sử dụng của Amazon Simple Storage Service (S3).

* Tìm hiểu các khái niệm cốt lõi của Amazon S3, bao gồm:
  * Buckets
  * Objects
  * Storage Classes
  * Bucket Policies
  * Static Website Hosting

* Tạo và cấu hình thành công một bucket S3 để lưu trữ website tĩnh.

* Tìm hiểu cách quản lý quyền đối với object và bật public access một cách an toàn.

* Hiểu mục đích của Amazon Route 53 và vai trò của nó như dịch vụ DNS được quản lý.

* Cấu hình thành công các thành phần của Route 53, bao gồm:
  * Hosted Zones
  * A Records
  * Alias Records
  * DNS Resolution

* Tìm hiểu cách Amazon CloudFront phân phối nội dung toàn cầu thông qua các edge locations.

* Tạo thành công một CloudFront distribution cho website tĩnh và xác minh việc phân phối nội dung qua CDN.

* Hiểu các khái niệm:
  * Cache Behavior
  * Cache Invalidation
  * Origin Configuration
  * HTTPS Content Delivery

* Tích hợp thành công Amazon S3, Route 53 và CloudFront để xây dựng một website tĩnh an toàn và có sẵn cao.

* Có kinh nghiệm thực tế trong việc triển khai các ứng dụng web tĩnh bằng các dịch vụ AWS được quản lý.

* Xây dựng nền tảng vững chắc để lưu trữ các ứng dụng web có thể mở rộng và chuẩn bị cho tích hợp cơ sở dữ liệu trong các tuần tiếp theo.