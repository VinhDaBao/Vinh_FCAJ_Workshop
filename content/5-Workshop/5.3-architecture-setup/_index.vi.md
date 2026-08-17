---
title: "5.3. Kiến trúc và Thiết lập"
weight: 3
---

Trong chương này, chúng ta sẽ đi sâu vào cơ sở hạ tầng AWS vận hành nền tảng PubliCast. Chúng ta sẽ khám phá cách Terraform được sử dụng để cấp phát và điều phối các dịch vụ AWS khác nhau thành một môi trường gắn kết, đạt chuẩn sản xuất (production-ready).

Bạn sẽ tìm hiểu về các tầng khác nhau trong kiến trúc của chúng ta, các quyết định cấu hình cụ thể được đưa ra cho từng dịch vụ, và quy trình từng bước để triển khai toàn bộ hệ thống lên tài khoản AWS của bạn.

## Triết lý Thiết kế Kiến trúc (Architectural Philosophy)

Trước khi đi sâu vào các dòng code, điều quan trọng là phải hiểu các nguyên tắc cốt lõi đã định hình nên hệ thống PubliCast:

1.  **Bảo mật từ trong Thiết kế (Security by Design - Least Privilege):** Mọi thành phần, từ luồng CI/CD cho đến các worker xử lý nền, đều hoạt động với quyền hạn tối thiểu tuyệt đối cần thiết. Các cấu hình nhạy cảm và API Key của bên thứ ba được quản lý nghiêm ngặt qua AWS Secrets Manager.
2.  **Tính Khả dụng Cao & Chịu lỗi (High Availability & Fault Tolerance):** Cơ sở hạ tầng được phân tán qua nhiều Vùng Sẵn sàng (Multi-AZ). Điều này đảm bảo rằng nếu một trung tâm dữ liệu gặp sự cố, ứng dụng vẫn hoạt động bình thường mà không bị gián đoạn.
3.  **Tối ưu hóa Chi phí (Serverless & Endpoints):** Chúng tôi tận dụng tối đa sức mạnh của điện toán máy chủ ảo (AWS Fargate) để tránh phải trả tiền cho các máy chủ rảnh rỗi. Hơn nữa, việc triển khai VPC Gateway Endpoints để định tuyến luồng dữ liệu S3 trong mạng nội bộ giúp chúng tôi tránh hoàn toàn các khoản phí xử lý dữ liệu đắt đỏ của NAT Gateway.
4.  **Xử lý Bất đồng bộ (Asynchronous Processing):** Bằng cách tách biệt máy chủ API chính khỏi các tác vụ xử lý media nặng nề thông qua Amazon SQS và EventBridge, chúng tôi đảm bảo trải nghiệm người dùng luôn mượt mà ngay cả trong những khung giờ cao điểm.

## Chiến lược Kiến trúc 5 Trụ cột (The 5-Pillar Architecture Strategy)

Cơ sở hạ tầng được chia thành 5 trụ cột logic riêng biệt, mà bạn sẽ được khám phá chi tiết trong các phần tiếp theo:

*   **Trụ cột 1: Mạng & Cách ly (Networking & Isolation).** Nền móng của hệ thống. Chúng tôi thiết lập một Đám mây Riêng Ảo (VPC) với các Mạng con Công cộng (Public Subnets) cho quyền truy cập bên ngoài và các Mạng con Riêng tư (Private Subnets) để che giấu an toàn các tài nguyên máy tính và cơ sở dữ liệu.
*   **Trụ cột 2: Tính toán & Điều phối (Compute & Orchestration).** Sử dụng Amazon ECS kết hợp với AWS Fargate để chạy các Docker containers mà không cần quản lý các máy chủ EC2 vật lý bên dưới.
*   **Trụ cột 3: Trạng thái & Lưu trữ Dữ liệu (State & Persistence).** Sự kết hợp giữa Amazon RDS (PostgreSQL) để lưu trữ dữ liệu quan hệ đáng tin cậy và Amazon ElastiCache (Redis) để làm bộ nhớ đệm tốc độ cao và trung gian truyền tin (message broker).
*   **Trụ cột 4: Lưu trữ Tệp (Storage).** Amazon S3 được sử dụng như một kho lưu trữ đối tượng có độ bền bỉ cao dành cho các tệp media (hình ảnh, video) do người dùng tải lên trước khi chúng được xuất bản lên mạng xã hội.
*   **Trụ cột 5: Giám sát & Tự động hóa (Monitoring & Automation).** Quan sát liên tục thông qua CloudWatch và định tuyến cảnh báo tự động qua SNS để giữ cho đội ngũ vận hành luôn nắm bắt được "sức khỏe" của hệ thống.

## Tại sao lại dùng Infrastructure as Code (IaC)?

Chúng tôi chọn **Terraform** làm công cụ IaC cho workshop này nhờ tính độc lập với nền tảng và khả năng quản lý trạng thái (state management) mạnh mẽ. Bằng cách định nghĩa hạ tầng dưới dạng code, chúng ta đạt được:
*   **Khả năng Tái tạo (Reproducibility):** Bạn có thể triển khai lại chính xác môi trường sản xuất đó chỉ trong vài phút, thay vì phải mất hàng giờ đồng hồ click chuột thủ công trên AWS Console.
*   **Khôi phục Thảm họa (Disaster Recovery):** Nếu hạ tầng vô tình bị xóa, Terraform có thể xây dựng lại toàn bộ hệ thống từ con số 0 chỉ bằng một câu lệnh duy nhất.
*   **Khả năng Kiểm toán (Auditability):** Mọi thay đổi đối với hạ tầng đều được theo dõi trên Git, giúp dễ dàng đánh giá lại các quy tắc bảo mật và các thay đổi về mạng lưới.

### Các Chủ đề Bao gồm trong Chương này

*   [Sơ đồ Kiến trúc Hạ tầng](5.3.1-infrastructure-layout/)
*   [Quyết định Kiến trúc](5.3.2-resource-properties/)
*   [Phân tích Terraform Modules](5.3.3-terraform-modules/)
*   [Thực thi Triển khai](5.3.4-deployment-execution/)