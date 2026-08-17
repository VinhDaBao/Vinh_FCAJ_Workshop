---
title: "5.3.3. Tham chiếu Module Terraform"
weight: 3
---

Hạ tầng của PubliCast được xây dựng bằng kiến trúc Terraform có tính module hóa cao. Việc viết cơ sở hạ tầng dưới dạng mã (IaC) mang lại sức mạnh to lớn, nhưng nếu nhồi nhét tất cả các cấu hình vào một file `main.tf` duy nhất khổng lồ thì dự án sẽ nhanh chóng trở nên không thể quản lý và rất nguy hiểm khi chạy trên môi trường thực tế (production).

Để đảm bảo hệ thống có thể dễ dàng bảo trì, mở rộng và bảo mật, chúng tôi đã nhóm các cấu hình Terraform riêng lẻ vào 5 lớp logic cốt lõi đã được định nghĩa trong Sơ đồ Kiến trúc.

## Cách tiếp cận Module (The Modular Approach)

Tại sao chúng tôi lại chọn kiến trúc module cho dự án này?
1. **Phân tách Trách nhiệm (Separation of Concerns):** Logic về mạng (VPC, Subnet) không nên bị đan xen lộn xộn với các cấu hình cơ sở dữ liệu. Module cho phép chúng ta cô lập các trách nhiệm này.
2. **Thu hẹp Phạm vi Ảnh hưởng (Blast Radius Reduction):** Nếu có lỗi cấu hình xảy ra trong module `ci` (Continuous Integration), nó sẽ không vô tình phá hủy module `database` vì các định nghĩa và tệp trạng thái (state) của chúng đã được tách biệt một cách hợp lý.
3. **Khả năng Tái sử dụng (Reusability):** Module `compute` có thể được tái sử dụng để khởi tạo cả máy chủ API chính lẫn các Worker chạy nền chỉ bằng cách truyền vào các biến (variables) khác nhau, giúp giảm thiểu đáng kể việc lặp code.

## Sơ đồ Phụ thuộc Module (Module Dependency Graph)

Trong hệ thống của chúng tôi, các module không tồn tại độc lập. Chúng tạo thành một biểu đồ phụ thuộc có hướng (directed dependency graph), nơi các module nền tảng ở cấp thấp sẽ cung cấp các giá trị đầu ra cần thiết (như ID và ARN) cho các module ứng dụng ở cấp cao hơn.
*   Module **Network** (Mạng) bắt buộc phải được triển khai đầu tiên, vì tất cả các tài nguyên khác đều cần một VPC và các Mạng con (Subnets) để có nơi trú ngụ.
*   Module **Database** và **Storage** sẽ được triển khai tiếp theo.
*   Module **Compute** được triển khai cuối cùng, do các container ECS yêu cầu cả Mạng (để biết chúng sẽ chạy ở đâu) và Cơ sở dữ liệu (để kết nối tới PostgreSQL và Redis).

> [!TIP]
> **Tham chiếu Mã nguồn (Source Code)**
> Vì các file Terraform có thể rất dài, chúng tôi hạn chế việc đưa các khối code khổng lồ trực tiếp vào tài liệu này để tránh gây rối mắt. Thay vào đó, đối với mỗi module ở các trang tiếp theo, chúng tôi tập trung giải thích khái niệm cốt lõi của các khối tài nguyên `aws_*` đã được sử dụng. Vì bạn đã clone dự án về máy, bạn có thể xem trực tiếp toàn bộ source code thực tế bằng cách mở thư mục `terraform/modules/` trong Code Editor của bạn.

## Cách đọc Hướng dẫn này

Đối với mỗi lớp trong số 5 lớp được liệt kê ở thanh menu bên trái, bạn sẽ tìm thấy:
*   **Mục đích:** Giải thích rõ ràng về những gì module đó sẽ thực hiện.
*   **Tài nguyên Terraform cốt lõi:** Các khối tài nguyên `aws_*` cụ thể được cấp phát (ví dụ: `aws_vpc`, `aws_ecs_cluster`).
*   **Minh họa trực quan:** Một ảnh chụp màn hình của AWS Console để giúp bạn xác minh rằng mã Terraform đã thực thi chính xác và tài nguyên thực sự đã được tạo trên mây.
