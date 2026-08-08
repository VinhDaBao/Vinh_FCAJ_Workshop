---
title : "5.4.3. Kết nối Nguồn GitHub"
weight : 3
---

Để AWS CodePipeline có thể tự động lấy mã nguồn (pull) mới nhất mỗi khi bạn thực hiện hành động push lên GitHub, AWS cần có quyền truy cập bảo mật vào kho lưu trữ GitHub của bạn.

Chúng tôi triển khai sự tích hợp này bằng cách sử dụng dịch vụ AWS CodeStar Connections thông qua Terraform:

```hcl
# ------------------------------------------------------------------------------
# 2. CODESTAR CONNECTION (KẾT NỐI GITHUB)
# ------------------------------------------------------------------------------
resource "aws_codestarconnections_connection" "github" {
  name          = "${var.project}-${var.environment}-github-conn"
  provider_type = "GitHub"
}
```

## Vấn đề về trạng thái "Pending"

Khi bạn chạy lệnh `terraform apply`, Terraform sẽ tạo thành công tài nguyên kết nối (connection) trong Tài khoản AWS của bạn. **Tuy nhiên, Terraform không thể tự động cấp quyền cho AWS đọc tài khoản GitHub cá nhân của bạn.** Quá trình này yêu cầu một luồng đăng nhập OAuth có tính tương tác thực hiện trên trình duyệt web của bạn.

Bởi vì yêu cầu bảo mật này, ngay sau khi Terraform hoàn tất quá trình triển khai, kết nối này sẽ bị kẹt ở trạng thái **Pending** (Đang chờ xử lý). CodePipeline của bạn sẽ không thể kích hoạt (bị báo lỗi) cho đến khi bạn phê duyệt thủ công kết nối này.

## Cách Ủy quyền cho Kết nối (Authorize)

Hãy làm theo các bước sau ngay lập tức sau khi chạy lệnh `terraform apply`:

1.  Đăng nhập vào AWS Management Console và tìm kiếm dịch vụ **Developer Tools**.
2.  Trên bảng điều hướng bên trái, dưới mục **Settings**, hãy nhấp vào **Connections**.
3.  Bạn sẽ thấy kết nối do Terraform tạo ra (ví dụ: `publicast-workshop-github-conn`). Trạng thái (Status) sẽ hiển thị là **Pending**.
    
    {{< img "images/Workshop/services/github-pending.png" "AWS Console - Pending Connection" >}}
    <p align="center"><i>Trạng thái Pending ban đầu</i></p>

4.  Chọn kết nối đó và nhấp vào nút **Update pending connection** (Cập nhật kết nối đang chờ).
5.  Trình duyệt sẽ chuyển hướng bạn đến trang **Connect to GitHub**. Tại phần **App Installation**, hãy chọn ứng dụng AWS đã cài đặt trước đó (hoặc bấm **Install a new app** nếu đây là lần đầu tiên). Sau đó bấm nút **Connect** màu cam.
    
    {{< img "images/Workshop/services/github-update.png" "AWS Console - Connect to GitHub" >}}
    <p align="center"><i>Giao diện cập nhật kết nối (Connect to GitHub)</i></p>

    > [!NOTE]
    > **Bảo mật GitHub (Sudo mode):** Nếu bạn gặp màn hình **"Confirm access"**, đây là cơ chế bảo mật bình thường của GitHub. Hãy bấm **Verify via email**, lấy mã 6 số từ hòm thư của bạn và nhập vào để đi tiếp.

6.  Nhấp vào **Authorize AWS Connector for GitHub**. Bạn sẽ được yêu cầu cài đặt ứng dụng AWS trên kho lưu trữ cụ thể của mình (đảm bảo rằng bạn chọn đúng kho lưu trữ PubliCast monorepo đã clone).
    
    {{< img "images/Workshop/services/github-authorize.png" "GitHub - Authorize AWS Connector" >}}
    <p align="center"><i>Cấp quyền cho AWS Connector trên GitHub</i></p>

7.  Sau khi hoàn tất, trạng thái trong AWS Console sẽ chuyển từ *Pending* sang **Available** (Có sẵn).
    
    {{< img "images/Workshop/services/github-available.png" "AWS Console - Available Connection" >}}
    <p align="center"><i>Kết nối thành công với trạng thái Available</i></p>

> [!IMPORTANT]
> Các đường ống CI/CD của bạn giờ đây đã được liên kết hoàn toàn với kho lưu trữ GitHub. Bất kỳ lệnh `git push` nào trong tương lai tới nhánh mà bạn đã cấu hình sẽ tự động kích hoạt các bản build của CodePipeline!
