---
title: "Week 10 Worklog"
date: 2026-05-25
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Hiểu các dịch vụ bảo mật và các best practices của AWS.
* Tìm hiểu cách bảo vệ tài nguyên đám mây và thông tin nhạy cảm.
* Khám phá các dịch vụ quản lý danh tính, mã hóa và phát hiện mối đe dọa.
* Có kinh nghiệm thực hành bảo mật các workload trên AWS.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu các best practices bảo mật AWS.<br>&emsp;+ Shared Responsibility Model.<br>&emsp;+ Principle of Least Privilege.<br>&emsp;+ Multi-Factor Authentication (MFA).<br>&emsp;+ AWS IAM Security Best Practices. | 05/26/2026 | 05/26/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu AWS Key Management Service (KMS) và AWS Secrets Manager.<br>&emsp;+ Customer Managed Keys.<br>&emsp;+ Encryption at Rest.<br>&emsp;+ Encryption in Transit.<br>&emsp;+ Secret Rotation. | 05/27/2026 | 05/27/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu AWS WAF và AWS Shield.<br>- Cấu hình các quy tắc Web ACL.<br>- Bảo vệ các ứng dụng web trước các cuộc tấn công phổ biến.<br>- Hiểu cơ chế bảo vệ DDoS. | 05/28/2026 | 05/28/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu Amazon GuardDuty và AWS Security Hub.<br>- Khám phá phát hiện mối đe dọa và các finding bảo mật.<br>- Xem lại khuyến nghị bảo mật và báo cáo tuân thủ. | 05/29/2026 | 05/29/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Mã hóa dữ liệu ứng dụng bằng AWS KMS.<br>&emsp;+ Lưu trữ thông tin xác thực trong Secrets Manager.<br>&emsp;+ Bật GuardDuty và Security Hub.<br>&emsp;+ Xem lại cảnh báo bảo mật và khắc phục các rủi ro đã xác định. | 05/30/2026 | 05/30/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 10:

* Hiểu được AWS Shared Responsibility Model và tầm quan trọng của việc triển khai bảo mật xuyên suốt hạ tầng đám mây.

* Tìm hiểu các best practices bảo mật AWS, bao gồm:
  * Identity and Access Management
  * Least Privilege Principle
  * Multi-Factor Authentication (MFA)
  * Resource Access Control

* Triển khai thành công mã hóa bằng AWS Key Management Service (KMS) để bảo vệ dữ liệu ứng dụng nhạy cảm.

* Tìm hiểu cách AWS Secrets Manager lưu trữ và quản lý an toàn thông tin xác thực và khóa API của ứng dụng.

* Cấu hình thành công các secret được mã hóa và hiểu cơ chế tự động xoay secret.

* Hiểu cách AWS WAF bảo vệ ứng dụng web trước các cuộc tấn công web phổ biến bằng cách lọc các yêu cầu HTTP và HTTPS.

* Tìm hiểu khả năng của AWS Shield trong việc giảm thiểu các cuộc tấn công Distributed Denial-of-Service (DDoS).

* Bật Amazon GuardDuty để giám sát liên tục tài khoản AWS cho các hoạt động đáng ngờ và mối đe dọa bảo mật tiềm ẩn.

* Sử dụng AWS Security Hub để tổng hợp các finding bảo mật và xem lại tuân thủ trên các dịch vụ AWS.

* Có kinh nghiệm thực tế trong việc xác định lỗ hổng bảo mật và áp dụng các biện pháp khắc phục phù hợp.

* Cải thiện hiểu biết về việc bảo mật môi trường đám mây sẵn sàng cho production bằng các dịch vụ bảo mật được quản lý của AWS.

* Xây dựng nền tảng bảo mật vững chắc cho việc thiết kế các kiến trúc đám mây đáng tin cậy, tuân thủ và đạt tiêu chuẩn doanh nghiệp.