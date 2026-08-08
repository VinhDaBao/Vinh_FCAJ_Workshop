---
title: "5.3.3.4. Lớp Lưu trữ & Cơ sở dữ liệu"
weight: 4
---

Lớp này cung cấp "trí nhớ dài hạn" cho ứng dụng của chúng ta. Trong khi các container ECS của chúng ta là "stateless" (không lưu trạng thái - nghĩa là nếu chúng bị sập, mọi dữ liệu bên trong sẽ bị mất sạch), thì lớp này cung cấp khả năng lưu trữ dữ liệu vĩnh viễn, bộ đệm siêu tốc và quản lý an toàn các thông tin xác thực nhạy cảm.

## Amazon S3 Media Bucket (`s3_cloudfront`)
**Mục đích:** Lưu trữ đối tượng (object storage) có độ bền cực cao được sử dụng để lưu vô thời hạn số lượng lớn video và hình ảnh do người dùng tải lên. Không giống như một ổ cứng truyền thống có thể bị đầy, S3 cung cấp dung lượng lưu trữ gần như vô hạn.

> [!IMPORTANT]
> **Sử dụng Tên Bucket Duy nhất Toàn cầu**
> Tên bucket của Amazon S3 phải là duy nhất trên toàn cầu đối với tất cả các tài khoản AWS. Khi triển khai mã Terraform này, hãy đảm bảo thay đổi tên bucket mặc định (ví dụ: `publicast-media-storage`) thành một cái tên duy nhất cho môi trường của riêng bạn.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_s3_bucket`: Tạo các bucket lưu trữ cơ sở cho ứng dụng. Như bạn thấy trên console, dự án của chúng ta sử dụng 3 bucket riêng biệt, mỗi cái có một mục đích cụ thể:
    1.  **Bucket `frontend`**: Được phân phối bởi CloudFront, bucket này lưu trữ các tệp tĩnh (HTML, CSS, JS) đã được biên dịch của ứng dụng React (Single Page Application).
    2.  **Bucket `backend-storage`**: Cũng được phân phối bảo mật qua CloudFront, đây là "ổ cứng" khổng lồ chuyên dụng để lưu trữ các tệp media do người dùng tải lên (video, ảnh thu nhỏ, ảnh đại diện).
    3.  **Bucket `pipeline-artifacts`**: (Được tạo bởi module CI/CD) Bucket này được AWS CodePipeline sử dụng nội bộ để lưu trữ tạm thời các tệp build trung gian trong quá trình tự động hóa triển khai mã nguồn.
    
    {{< img "images/Workshop/services/s3-bucket.png" "AWS Console - S3 Buckets List" >}}
    <p align="center"><i>AWS Console - S3 Bucket</i></p>
    
*   `aws_s3_bucket_public_access_block`: Một biện pháp bảo mật tối quan trọng nhằm chặn cưỡng bức tất cả các Danh sách kiểm soát truy cập (ACL) và chính sách bucket công khai. Điều này đảm bảo bucket bị khóa chặt ở cấp độ tài khoản, không ai có thể vô tình làm lộ toàn bộ bucket ra công cộng.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `block_public_acls = true` & `block_public_policy = true`: Mã hóa cứng các thiết lập phong tỏa này.
*   `aws_s3_bucket_policy`: Chúng ta áp dụng một chính sách IAM JSON nghiêm ngặt cho bucket. Nhìn vào mã JSON trên console, chúng ta có thể thấy chính xác cách cơ chế Zero-Trust này hoạt động:
    *   **`Principal`**: Khai báo với AWS rằng chỉ có dịch vụ `cloudfront.amazonaws.com` mới được phép đứng ra yêu cầu lấy file.
    *   **`Action`**: Chúng ta chỉ cấp quyền `s3:GetObject` (đọc file). Không ai có thể xóa hay ghi đè file qua đường này.
    *   **`Condition`**: Ổ khóa cuối cùng. Nó kiểm tra đối chiếu bắt buộc rằng yêu cầu phải xuất phát từ đúng `AWS:SourceArn` của CloudFront Distribution thuộc dự án chúng ta (không phải CloudFront của người khác).
    
    Điều này đảm bảo rằng media không thể được truy cập trực tiếp qua URL S3 thô, ngăn chặn việc đánh cắp băng thông và ép toàn bộ lưu lượng phải đi qua CDN tốc độ cao.
    
    {{< img "images/Workshop/services/s3-policy.png" "AWS Console - S3 Bucket Policy" >}}
    <p align="center"><i>AWS Console - S3 Bucket Policy</i></p>

📁 **Source Code:** Mở file `terraform/modules/s3_cloudfront/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## Relational Database Service (`rds`)
**Mục đích:** Một dịch vụ cơ sở dữ liệu quan hệ được quản lý hoàn toàn, có tính sẵn sàng cao. Chúng ta dùng nó để lưu trữ dữ liệu có cấu trúc đòi hỏi các truy vấn phức tạp và có tính liên kết chặt chẽ, chẳng hạn như thông tin Người dùng, Bài đăng, và Bình luận.

Trước khi có thể khởi tạo cơ sở dữ liệu thực sự, AWS yêu cầu chúng ta phải xác định chính xác *vị trí* an toàn trong mạng mà nó được phép tồn tại. Do đó, cấu hình Terraform được chia thành hai bước logic: xác định ranh giới mạng trước, sau đó mới triển khai database vào bên trong.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_db_subnet_group`: Nhóm các Private Subnets của chúng ta lại với nhau. Điều này báo cho RDS biết chính xác những vùng mạng cô lập nào mà nó được phép triển khai các node cơ sở dữ liệu.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `subnet_ids`: Mảng chứa các ID của Private Subnet được lấy từ lớp VPC. Như bạn có thể thấy trong bức ảnh, thiết lập này đã giới hạn thành công không gian hoạt động của database vào đúng hai mạng con `publicast-staging-private-0` và `publicast-staging-private-1` trải dài trên 2 Availability Zone khác nhau, đảm bảo tuyệt đối sự an toàn mạng và tính sẵn sàng cao (High Availability).
    
    {{< img "images/Workshop/services/rds-subnet.png" "AWS Console - RDS Subnet Group" >}}
    <p align="center"><i>AWS Console - RDS Subnet Group</i></p>
    
*   `aws_db_instance`: Định nghĩa cấu hình cơ sở dữ liệu cốt lõi. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `engine = "mysql"` & `instance_class = "db.t3.micro"`: Định nghĩa phần mềm và phần cứng chạy cơ sở dữ liệu. Chúng ta sử dụng `t3.micro` để tối ưu chi phí cho workshop.
        *   `allocated_storage`: Dung lượng đĩa khởi điểm tính bằng Gigabyte.
        *   `vpc_security_group_ids`: Đính kèm tường lửa khắt khe `rds-sg` chỉ cho phép truy cập từ ECS.
        *   `multi_az = false`: Đối với môi trường production thực tế, giá trị này phải là `true` để đảm bảo tính sẵn sàng cao. Tuy nhiên, để tiết kiệm chi phí trong quá trình thực hành workshop, chúng ta chủ động tắt tính năng bản sao dự phòng này.
        *   `skip_final_snapshot = true`: Đối với môi trường workshop hoặc dev, chúng ta đặt thiết lập này để Terraform có thể phá hủy DB nhanh chóng mà không cần chờ 15 phút để chụp bản sao lưu cuối cùng.
    
    {{< img "images/Workshop/services/rds-instance.png" "AWS Console - RDS Instance" >}}
    <p align="center"><i>AWS Console - RDS Instance</i></p>

📁 **Source Code:** Mở file `terraform/modules/rds/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## Amazon ElastiCache (`elasticache`)
**Mục đích:** Một kho lưu trữ dữ liệu Redis chạy hoàn toàn trên RAM. Vì tốc độ đọc từ RAM nhanh hơn gấp nhiều lần so với đọc từ ổ đĩa database truyền thống, chúng ta dùng Redis để cache (lưu đệm) các phản hồi API và cung cấp năng lượng cho các hàng đợi công việc nền BullMQ.

> [!TIP]
> **Cách tìm Cluster Redis của bạn**
> Trong giao diện ElastiCache, có thể bạn sẽ không thấy cluster của mình ngay lập tức. Hãy nhìn sang thanh menu bên trái, dưới mục **Resources**, bấm chọn **Redis OSS caches** để thấy các thành phần đang chạy nhé.

{{< img "images/Workshop/services/elasticache-menu.png" "AWS Console - ElastiCache OSS Menu" >}}
<p align="center"><i>AWS Console - ElastiCache OSS Menu</i></p>

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_elasticache_subnet_group`: Đặt cluster Redis an toàn trong Private Subnets cùng với cơ sở dữ liệu RDS.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `subnet_ids`: Dùng chính các ID của Private Subnet giống như RDS.
    
    {{< img "images/Workshop/services/elasticache-subnet.png" "AWS Console - ElastiCache Subnet Group" >}}
    <p align="center"><i>AWS Console - ElastiCache Subnet Group</i></p>
    
*   `aws_elasticache_replication_group`: Triển khai công cụ Redis. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `engine = "redis"` & `node_type = "cache.t3.micro"`: Định nghĩa phần mềm và phần cứng cho cache. Tương tự như RDS, chúng ta dùng instance `micro` để tối ưu chi phí cho workshop.
        *   `security_group_ids`: Gắn tường lửa `redis-sg` để chặn mọi truy cập trái phép.
        *   `num_cache_clusters = 1` & `automatic_failover_enabled = false`: Đối với môi trường production thực tế, chúng ta sẽ dùng nhiều cluster và bật tính năng failover để các background worker không bao giờ bị rớt task nếu có sự cố. Tuy nhiên, trong workshop này, chúng ta chủ động giới hạn ở 1 node và tắt failover để tiết kiệm chi phí.
    
    {{< img "images/Workshop/services/elasticache-cluster.png" "AWS Console - ElastiCache Cluster" >}}
    <p align="center"><i>AWS Console - ElastiCache Cluster</i></p>

📁 **Source Code:** Mở file `terraform/modules/elasticache/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## AWS Secrets Manager (`secrets_manager`)
**Mục đích:** Lưu trữ, mã hóa và xoay vòng an toàn các thông tin xác thực nhạy cảm (như mật khẩu database và các khóa API của bên thứ ba) mà không để lộ chúng trong code hoặc dưới dạng văn bản gốc (plain text).

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_secretsmanager_secret`: Tạo bộ chứa logic (tên và mô tả) của bí mật trong AWS.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name` & `description`: Dán nhãn cho chiếc hộp bí mật để quản trị viên biết nó dùng làm gì.
    
    {{< img "images/Workshop/services/secrets-list.png" "AWS Console - Secrets Manager List" >}}
    <p align="center"><i>AWS Console - Secrets Manager List</i></p>
    
*   `aws_secretsmanager_secret_version`: Tài nguyên này chứa dữ liệu nhạy cảm thực sự. Như bạn có thể thấy trong bức ảnh bên dưới, thay vì tạo ra hàng tá các bí mật lẻ tẻ khác nhau, chúng ta gom tất cả chúng lại vào một két sắt duy nhất.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `secret_id`: Ràng buộc dữ liệu vào bộ chứa logic vừa tạo ở trên.
        *   `secret_string`: Chúng ta sử dụng hàm `jsonencode()` của Terraform để định dạng một cách an toàn một bản đồ (map) chứa tất cả các thông tin xác thực của dự án (như `DB_PASSWORD`, `REDIS_HOST`, và `YOUTUBE_CLIENT_ID`) thành chuỗi JSON duy nhất mà bạn đang thấy trên ảnh. Trong quá trình triển khai, Terraform đẩy chuỗi này lên AWS dưới dạng mã hóa. Các ECS task sau này sẽ lấy chính xác các cặp key-value này và nạp thẳng vào ứng dụng một cách an toàn khi chạy (runtime).
    
    {{< img "images/Workshop/services/secrets-value.png" "AWS Console - Secrets Manager Value" >}}
    <p align="center"><i>AWS Console - Secrets Manager Value</i></p>

📁 **Source Code:** Mở file `terraform/modules/secrets_manager/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.
