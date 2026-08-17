---
title: "5.3.4. Thực thi Triển khai"
weight: 4
---

Bây giờ bạn đã hiểu rõ bố cục hạ tầng và các module Terraform ẩn bên dưới, đã đến lúc thực sự triển khai các tài nguyên AWS!

## Bước 1: Cấu hình Tài khoản AWS (AWS Configure)

Trước khi Terraform có thể tương tác và tạo ra các tài nguyên thực tế trên AWS, máy tính (hoặc môi trường dòng lệnh) của bạn cần được cấp quyền truy cập thông qua một bộ mã khóa bảo mật. Việc cấu hình này đảm bảo tính xác thực và phân quyền chính xác cho quá trình triển khai tự động.

Dưới đây là các thao tác chi tiết:

1. **Truy cập AWS IAM:** Đăng nhập vào [AWS Management Console](https://console.aws.amazon.com/). Trên thanh tìm kiếm ở góc trên cùng màn hình, gõ **IAM** (Identity and Access Management) và chọn dịch vụ này.
   {{< img "images/Workshop/services/iam-search.png" "Search IAM" >}}
2. **Tạo Người dùng (User):** Ở menu bên trái, chọn **Users** và nhấn nút **Create user**. Đặt tên cho người dùng (ví dụ: `terraform-admin`) và nhấn Next.
   {{< img "images/Workshop/services/iam-create-user.png" "Create IAM User" >}}
3. **Cấp quyền Quản trị viên:** Ở bước Set permissions, chọn **Attach policies directly**. Tìm kiếm và tick vào ô **AdministratorAccess** (quyền cao nhất) để đảm bảo Terraform có thể tạo mọi tài nguyên cần thiết cho PubliCast, sau đó nhấn Create user.
   {{< img "images/Workshop/services/iam-attach-policy.png" "Attach AdministratorAccess Policy" >}}
   > [!WARNING]
   > Việc cấp quyền `AdministratorAccess` ở đây chỉ nhằm mục đích thuận tiện cho bài thực hành (Lab), giúp Terraform có thể khởi tạo toàn bộ tài nguyên mà không gặp lỗi từ chối truy cập. Trong môi trường thực tế (Production), bạn cần tuân thủ nguyên tắc Đặc quyền tối thiểu (Least Privilege) bằng cách cấu hình các policy chặt chẽ hơn, chỉ cấp đúng những quyền mà Terraform thực sự cần.
4. **Tạo Access Key:** Bấm vào tên người dùng `terraform-admin` vừa tạo.
   {{< img "images/Workshop/services/iam-user-created.png" "User Created Successfully" >}}
   
   Chuyển sang tab **Security credentials**. Kéo xuống phần Access keys và bấm **Create access key**.
   {{< img "images/Workshop/services/iam-security-credentials.png" "Security Credentials Tab" >}}
   
   Ở bước 1 (Access key best practices & alternatives), chọn Use case là **Command Line Interface (CLI)**, tích chọn ô xác nhận cảnh báo ở dưới cùng và nhấn Next.
   {{< img "images/Workshop/services/iam-access-key-cli.png" "Select CLI Use Case" >}}
   
   Ở bước 2 (Set description tag), bạn có thể đặt mô tả ngắn gọn (ví dụ: `terraform`) và nhấn **Create access key**.
   {{< img "images/Workshop/services/iam-access-key-tag.png" "Set Access Key Tag" >}}
   
   Ở bước 3 (Retrieve access keys), màn hình sẽ hiển thị `Access key` và `Secret access key`. Đây là lần duy nhất bạn có thể xem được Secret Key, hãy copy lại và nhấn **Done** để hoàn tất.
   {{< img "images/Workshop/services/iam-access-key-retrieve.png" "Retrieve Access Keys" >}}
5. **Khởi chạy cấu hình AWS CLI:** Đảm bảo bạn đã cài đặt AWS CLI trên máy tính. Mở terminal (hoặc Command Prompt/PowerShell) của bạn và chạy lệnh khởi tạo:
   ```bash
   aws configure
   ```
   {{< img "images/Workshop/services/aws-configure-cmd.png" "Run aws configure" >}}
6. **Nhập thông tin kết nối:** Terminal sẽ lần lượt yêu cầu bạn nhập 4 thông số. Hãy điền cẩn thận:
   * **AWS Access Key ID**: Dán chuỗi Key ID bạn vừa nhận được ở bước 4 (Ví dụ: `AKIAIOSFODNN7EXAMPLE`).
   * **AWS Secret Access Key**: Dán chuỗi Secret Key tương ứng (chỉ hiện ra một lần duy nhất).
   * **Default region name**: Nhập mã khu vực AWS mà bạn muốn triển khai dự án (ví dụ: `ap-southeast-1` cho vùng Singapore, hoặc `us-east-1` cho Bắc Virginia).
   * **Default output format**: Nhập `json` để kết quả các lệnh AWS CLI trả về dễ đọc hơn.
   
   {{< img "images/Workshop/services/aws-configure-inputs.png" "Enter AWS Credentials" >}}

## Bước 2: Cấu hình Biến Môi trường (Variables)

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

## Bước 3: Khởi tạo Terraform (Init)

Khởi tạo thư mục làm việc Terraform. Lệnh này sẽ tải xuống các plugin provider AWS cần thiết.

```bash
terraform init
```

## Bước 4: Xem trước Thay đổi (Plan)

Tạo một kế hoạch thực thi. Bước này cho phép bạn xem trước một cách an toàn chính xác những tài nguyên AWS nào Terraform sẽ tạo ra trước khi nó thực sự làm việc đó.

```bash
terraform plan
```

## Bước 5: Áp dụng Cấu hình (Apply)

Khi bạn đã hài lòng với kế hoạch, hãy thực thi nó để cấp phát hạ tầng.

```bash
terraform apply
```

Lúc này, Terraform sẽ liệt kê lại toàn bộ các thay đổi một lần cuối và dừng lại để hỏi bạn có chắc chắn muốn thực hiện không. Hãy gõ `yes` và nhấn Enter để xác nhận.

{{< img "images/Workshop/services/terraform-apply-yes.png" "Terminal - Terraform Apply Confirm" >}}

> [!NOTE]
> Quá trình triển khai sẽ mất khoảng 10-15 phút, chủ yếu là vì việc cấp phát RDS Database và cluster ElastiCache Redis mất khá nhiều thời gian. Hãy pha một tách cà phê và chờ đợi thông báo `Apply complete!` xuất hiện trong terminal của bạn.

{{< img "images/Workshop/services/terraform-apply-complete.png" "Terminal - Terraform Apply Complete" >}}

## Bước 6: Kiểm thử và Đo lường (Test & Validation)

Sau khi hệ thống được triển khai thành công, bạn cần kiểm thử để đảm bảo mọi thứ hoạt động như mong đợi:

1. **Truy cập ứng dụng:** Mở trình duyệt và truy cập vào tên miền của bạn (ví dụ: `https://publicast.yourdomain.com`).
   {{< img "images/Workshop/services/test-access-app.png" "Access Application" >}}
2. **Gửi Request:** Thử đăng nhập, tạo một bài viết (Worklog) mới hoặc tải lên một hình ảnh.
   {{< img "images/Workshop/services/test-send-request.png" "Send Request" >}}
3. **Xem Log hệ thống:** Truy cập AWS Console > **CloudWatch** > **Log groups**. Tìm đến `/ecs/publicast-staging` và kiểm tra luồng log để xác minh request của bạn đã được ghi nhận.
   {{< img "images/Workshop/services/test-cloudwatch-logs.png" "CloudWatch Logs" >}}
4. **Kiểm thử cảnh báo (CloudWatch Metrics & Alarms):** Hệ thống được cấu hình tự động quét log để tìm lỗi và phát cảnh báo. Để kiểm thử luồng này, bạn làm theo các bước sau:
   * **Tạo lỗi nhân tạo (Trigger Error):** Cố tình gọi một API không tồn tại (ví dụ: `https://api.publicast.yourdomain.com/v1/invalid-endpoint`) hoặc nhập sai thông tin đăng nhập nhiều lần liên tiếp để ép backend sinh ra log có chứa từ khóa `ERROR` hoặc `Exception`.
   * **Kiểm tra Metric Filter:** Truy cập vào **CloudWatch** > **Log groups** > chọn `/ecs/publicast-staging`. Chuyển sang tab **Metric filters**, bạn sẽ thấy bộ đếm (metric) của từ khóa lỗi tăng lên.
   * **Quan sát trạng thái Alarm:** Điều hướng sang mục **All alarms** ở thanh menu bên trái. Tìm Alarm có tên `publicast-staging-error-alarm`. Trạng thái của nó sẽ chuyển từ cột màu xanh (OK) sang màu đỏ (In alarm) do số lượng lỗi vượt quá ngưỡng cho phép (threshold) trong thời gian ngắn.
   * **Xác minh Email Alert (SNS):** Kiểm tra hộp thư email mà bạn đã đăng ký nhận thông báo (đã confirm subscription ở phần thiết lập trước đó). Bạn sẽ nhận được một email tự động từ AWS Notifications báo cáo chi tiết về sự cố, giúp đội ngũ vận hành phản ứng kịp thời.
   
   {{< img "images/Workshop/services/test-cloudwatch-alarm.png" "CloudWatch Alarm" >}}
