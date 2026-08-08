---
title: "5.3.3.2. Lớp Biên & Cân bằng tải"
weight: 2
---

Lớp này đóng vai trò là "cửa trước" cho ứng dụng PubliCast. Nó chịu trách nhiệm tiếp nhận toàn bộ lưu lượng truy cập toàn cầu từ internet, lọc chúng và định tuyến một cách hiệu quả đến các container backend đang ẩn mình. Hãy coi lớp này giống như quầy lễ tân và hệ thống thang máy siêu tốc của tòa nhà ảo của chúng ta.

## Route53 (`route53`)
**Mục đích:** Hoạt động như một dịch vụ Hệ thống Phân giải Tên miền (DNS) có tính sẵn sàng cao để dịch các tên dễ nhớ (như `api.publicast.app`) thành địa chỉ IP máy móc hoặc các endpoint của AWS. Nếu không có Route53, người dùng sẽ phải gõ những đường dẫn URL AWS cực kỳ phức tạp để truy cập ứng dụng của chúng ta.

> [!IMPORTANT]
> **Sử dụng Tên miền của riêng bạn**
> Nếu bạn đang thực hành theo workshop này, hãy nhớ thay thế các giá trị như `api.publicast.app` bằng tên miền mà bạn đã đăng ký trong các biến của Terraform.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_route53_zone`: Quản lý "Hosted Zone" (Vùng lưu trữ) cho tên miền tùy chỉnh của chúng ta. Hosted Zone về cơ bản là một chiếc hộp chứa tất cả các bản ghi DNS (luật định tuyến) cho một tên miền cụ thể.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name`: Tên miền gốc của bạn (ví dụ: `publicast.app`).
    
    {{< img "images/Workshop/services/route53-zone.png" "AWS Console - Route53 Hosted Zone" >}}
    <p align="center"><i>AWS Console - Route53 Hosted Zone</i></p>
    
*   `aws_route53_record`: Route53 không chỉ dành cho việc định tuyến nội bộ trong AWS; nó còn cực kỳ quan trọng để xác minh tên miền của chúng ta với các nền tảng mạng xã hội và email của bên thứ ba. Như bạn thấy trên console, chúng ta định nghĩa nhiều loại bản ghi:
    *   **Bản ghi A (Alias):** Dùng để ánh xạ tên miền trực tiếp đến tên miền động của Application Load Balancer và CloudFront CDN. Chúng ta dùng khối `alias` đặc biệt của AWS thay vì hardcode IP, đảm bảo rằng nếu Load Balancer đổi IP, quá trình định tuyến vẫn hoạt động bình thường.
    *   **Bản ghi TXT (Tích hợp Mạng xã hội):** Dùng để xác minh quyền sở hữu tên miền. Ví dụ: chúng ta thêm một bản ghi TXT chứa mã `tiktok-developers-site-verification=...` để chứng minh với TikTok (hoặc Meta) rằng chúng ta sở hữu tên miền này, từ đó họ mới cấp quyền sử dụng các API đăng nhập (OAuth) của họ.
    *   **Bản ghi MX & TXT (Bảo mật Email):** Để đảm bảo các email gửi từ ứng dụng (qua hệ thống như Resend hoặc AWS SES) không bị rơi vào hộp thư rác (spam) của người dùng, chúng ta cấu hình các bản ghi `MX` để định tuyến thư, và các bản ghi `TXT` cụ thể (như `_domainkey` và `_dmarc`) cho các chính sách xác thực email SPF, DKIM, và DMARC.
    
    {{< img "images/Workshop/services/route53-record.png" "AWS Console - Route53 Record" >}}
    <p align="center"><i>AWS Console - Route53 Record</i></p>

📁 **Source Code:** Mở file `terraform/modules/route53/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## Application Load Balancer (`alb`)
**Mục đích:** Phân phối lưu lượng ứng dụng đi vào chia đều qua nhiều đích đến (các ECS container của chúng ta) trải rộng trên nhiều Availability Zones. Nếu một container bị quá tải do nhận quá nhiều request hoặc bị sập, ALB sẽ tự động chuyển hướng các truy cập mới đến những container đang khỏe mạnh.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_lb`: Đây chính là bản thân bộ cân bằng tải. Nó được thiết kế rõ ràng để đối mặt trực tiếp với internet.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `load_balancer_type = "application"`: Nói cho AWS biết nó cần phải hiểu sâu giao thức web HTTP/HTTPS (Layer 7).
        *   `internal = false`: Đảm bảo ALB được cấp các địa chỉ IP công cộng.
        *   `subnets`: Ràng buộc ALB vào các Public Subnets của chúng ta để nó có thể nhận kết nối từ thế giới bên ngoài.
        *   `security_groups`: Gắn tường lửa `alb-sg` cụ thể vào nó.
    
    {{< img "images/Workshop/services/alb.png" "AWS Console - Application Load Balancer" >}}
    <p align="center"><i>AWS Console - Application Load Balancer</i></p>
    
*   `aws_lb_target_group`: Target Group (Nhóm mục tiêu) sẽ cho bộ cân bằng tải biết *cần phải gửi traffic đi đâu*. Vì chúng ta dùng AWS Fargate (công nghệ container không máy chủ), nên không có các máy chủ EC2 truyền thống để đăng ký. Các task Fargate được gán các giao diện mạng (ENI) có IP riêng (private IP).
    
    {{< img "images/Workshop/services/tg.png" "AWS Console - Target Groups List" >}}
    <p align="center"><i>AWS Console - Danh sách Target Groups</i></p>

    *   **Bên trong Chi tiết Target Group:** Như bạn thấy trong ảnh chi tiết bên dưới, target group của chúng ta đang ghi nhận 2 target "Healthy" (khỏe mạnh) chạy ở cổng 3000, trải đều một cách an toàn trên 2 Availability Zones khác nhau để đảm bảo tính sẵn sàng cao.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `target_type = "ip"`: Cài đặt sống còn cho Fargate. Như hiển thị ở tab "Registered targets", ALB định tuyến trực tiếp đến các IP nội bộ của container (ví dụ: `10.0.x.x`) chứ không phải ID máy chủ.
        *   `vpc_id`, `port`, `protocol`: Định nghĩa cấu hình mạng cơ sở (HTTP ở cổng 3000, nơi ứng dụng Node.js của chúng ta đang chạy).
        *   `health_check` block: Cấu hình đường dẫn URL (ví dụ: `/health`), khoảng thời gian (interval) và thời gian chờ (timeout). ALB sẽ liên tục ping đường dẫn này để xác minh xem container đã khỏe mạnh và sẵn sàng nhận traffic hay chưa.
    
    {{< img "images/Workshop/services/tg-detail.png" "AWS Console - Target Group Details" >}}
    <p align="center"><i>AWS Console - Chi tiết Target Group</i></p>
    
*   `aws_lb_listener`: Listener là các tiến trình làm nhiệm vụ "lắng nghe" các yêu cầu kết nối. Như bạn thấy trên giao diện, chúng ta tạo listener cho cả HTTP (Cổng 80) và HTTPS (Cổng 443).
    *   **Bên trong Chi tiết ALB Listener & Rules:** 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `port` & `protocol`: Định nghĩa cổng lắng nghe (80 hoặc 443).
        *   `certificate_arn` (cho HTTPS): Kết nối listener với một chứng chỉ SSL từ AWS Certificate Manager (ACM) (ví dụ: `*.domain.com`) để mã hóa bảo mật.
        *   `default_action` block: Ở cả hai cổng, chúng ta đặt `type = "forward"` và cung cấp `target_group_arn` để chuyển tiếp thẳng lưu lượng truy cập tới các container thông qua target group.
    
    {{< img "images/Workshop/services/listener.png" "AWS Console - ALB Listener Details & Rules" >}}
    <p align="center"><i>AWS Console - Chi tiết ALB Listener & Rules</i></p>

📁 **Source Code:** Mở file `terraform/modules/alb/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## CloudFront CDN (`s3_cloudfront`)
**Mục đích:** Một Mạng Phân phối Nội dung (CDN) giúp tăng tốc độ phân phối media tĩnh (video, hình ảnh) lên đáng kể. Nó làm được điều này bằng cách lưu trữ các bản sao (cache) file của bạn tại các vị trí máy chủ vùng biên (edge locations) trên toàn cầu. Khi một người dùng ở Châu Âu yêu cầu xem một hình ảnh, CloudFront sẽ phục vụ ảnh đó từ một máy chủ ở Châu Âu, thay vì phải tải hình ảnh đó từ tận máy chủ chính của chúng ta ở Mỹ.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_cloudfront_distribution`: Đây là cấu hình cốt lõi của CDN. Như bạn thấy trên console, distribution của chúng ta được cấu hình như một "cửa ngõ thống nhất" xử lý các loại lưu lượng truy cập khác nhau thông qua một tên miền duy nhất. Chúng ta làm được điều này nhờ kết hợp nhiều Nguồn (Origins) và Hành vi (Behaviors).
    
    {{< img "images/Workshop/services/cloudfront-dist.png" "AWS Console - CloudFront Distribution" >}}
    <p align="center"><i>AWS Console - CloudFront Distribution</i></p>
    
    *   **Nhiều khối `origin` (Nguồn dữ liệu):** *(Origin là gì? Origin là nơi chứa bản gốc dữ liệu của bạn, chẳng hạn như máy chủ backend hoặc S3 bucket, nơi CloudFront sẽ tìm đến để lấy dữ liệu).* Chúng ta định nghĩa các nguồn riêng biệt cho từng thành phần kiến trúc. Hãy cùng phân tích chức năng của từng origin:
        1.  `s3-origin`: Trỏ tới S3 bucket dùng để lưu trữ mã nguồn tĩnh (HTML, CSS, JS đã được biên dịch) của giao diện Frontend.
        2.  `alb-origin`: Trỏ tới Application Load Balancer. Đây là cửa ngõ để xử lý tất cả các truy vấn API động cần được tính toán bởi các container Node.js ở backend.
        3.  `backend-storage-origin`: Trỏ tới một S3 bucket riêng biệt, chuyên dùng để lưu trữ các file media do người dùng tải lên (video, ảnh đại diện, ảnh thu nhỏ).
    
    {{< img "images/Workshop/services/cloudfront-origins.png" "AWS Console - CloudFront Origins" >}}
    <p align="center"><i>AWS Console - Nguồn CloudFront (Origins)</i></p>

    *   **Khối `default_cache_behavior` & `ordered_cache_behavior` (Hành vi):** *(Behavior là gì? Là một tập hợp các quy tắc cho CloudFront biết chính xác cách định tuyến và xử lý dữ liệu dựa trên đường dẫn URL của người dùng).* Quyết định chính xác cách CDN định tuyến lưu lượng dựa trên đường dẫn URL. Như hiển thị ở tab Behaviors:
        *   `/api/*` (Độ ưu tiên 0): Định tuyến các lệnh gọi API backend tới `alb-origin`.
        *   `/media/*` (Độ ưu tiên 1): Định tuyến các yêu cầu tải media tới `backend-storage-origin`.
        *   `Default (*)` (Độ ưu tiên 2): Định tuyến tất cả lưu lượng frontend còn lại tới `s3-origin`.
        *   Chúng ta bắt buộc `viewer_protocol_policy = "redirect-to-https"` trên tất cả các hành vi để đảm bảo bảo mật nghiêm ngặt.
        *   `viewer_certificate`: Cấu hình chứng chỉ SSL nếu sử dụng tên miền CDN tùy chỉnh.
    
    {{< img "images/Workshop/services/cloudfront-behaviors.png" "AWS Console - CloudFront Behaviors" >}}
    <p align="center"><i>AWS Console - Hành vi định tuyến CloudFront (Behaviors)</i></p>
    
*   `aws_cloudfront_origin_access_control` (OAC): *(OAC là gì? Đây là tính năng bảo mật nhằm đảm bảo S3 bucket của bạn chỉ chấp nhận luồng dữ liệu đi qua CloudFront).* Đây là một phương pháp bảo mật cực kỳ hiện đại. Nó tạo ra các định danh duy nhất dành riêng cho CloudFront. Sau đó, chúng ta sử dụng các định danh OAC này trong chính sách bảo mật (bucket policy) của S3. Việc này tạo ra một lớp bảo vệ hoàn hảo: người dùng bị ép buộc phải truy cập file thông qua CDN và hoàn toàn bị chặn nếu cố tình đọc file trực tiếp từ S3. Như bạn thấy trong ảnh, chúng ta đã tạo các OAC riêng biệt cho từng S3 bucket:
    1.  `publicast-staging-frontend-oac`: Bảo mật cho bucket chứa mã nguồn Frontend.
    2.  `publicast-staging-backend-storage-oac`: Bảo mật cho bucket chứa media do người dùng tải lên.
    
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `origin_access_control_origin_type = "s3"`: Chỉ định rằng định danh này được sử dụng cho Amazon S3.
        *   `signing_behavior = "always"` & `signing_protocol = "sigv4"`: Đảm bảo mọi truy vấn từ CloudFront đến S3 đều được ký bằng khóa bí mật và xác thực an toàn tuyệt đối.
    
    {{< img "images/Workshop/services/cloudfront-oac.png" "AWS Console - CloudFront OAC" >}}
    <p align="center"><i>AWS Console - CloudFront OAC</i></p>

📁 **Source Code:** Mở file `terraform/modules/s3_cloudfront/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.
