---
title: "5.3.4. Thực thi Triển khai"
weight: 4
---

Bây giờ bạn đã hiểu rõ bố cục hạ tầng và các module Terraform ẩn bên dưới, đã đến lúc thực sự triển khai các tài nguyên AWS!

## Bước 1: Cấu hình Biến Môi trường (Variables)

Trước khi chạy bất kỳ lệnh Terraform nào, bạn cần cung cấp các giá trị cụ thể của riêng mình. Bởi vì nền tảng PubliCast tích hợp với rất nhiều dịch vụ của bên thứ ba, file `terraform.tfvars` của bạn sẽ đóng vai trò là sổ đăng ký trung tâm cho toàn bộ API keys và mật khẩu.

Chúng ta **tuyệt đối không bao giờ** hardcode các giá trị nhạy cảm này vào hệ thống kiểm soát phiên bản (như Git).

1. Mở terminal và di chuyển vào thư mục `terraform`:
   ```bash
   cd terraform
   ```
2. Copy file biến mẫu để tạo ra file `terraform.tfvars` cục bộ của bạn:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```
3. Mở file `terraform.tfvars` trong code editor của bạn và điền vào các giá trị thực tế. Các biến đã được nhóm theo logic:
   
   *   **Bảo mật lõi & Database (`db_password`, `jwt_secret`, `encryption_key`, v.v.)**: Hãy tự tạo ra các chuỗi ký tự ngẫu nhiên, độ khó cao cho các giá trị này. Chúng được hệ thống dùng nội bộ để truy cập cơ sở dữ liệu và mã hóa phiên đăng nhập của người dùng.
   *   **Cấu hình AWS (`acm_certificate_arn`, `domain_name`, `frontend_bucket_name`)**: Dán chuỗi ARN của chứng chỉ SSL mà bạn đã tạo ở phần 5.2, điền tên miền đã đăng ký của bạn, và chọn một cái tên duy nhất trên toàn cầu cho S3 bucket.
   *   **Mã khóa API Mạng xã hội (Social OAuth & Publishing)**: PubliCast đóng vai trò là một hub mạng xã hội. Bạn cần đăng ký ứng dụng của mình trên các nền tảng này để lấy Client ID và Secret:
       *   **Google/YouTube**: Truy cập [Google Cloud Console](https://console.cloud.google.com/), kích hoạt YouTube Data API v3, và tạo thông tin xác thực OAuth 2.0.
       *   **Meta (Facebook, Instagram, Threads)**: Truy cập [Meta for Developers](https://developers.facebook.com/), tạo Business App, và thêm các sản phẩm Facebook Login, Instagram Graph API, cùng Threads API.
       *   **TikTok**: Đăng ký tại cổng thông tin [TikTok for Developers](https://developers.tiktok.com/).
       *   **Discord**: Tạo một ứng dụng trong [Discord Developer Portal](https://discord.com/developers/applications) để lấy Client ID và Bot Token.
   *   **Tích hợp Bên thứ ba (3rd Party Integrations)**:
       *   **Thanh toán (VietQR/SePay)**: Điền số tài khoản ngân hàng của bạn (`vietqr_account_no`) và [SePay API Key](https://sepay.vn/) để tự động hóa việc xác thực giao dịch.
       *   **Gửi Email (Resend)**: Lấy API key của bạn từ [Resend](https://resend.com/).
       *   **AI (Trí tuệ nhân tạo)**: Lấy mã khóa từ [OpenAI](https://platform.openai.com/) (ChatGPT) và [Google AI Studio](https://aistudio.google.com/) (Gemini) cho các tính năng tự động tạo nội dung (AI content generation).
   *   **Đường ống CI/CD (`github_monorepo`, `github_branch`)**: Điền chuỗi kho lưu trữ GitHub thực tế của bạn (ví dụ: `username/publicast`) và nhánh (branch) bạn muốn CodePipeline theo dõi (ví dụ: `main`).

> [!WARNING]
> **Bảo mật mã khóa của bạn!** 
> File `terraform.tfvars` đã tự động được bỏ qua bởi `.gitignore`. **Tuyệt đối không bao giờ** commit file `terraform.tfvars` thực tế của bạn lên GitHub, vì nó chứa các thông tin xác thực rất nhạy cảm mà hacker có thể khai thác.

## Bước 2: Khởi tạo Terraform (Init)

Khởi tạo thư mục làm việc Terraform. Lệnh này sẽ tải xuống các plugin provider AWS cần thiết.

```bash
terraform init
```

## Bước 3: Xem trước Thay đổi (Plan)

Tạo một kế hoạch thực thi. Bước này cho phép bạn xem trước một cách an toàn chính xác những tài nguyên AWS nào Terraform sẽ tạo ra trước khi nó thực sự làm việc đó.

```bash
terraform plan
```

## Bước 4: Áp dụng Cấu hình (Apply)

Khi bạn đã hài lòng với kế hoạch, hãy thực thi nó để cấp phát hạ tầng.

```bash
terraform apply
```

Lúc này, Terraform sẽ liệt kê lại toàn bộ các thay đổi một lần cuối và dừng lại để hỏi bạn có chắc chắn muốn thực hiện không. Hãy gõ `yes` và nhấn Enter để xác nhận.

{{< img "images/Workshop/services/terraform-apply-yes.png" "Terminal - Terraform Apply Confirm" >}}

> [!NOTE]
> Quá trình triển khai sẽ mất khoảng 10-15 phút, chủ yếu là vì việc cấp phát RDS Database và cluster ElastiCache Redis mất khá nhiều thời gian. Hãy pha một tách cà phê và chờ đợi thông báo `Apply complete!` xuất hiện trong terminal của bạn.

{{< img "images/Workshop/services/terraform-apply-complete.png" "Terminal - Terraform Apply Complete" >}}
