---
title: "Tham chiếu Module Terraform"
weight: 3
---

Hạ tầng của PubliCast được xây dựng bằng kiến trúc Terraform có tính module hóa cao. Để hiểu cách hệ thống được triển khai, chúng tôi đã nhóm các module Terraform riêng lẻ vào 5 lớp logic cốt lõi được định nghĩa trong Bố cục Kiến trúc.

> [!TIP]
> **Tham chiếu Mã nguồn (Source Code)**
> Vì các file Terraform có thể rất dài, chúng tôi tránh việc đặt các ảnh chụp màn hình code lớn ở đây. Thay vào đó, đối với mỗi module bên dưới, chúng tôi cung cấp lời giải thích về mặt khái niệm của các khối tài nguyên `aws_*` chính được sử dụng. Vì bạn đã clone dự án về máy, bạn có thể xem trực tiếp toàn bộ source code bằng cách mở thư mục `terraform/modules/` trong Code Editor của bạn (ví dụ: VSCode).

Đối với mỗi dịch vụ được triển khai, bạn sẽ tìm thấy giải thích về mục đích của nó, tổng quan về cấu trúc code Terraform cốt lõi, và một ảnh chụp màn hình của AWS Console để xác minh việc khởi tạo dịch vụ đó.
