---
title: "5.5. Dọn dẹp"
weight: 5
---

Sau khi hoàn thành workshop hoặc khi bạn không còn cần môi trường thực hành này nữa, việc dọn dẹp (phá hủy) các tài nguyên AWS là cực kỳ quan trọng để tránh bị tính phí không mong muốn.

## Hướng dẫn dọn dẹp từng bước

Chúng ta sẽ sử dụng Terraform để phá bỏ toàn bộ cơ sở hạ tầng đã tạo. Lệnh chính là:

```bash
terraform destroy --auto-approve
```

## Lưu ý Cảnh báo Quan trọng

> [!WARNING]
> **Làm trống S3 Buckets trước khi chạy Terraform Destroy**
> 
> Theo mặc định, Terraform **không thể xóa** một Amazon S3 Bucket nếu bên trong nó vẫn còn chứa dữ liệu (các objects). Nếu bạn chạy lệnh `terraform destroy` khi S3 bucket chưa trống, quá trình này sẽ bị lỗi với thông báo `BucketNotEmpty`.
> 
> **Cách khắc phục:** 
> Trước khi chạy `terraform destroy`, bạn phải sử dụng AWS Console hoặc AWS CLI để xóa sạch toàn bộ nội dung bên trong S3 Media Bucket của PubliCast.
> 
> *Sử dụng AWS CLI để làm trống bucket:*
> ```bash
> aws s3 rm s3://your-publicast-media-bucket-name --recursive
> ```
> *(Thay thế `your-publicast-media-bucket-name` bằng tên bucket thực tế của bạn).* Khi bucket đã trống, lệnh `terraform destroy` sẽ thực thi một cách trơn tru.
