---
title: "5.3.2. Quyết định Kiến trúc"
weight: 2
---

Phần này giải thích lý do (rationale) đằng sau các lựa chọn công nghệ then chốt được đưa ra khi thiết kế hạ tầng AWS cho PubliCast. Việc hiểu rõ *tại sao* một dịch vụ được chọn trong bối cảnh các yêu cầu kinh doanh cụ thể của chúng ta cũng quan trọng không kém việc biết *cách* triển khai nó.

PubliCast là một công cụ xuất bản mạng xã hội đa nền tảng. Kiến trúc của chúng ta phải xử lý phương tiện truyền thông độ phân giải cao, bảo vệ các thông tin xác thực bên thứ ba một cách an toàn và xử lý các khối lượng công việc không thể đoán trước. Dưới đây là cách các dịch vụ AWS đáp ứng các yêu cầu đó.

## 1. Tại sao lại dùng Amazon S3 và CloudFront? (Lưu trữ & Phân phối Media)
**Yêu cầu Kinh doanh:** Người dùng liên tục tải lên các tệp video lớn và hình ảnh độ phân giải cao dành cho các nền tảng như YouTube, Facebook và Instagram.
**Lựa chọn Kỹ thuật:** 
*   **Amazon S3:** Chúng tôi chọn S3 vì nó cung cấp khả năng mở rộng gần như vô hạn cho các khối dữ liệu nhị phân (binary blobs) khổng lồ. Nó tiết kiệm chi phí hơn nhiều so với việc lưu trữ phương tiện trên block storage (EBS) hoặc bên trong cơ sở dữ liệu quan hệ.
*   **Amazon CloudFront:** Bằng cách đặt một CDN ở phía trước S3, chúng tôi đảm bảo rằng người dùng trên các khu vực địa lý khác nhau đều có độ trễ cực thấp khi xem trước hoặc tải lên nội dung của họ, giúp cải thiện đáng kể trải nghiệm người dùng.

## 2. Tại sao lại dùng AWS Secrets Manager? (Bảo mật Thông tin xác thực)
**Yêu cầu Kinh doanh:** Để xuất bản nội dung thay mặt người dùng, PubliCast phải quản lý và sử dụng các mã khóa API bên thứ ba, token OAuth (Meta, Google) và mật khẩu cơ sở dữ liệu cực kỳ nhạy cảm.
**Lựa chọn Kỹ thuật:** 
Việc hardcode các thông tin xác thực này trong mã nguồn hoặc các biến môi trường tiêu chuẩn là một rủi ro bảo mật lớn. Chúng tôi tích hợp **AWS Secrets Manager** để mã hóa và lưu trữ tập trung các bí mật này. Tại thời điểm chạy (runtime), các tác vụ ECS sẽ tìm nạp (fetch) các bí mật này một cách an toàn bằng cách sử dụng các IAM roles, đảm bảo tuân thủ nghiêm ngặt và không rò rỉ bất kỳ thông tin xác thực nào.

## 3. Tại sao lại tách thành 3 Microservices? (Cách ly Khối lượng công việc)
**Yêu cầu Kinh doanh:** Việc tải lên và stream các tệp video nặng sang YouTube mất nhiều thời gian và tiêu tốn lượng lớn CPU/RAM. Nếu xử lý đồng bộ (synchronously), nó sẽ làm treo toàn bộ ứng dụng web đối với những người dùng đang cố gắng thực hiện các tác vụ đơn giản khác.
**Lựa chọn Kỹ thuật:** 
Chúng tôi đã tách kiến trúc nguyên khối (monolith) thành ba vi dịch vụ (microservices) chuyên biệt:
*   **API Service:** Xử lý các yêu cầu HTTP thời gian thực nhẹ nhàng (ví dụ: tải trang tổng quan dashboard). Yêu cầu CPU thấp nhưng thời gian phản hồi nhanh.
*   **Worker Light:** Xử lý các tác vụ nền nhanh gọn như gửi email thông báo.
*   **Worker Heavy:** Dành riêng cho các tác vụ đòi hỏi nhiều tài nguyên như mã hóa video và stream dữ liệu sang các mạng xã hội. 
*   **Kết quả:** Sự cách ly này đảm bảo rằng một đợt gia tăng đột biến (spike) trong các tác vụ xuất bản video sẽ không bao giờ làm sập hoặc làm chậm giao diện người dùng chính.

## 4. Tại sao lại dùng ECS trên AWS Fargate? (Tính toán Không máy chủ - Serverless)
**Yêu cầu Kinh doanh:** Lưu lượng truy cập xuất bản mạng xã hội thường có dạng gai nhọn/đột biến (ví dụ: hàng ngàn người dùng lên lịch đăng bài vào đúng 8:00 sáng Thứ Hai).
**Lựa chọn Kỹ thuật:** 
Thay vì cấp phát các máy ảo EC2 truyền thống, chúng tôi chọn **ECS trên AWS Fargate**. Fargate loại bỏ nhu cầu phải vá lỗi hoặc quản lý hệ điều hành. Quan trọng hơn, nó cho phép các vi dịch vụ của chúng ta tự động mở rộng (auto-scale) một cách nhanh chóng và độc lập dựa trên nhu cầu theo thời gian thực, đảm bảo chúng ta chỉ phải trả tiền cho chính xác lượng sức mạnh tính toán mà chúng ta sử dụng.

## 5. Tại sao lại dùng Amazon RDS và ElastiCache? (Dữ liệu & Hàng đợi Công việc)
**Yêu cầu Kinh doanh:** Hệ thống cần lưu trữ dữ liệu người dùng có cấu trúc (Workspaces, Lịch sử Bài đăng) một cách đáng tin cậy và quản lý hàng ngàn tác vụ xuất bản theo lịch trình mà không làm rơi rớt bất kỳ tác vụ nào.
**Lựa chọn Kỹ thuật:** 
*   **Amazon RDS (MySQL):** Cung cấp bộ lưu trữ quan hệ có tính khả dụng cao với tính năng tự động sao lưu hàng ngày, đảm bảo dữ liệu lõi của người dùng không bao giờ bị mất.
*   **Amazon ElastiCache (Redis):** Đóng vai trò là một message broker cực kỳ nhanh nhạy hỗ trợ cho các hàng đợi công việc BullMQ của chúng ta. Nó đảm bảo rằng mọi bài đăng theo lịch trình đều được một Worker nhận lấy một cách đáng tin cậy, xử lý theo đúng thứ tự và không bao giờ bị trùng lặp, đồng thời lưu trữ bộ nhớ đệm cho các truy vấn cơ sở dữ liệu nặng để tăng tốc độ API.