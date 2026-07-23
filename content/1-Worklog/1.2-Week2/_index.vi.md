---
title: "Week 2 Worklog"
date: 2026-03-30
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Hiểu các dịch vụ mạng cơ bản của AWS.
* Tìm hiểu cách xây dựng kiến trúc mạng an toàn bằng Amazon VPC.
* Cấu hình quyền truy cập mạng cho các EC2 instance.
* Thực hành triển khai và kiểm tra kết nối mạng.

### Các công việc sẽ thực hiện trong tuần này:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2 | - Tìm hiểu Amazon Virtual Private Cloud (VPC).<br>&emsp;+ Kiến trúc VPC.<br>&emsp;+ CIDR Blocks.<br>&emsp;+ Public và Private Subnets.<br>&emsp;+ Route Tables. | 03/31/2026 | 03/31/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu Internet Gateway và cấu hình Route Table.<br>- Tạo một VPC tùy chỉnh.<br>- Cấu hình Public và Private Subnets.<br>- Xác minh kết nối Internet cho các tài nguyên công khai. | 04/01/2026 | 04/01/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tìm hiểu Security Groups và Network ACLs.<br>- So sánh cơ chế firewall stateful và stateless.<br>- Cấu hình quy tắc lưu lượng đến và đi cho EC2. | 04/02/2026 | 04/02/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu về địa chỉ Elastic IP.<br>- Gắn và tách Elastic IP với các EC2 instance.<br>- Kiểm tra kết nối SSH từ xa sau khi thay đổi IP. | 04/03/2026 | 04/03/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:**<br>&emsp;+ Triển khai một EC2 instance trong VPC tùy chỉnh.<br>&emsp;+ Cấu hình Security Groups và Route Tables.<br>&emsp;+ Xác thực quyền truy cập Internet.<br>&emsp;+ Khắc phục sự cố mạng. | 04/04/2026 | 04/04/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Thành tựu của tuần 2:

* Hiểu được kiến trúc và các thành phần của Amazon Virtual Private Cloud (VPC).

* Thiết kế và triển khai thành công một VPC tùy chỉnh gồm:
  * Public Subnet
  * Private Subnet
  * Route Tables
  * Internet Gateway

* Tìm hiểu cách CIDR blocks xác định việc phân bổ địa chỉ IP trong một VPC.

* Hiểu rõ sự khác nhau giữa:
  * Public Subnets
  * Private Subnets
  * Tài nguyên Internet-facing
  * Tài nguyên nội bộ

* Cấu hình thành công các quy tắc định tuyến để cho phép truy cập Internet cho các tài nguyên triển khai trong public subnets.

* Tìm hiểu chức năng của Security Groups như firewall ảo dạng stateful.

* Hiểu cách Network ACLs cung cấp lọc lưu lượng ở mức subnet, bao gồm:
  * Stateless packet filtering
  * Thứ tự đánh giá quy tắc
  * Quy tắc inbound và outbound

* Cấu hình thành công Security Groups để cho phép SSH an toàn trong khi hạn chế lưu lượng inbound không cần thiết.

* Tìm hiểu cách Elastic IP cấp địa chỉ public IP bền vững cho EC2 instances.

* Gắn thành công Elastic IP với một EC2 instance và xác minh kết nối từ xa.

* Thực hành khắc phục các sự cố mạng phổ biến như:
  * Thiếu Internet Gateway
  * Cấu hình Route Table không đúng
  * Quy tắc Security Group cấu hình sai
  * Hạn chế của Network ACL

* Xây dựng nền tảng mạng vững chắc cho việc triển khai các ứng dụng AWS có thể mở rộng và an toàn trong các tuần tiếp theo.