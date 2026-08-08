---
title: "5.2. Điều kiện Tiên quyết"
weight: 2
---

Trước khi triển khai hạ tầng cho dự án PubliCast, hãy đảm bảo rằng các công cụ sau đã được cài đặt và cấu hình đúng cách trên máy tính của bạn:

## Các công cụ cần thiết

1.  **AWS CLI**: Đã được cài đặt và cấu hình với thông tin xác thực AWS của bạn (Access Key ID & Secret Access Key). Đảm bảo rằng user IAM có đủ quyền để tạo các tài nguyên như VPC, ECS, RDS, S3, v.v.
2.  **Terraform**: Phiên bản >= 1.5.0. Được sử dụng để triển khai Hạ tầng dưới dạng Code (IaC).
3.  **Docker**: Để build và kiểm thử các image cục bộ nếu cần.
4.  **Git**: Dùng để quản lý mã nguồn.

## Hướng dẫn clone kho lưu trữ dự án

Clone mã nguồn chứa cấu hình Terraform của dự án về máy tính cục bộ của bạn bằng các lệnh sau:

```bash
git clone https://github.com/Nguyen-Thanh-Huy-io/FCAJ-AWS-Project.git
cd publicast-terraform/terraform/envs/staging
```

*Lưu ý: Bạn có thể thay thế `https://github.com/your-username/publicast-terraform.git` bằng URL kho lưu trữ thực tế của bạn nếu cần.*