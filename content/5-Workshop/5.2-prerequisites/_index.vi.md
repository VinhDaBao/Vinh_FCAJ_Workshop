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

## Tạo Chứng chỉ SSL/TLS cho CloudFront bằng ACM (Bắt buộc)

Để website có thể truy cập thông qua giao thức HTTPS với domain riêng `publicast.trinhquoccongvinh.id.vn`, hệ thống sử dụng **AWS Certificate Manager (ACM)** để tạo và quản lý chứng chỉ SSL/TLS. Đây là điều kiện tiên quyết trước khi chạy Terraform.

### Bước 1: Xác định domain cần cấp certificate

> [!NOTE]
> Trong hướng dẫn này, `trinhquoccongvinh.id.vn` được sử dụng làm domain ví dụ. Khi thực hành, bạn **BẮT BUỘC** phải thay thế tên miền này bằng domain thực tế mà bạn sở hữu.

Domain chính được quản lý là: `trinhquoccongvinh.id.vn` *(thay bằng domain của bạn)*
Hệ thống sử dụng một subdomain dành cho ứng dụng: `publicast.trinhquoccongvinh.id.vn` *(thay bằng subdomain của bạn)*

Do CloudFront yêu cầu certificate phải chứa chính xác domain được cấu hình trong Alternate Domain Name (CNAME), certificate cần được cấp cho: `publicast.trinhquoccongvinh.id.vn`

### Bước 2: Tạo certificate trên AWS Certificate Manager

Truy cập **AWS Certificate Manager (ACM)** và chuyển sang region: `US East (N. Virginia) – us-east-1`
> [!IMPORTANT]
> Đây là yêu cầu bắt buộc khi sử dụng certificate với Amazon CloudFront.

Tại ACM, chọn **Request a certificate** từ trang tổng quan.
{{< img "images/Workshop/services/acm-dashboard.png" "ACM Dashboard" >}}

Chọn loại chứng chỉ là: **Request a public certificate**
{{< img "images/Workshop/services/acm-cert-type.png" "Request a public certificate" >}}

Sau đó nhập các thông tin sau:
* Fully qualified domain name: `publicast.trinhquoccongvinh.id.vn`
* Validation method: DNS validation - recommended

Nhấn **Request** để gửi yêu cầu.
{{< img "images/Workshop/services/acm-request-details.png" "ACM Request Details" >}}

### Bước 3: Xác thực domain bằng DNS

Sau khi tạo certificate, ACM cung cấp một DNS validation record dạng CNAME. 
Từ màn hình chi tiết của Certificate, bạn sẽ thấy trạng thái là **Pending validation** kèm theo thông tin CNAME.
{{< img "images/Workshop/services/acm-pending-validation.png" "ACM Pending Validation" >}}

**Cách tự động thêm vào Route 53:**
Vì chúng ta đang quản lý domain bằng AWS Route 53, bạn chỉ cần nhấn nút **Create records in Route 53**. 
Một màn hình xác nhận sẽ hiện ra, bạn tiếp tục nhấn **Create records**. AWS sẽ tự động gắn bản ghi CNAME này vào Hosted Zone của bạn để xác minh quyền sở hữu domain.
{{< img "images/Workshop/services/acm-create-records.png" "Create Route 53 Records" >}}

Để chắc chắn, bạn có thể truy cập dịch vụ **Route 53 > Hosted zones** và xem danh sách các bản ghi (Records). Bạn sẽ thấy bản ghi CNAME mới vừa được tự động thêm vào.
{{< img "images/Workshop/services/route53-acm-record.png" "Route 53 ACM Record" >}}

### Bước 4: Kiểm tra trạng thái certificate

Certificate chỉ được sử dụng cho CloudFront sau khi đã ở trạng thái **`ISSUED`** (Màu xanh lá). Quá trình xác thực DNS thường mất từ vài phút đến vài chục phút. Bạn hãy làm mới trang để kiểm tra.

### Bước 5: Cấu hình certificate cho CloudFront

Trong Terraform, ARN của certificate được truyền vào biến:
```hcl
acm_certificate_arn = "arn:aws:acm:us-east-1:ACCOUNT_ID:certificate/..."
```

CloudFront Distribution sử dụng certificate thông qua cấu hình:
```hcl
viewer_certificate {
  acm_certificate_arn      = var.acm_certificate_arn
  ssl_support_method       = "sni-only"
  minimum_protocol_version = "TLSv1.2_2021"
}
```

Đồng thời, Alternate Domain Name của CloudFront được cấu hình:
```hcl
aliases = [
  var.domain_name
]
```
với `domain_name = "publicast.trinhquoccongvinh.id.vn"`
Như vậy, certificate và CloudFront sử dụng cùng một hostname.

### Bước 6: Kết quả

Sau khi certificate được cấp và CloudFront được cấu hình thành công, người dùng có thể truy cập ứng dụng thông qua: `https://publicast.trinhquoccongvinh.id.vn`

Kết nối HTTPS được CloudFront xử lý bằng certificate do AWS Certificate Manager cung cấp, giúp mã hóa dữ liệu giữa người dùng và hệ thống.

> [!WARNING]
> **Lưu ý quan trọng:** Certificate sử dụng cho CloudFront **phải được tạo tại region `us-east-1`**, ngay cả khi các tài nguyên backend như VPC, ECS, RDS, ElastiCache và S3 của hệ thống được triển khai tại region khác, chẳng hạn `ap-southeast-2`.
> 
> Ngoài ra, certificate phải chứa đúng domain được cấu hình trong CloudFront. Ví dụ: `trinhquoccongvinh.id.vn` không thể thay thế cho `publicast.trinhquoccongvinh.id.vn` nếu certificate không chứa subdomain này trong Common Name hoặc Subject Alternative Names.