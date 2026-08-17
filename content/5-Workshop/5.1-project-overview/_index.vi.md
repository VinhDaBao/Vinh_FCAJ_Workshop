---
title : "Tổng quan dự án"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

## Tóm tắt Điều hành (Executive Summary)
PubliCast là một nền tảng quản lý nội dung và mạng xã hội toàn diện, được xây dựng hoàn toàn trên nền tảng điện toán đám mây (Cloud-native). Sứ mệnh của dự án là thu hẹp khoảng cách giữa những nhà sáng tạo nội dung và sự phức tạp của các thuật toán mạng xã hội. Bằng cách tập trung mọi thao tác vào một không gian làm việc (workspace) duy nhất, PubliCast trao quyền cho người dùng dễ dàng quản lý nội dung, điều phối đội ngũ, xử lý tương tác khách hàng và theo dõi phân tích chéo trên nhiều nền tảng mà không phải chịu cảnh chuyển đổi liên tục giữa các ứng dụng khác nhau.

## Đặt vấn đề (The Problem Statement)
Quản lý sự hiện diện kỹ thuật số trên nhiều nền tảng ngày càng trở nên phức tạp. Các đội ngũ Marketing và nhà sáng tạo hiện đại đang phải đối mặt với nhiều điểm nghẽn nghiêm trọng:
*   **Quy trình làm việc phân mảnh:** Việc liên tục chuyển đổi qua lại giữa Facebook, Instagram, TikTok và YouTube để soạn bài và trả lời bình luận gây lãng phí rất nhiều thời gian.
*   **Giới hạn tốc độ & Sự ổn định (Rate Limiting):** Việc ồ ạt đăng tải các tệp media nặng hoặc gọi API đồng thời thường xuyên dẫn đến ứng dụng bị sập, hoặc tệ hơn là tài khoản bị các mạng xã hội tạm khóa do vi phạm tốc độ truy cập.
*   **Thiếu tính cộng tác:** Các công cụ truyền thống thiếu tính minh bạch về việc ai đang lên lịch bài nào, dẫn đến sự chồng chéo nội dung và thông điệp truyền thông không nhất quán.

## Giải pháp của PubliCast (The PubliCast Solution)
PubliCast giải quyết triệt để các thách thức trên thông qua một kiến trúc đám mây mạnh mẽ được triển khai trên AWS. Nền tảng cốt lõi cung cấp:

*   **Trung tâm Nội dung Thống nhất (Unified Content Hub):** Một bảng điều khiển duy nhất để nháp, đánh giá và phê duyệt nội dung cho nhiều nền tảng cùng lúc.
*   **Động cơ Lên lịch Lai (Hybrid Scheduling Engine):** Tận dụng sức mạnh của AWS EventBridge và Amazon SQS, nền tảng có khả năng xử lý hàng ngàn tác vụ hẹn giờ. Hệ thống hoạt động như một "Người điều phối" thông minh để xử lý trơn tru các tác vụ nền, loại bỏ hoàn toàn rủi ro bị khóa API.
*   **Tích hợp OAuth Liền mạch:** Kết nối bảo mật tuyệt đối với Meta (Facebook/Instagram), Google (YouTube) và TikTok thông qua việc quản lý thông tin xác thực cấp doanh nghiệp với AWS Secrets Manager.
*   **Phân tích cấp Doanh nghiệp (Enterprise-Grade Analytics):** Hợp nhất các chỉ số tương tác (lượt thích, chia sẻ, lượt xem) trên các mạng xã hội để đo lường chính xác hiệu quả đầu tư (ROI).

## Đối tượng Mục tiêu (Target Audience)
Nền tảng này được thiết kế chuyên biệt để phục vụ:
*   **Digital Agencies:** Những công ty cần quản lý hàng tá tài khoản khách hàng cùng lúc.
*   **In-house Marketing Teams:** Các đội ngũ nội bộ đòi hỏi quy trình phê duyệt nghiêm ngặt và lưu vết nhật ký hoạt động (audit logs).
*   **Independent Creators/Freelancers:** Những nhà sáng tạo độc lập cần một hòm thư thống nhất và khả năng lên lịch tự động để tiết kiệm thời gian.

## Sơ đồ Kiến trúc Cấp cao (High-Level Architecture)
Dưới đây là sơ đồ kiến trúc tổng quan của hệ thống PubliCast, mô tả chi tiết cách các vi dịch vụ tương tác với nhau trên hạ tầng AWS. Hệ thống được thiết kế với độ sẵn sàng cao và khả năng mở rộng mạnh mẽ, bao gồm Frontend (Next.js), Backend API (Node.js), Database (RDS PostgreSQL), Cache/Message Broker (ElastiCache Redis), Object Storage (S3), và các Worker Queue xử lý nền thông minh.

{{< img "images/Workshop/services/architecture.drawio.png" "overview" >}}

## Mạch trình bày (Presentation Flow)
Buổi trình bày này sẽ hướng dẫn bạn qua toàn bộ vòng đời của dự án PubliCast:
1.  **Tầm nhìn Sản phẩm:** Hiểu rõ giá trị cốt lõi mà dự án mang lại.
2.  **Thiết lập Kiến trúc:** Đi sâu vào 5 trụ cột của hạ tầng AWS được triển khai qua Terraform.
3.  **CI/CD Pipeline:** Tự động hóa quy trình triển khai với AWS CodePipeline và CodeBuild.
4.  **Vận hành & Giám sát:** Theo dõi log, cảnh báo lỗi và các phương pháp bảo mật tốt nhất.
5.  **Tổng kết & Dọn dẹp:** Những phản ngẫm cuối cùng và hướng dẫn dọn dẹp tài nguyên.
