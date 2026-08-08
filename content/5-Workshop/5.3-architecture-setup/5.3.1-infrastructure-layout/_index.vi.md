---
title: "5.3.1. Sơ đồ Kiến trúc Hạ tầng"
weight: 1
---

Phần này cung cấp một cái nhìn tổng quan cấp cao về toàn bộ kiến trúc mạng và hệ thống cho dự án PubliCast. Hạ tầng được cấu thành từ nhiều dịch vụ AWS hoạt động phối hợp với nhau để mang lại một ứng dụng có tính khả dụng cao, bảo mật và dễ dàng mở rộng.

## Kiến trúc Hệ thống Tổng thể

Hệ thống PubliCast được chia một cách logic thành nhiều tầng chức năng, mỗi tầng sử dụng các dịch vụ được quản lý của AWS cụ thể và các module Terraform tương ứng.

### 1. Tầng Mạng & Bảo mật (Network & Security Layer)
Nền tảng của hệ thống được xây dựng trên một Đám mây riêng ảo (VPC) bảo mật cao.
*   **VPC, Subnets & Gateways:** Cách ly hạ tầng thành các Subnets Công cộng (kết nối internet) và Subnets Riêng tư (backend bảo mật).
*   **VPC Endpoints:** Cung cấp một lối tắt trực tiếp, miễn phí tới Amazon S3 cho các tài nguyên nằm trong private subnets.
*   **Security Groups:** Đóng vai trò là tường lửa ảo để kiểm soát nghiêm ngặt lưu lượng truy cập giữa các thành phần (ví dụ: đảm bảo cơ sở dữ liệu chỉ chấp nhận kết nối từ các container ứng dụng).
*   **IAM (Identity and Access Management):** Áp dụng nguyên tắc quyền hạn tối thiểu (least privilege), chỉ cấp cho các dịch vụ những quyền mà chúng thực sự cần để hoạt động.

### 2. Tầng Cạnh & Cân bằng tải (Edge & Load Balancing Layer)
Tầng này xử lý lưu lượng truy cập xuất phát từ internet công cộng và định tuyến nó đến các dịch vụ backend chính xác.
*   **Route53:** Quản lý phân giải DNS, trỏ các tên miền tùy chỉnh đến các điểm đầu vào của chúng ta.
*   **Application Load Balancer (ALB):** Tiếp nhận các yêu cầu HTTP/HTTPS và phân phối chúng đồng đều trên các instance chứa API backend.
*   **CloudFront (CDN):** Lưu trữ bộ nhớ đệm (cache) đa phương tiện công cộng trên toàn cầu, giảm độ trễ cho người dùng cuối và giảm tải băng thông tải xuống nặng nề từ backend cốt lõi.

### 3. Tầng Tính toán & Container (Compute & Container Layer)
Đây là nơi mã nguồn ứng dụng thực sự được thực thi.
*   **Elastic Container Registry (ECR):** Lưu trữ các Docker image đã được build cho các vi dịch vụ của chúng ta (API Service, Worker Light, Worker Heavy).
*   **Elastic Container Service (ECS) on Fargate:** Chạy các container ứng dụng trong môi trường tính toán không máy chủ (serverless). Chúng ta không cần quản lý các máy chủ vật lý bên dưới. Các quy tắc Auto-scaling sẽ tự động điều chỉnh số lượng container đang chạy dựa trên mức sử dụng CPU và Bộ nhớ trong thời gian thực.

### 4. Tầng Lưu trữ & Cơ sở dữ liệu (Storage & Database Layer)
Tầng này cung cấp khả năng lưu trữ dữ liệu bền bỉ và bộ nhớ đệm tốc độ cao.
*   **Amazon RDS (MySQL):** Cơ sở dữ liệu quan hệ chính chứa dữ liệu cốt lõi của ứng dụng (Users, Workspaces, Posts, v.v.).
*   **Amazon ElastiCache (Redis):** Đóng vai trò vừa là bộ nhớ đệm trong RAM để tăng tốc độ phản hồi API, vừa là message broker cho các hàng đợi công việc nền (BullMQ).
*   **Amazon S3:** Lưu trữ đối tượng bảo mật, khả năng mở rộng cao cho toàn bộ tệp đa phương tiện do người dùng tải lên (video, hình ảnh).
*   **AWS Secrets Manager:** Lưu trữ bảo mật các mật khẩu cơ sở dữ liệu và khóa API bên thứ ba, tiêm chúng vào các container ECS lúc khởi chạy để tránh việc hardcode khóa bí mật.

### 5. Tầng Giám sát & Tự động hóa (Monitoring & Automation Layer)
Đảm bảo hệ thống luôn trong trạng thái có thể quan sát, hoạt động tốt và các lần triển khai diễn ra liền mạch.
*   **CloudWatch:** Tập trung hóa log của container và các số liệu hệ thống. Nó kích hoạt các chính sách tự động mở rộng (auto-scaling) hoặc cảnh báo khi hiệu suất suy giảm.
*   **EventBridge:** Xử lý các tác vụ theo lịch trình (cron jobs) để tự động hóa việc bảo trì hệ thống định kỳ.
*   **Đường ống CI/CD (CodePipeline):** Tự động hóa việc kiểm thử, xây dựng và triển khai trực tiếp các Docker image lên ECS mà không cần can thiệp thủ công.
