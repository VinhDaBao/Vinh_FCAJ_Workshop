---
title: "5.5. Dọn dẹp"
weight: 5
---

Sau khi hoàn thành workshop hoặc khi bạn không còn cần môi trường thực hành này nữa, việc dọn dẹp (phá hủy) toàn bộ các tài nguyên AWS là vô cùng cấp thiết. Nếu không làm điều này, bạn sẽ phải đối mặt với các khoản phí không mong muốn phát sinh liên tục trên hóa đơn AWS của mình, đặc biệt là đối với các tài nguyên tốn kém như RDS, ElastiCache và NAT Gateways.

## Sức mạnh của Terraform Destroy

Một trong những lợi ích lớn nhất của Cơ sở hạ tầng dưới dạng Mã (IaC) là việc phá bỏ một môi trường cũng dễ dàng y như lúc xây dựng nó. Thay vì phải lùng sục khắp AWS Console để xóa thủ công hàng tá tài nguyên liên kết với nhau, Terraform tự động theo dõi mọi trạng thái và sự phụ thuộc để phá hủy chúng theo đúng thứ tự an toàn.

Lệnh chính để phá bỏ toàn bộ cơ sở hạ tầng là:

```bash
terraform destroy --auto-approve
```

Tuy nhiên, trước khi bạn chạy lệnh này, có một vài điều kiện tiên quyết mà bạn **bắt buộc** phải thực hiện thủ công để đảm bảo quá trình diễn ra suôn sẻ.

## Điều kiện Tiên quyết Trước khi Phá hủy

> [!WARNING]
> **Làm trống S3 Buckets trước khi chạy Terraform Destroy**
> 
> Theo mặc định, Terraform **không thể xóa** một Amazon S3 Bucket nếu bên trong nó vẫn còn chứa dữ liệu (các objects). Đây là cơ chế an toàn để chống mất mát dữ liệu vô ý. Nếu bạn chạy lệnh `terraform destroy` khi S3 bucket chưa trống, quá trình này sẽ bị lỗi với thông báo `BucketNotEmpty`.
> 
> **Cách khắc phục:** 
> Trước khi chạy `terraform destroy`, bạn phải sử dụng AWS Console hoặc AWS CLI để xóa sạch toàn bộ nội dung bên trong S3 Media Bucket của PubliCast.
> 
> *Sử dụng AWS CLI để làm trống bucket:*
> ```bash
> aws s3 rm s3://your-publicast-media-bucket-name --recursive
> ```
> *(Thay thế `your-publicast-media-bucket-name` bằng tên bucket thực tế của bạn).*

> [!IMPORTANT]
> **Xóa các Image trong ECR Repository**
> 
> Tương tự như S3, kho lưu trữ Amazon Elastic Container Registry (ECR) thường không thể bị xóa nếu nó vẫn còn chứa các Docker images bên trong. 
> Chúng tôi đặc biệt khuyến nghị bạn truy cập vào AWS ECR Console, chọn kho lưu trữ backend và frontend của dự án, chọn tất cả các image và xóa chúng thủ công trước khi bắt đầu chạy lệnh Terraform destroy.

## Danh sách Kiểm tra Dọn dẹp (Step-by-Step Cleanup Checklist)

1.  **Làm trống S3 Buckets:** Đảm bảo không còn bất kỳ tệp media nào do người dùng tải lên bị bỏ sót bên trong bucket.
2.  **Xóa ECR Images:** Xóa tất cả các Docker image đã được đẩy lên bởi luồng CI/CD.
3.  **Thực thi Terraform Destroy:** Chạy lệnh `terraform destroy --auto-approve` trong terminal của bạn. Quá trình này có thể mất từ 10-15 phút vì AWS cần thời gian để tắt một cách an toàn các cơ sở dữ liệu RDS, cụm ElastiCache, tác vụ ECS và các thành phần mạng VPC.
4.  **Xác minh qua AWS Console:** Đăng nhập lại vào AWS Console lần cuối. Kiểm tra nhanh Bảng điều khiển EC2 (để tìm các NAT Gateway hoặc Load Balancer còn sót), RDS và ElastiCache để xác nhận rằng **hiện có 0 instances** đang chạy.
5.  **Kiểm tra Bảng Thanh toán (Billing Dashboard):** Theo dõi Bảng điều khiển Quản lý Chi phí & Thanh toán AWS của bạn trong 24-48 giờ tiếp theo để đảm bảo rằng các khoản phí phát sinh hàng ngày đã giảm về mức 0.

Bằng cách tuân thủ nghiêm ngặt các bước này, bạn sẽ giữ cho môi trường AWS của mình luôn sạch sẽ và tránh được những "cú sốc" hóa đơn ngoài ý muốn!
