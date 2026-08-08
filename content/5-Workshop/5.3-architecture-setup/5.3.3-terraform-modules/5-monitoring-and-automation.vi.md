---
title: "5.3.3.5. Lớp Giám sát & Tự động hóa"
weight: 5
---

Lớp này đóng vai trò là "bộ não" và "đôi mắt" cho hạ tầng của chúng ta. Nó đảm bảo hệ thống có thể được quan sát liên tục, được cảnh báo khi có sự cố, và các công việc triển khai hoặc bảo trì diễn ra hoàn toàn tự động mà không cần sự can thiệp của con người.

## CloudWatch (`cloudwatch`)
**Mục đích:** Một dịch vụ giám sát và khả năng quan sát tập trung. Nó hoạt động như một chiếc phễu khổng lồ thu thập các log (nhật ký hệ thống), các số liệu hiệu suất (như mức sử dụng CPU), và các sự kiện từ tất cả các tài nguyên AWS của bạn vào một bảng điều khiển duy nhất.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_cloudwatch_log_group`: Hãy coi Log Group như một thư mục nơi một dịch vụ cụ thể ghi log của nó vào. Chúng ta tạo một Log Group cho mỗi vi dịch vụ ECS. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name`: Tạo một đường dẫn phân tách (ví dụ: `/ecs/publiast-staging`).
        *   `retention_in_days`: Chúng ta thiết lập rõ ràng thuộc tính này (ví dụ: 7 ngày - 1 week). Nếu không có thiết lập này, AWS sẽ giữ log của bạn vĩnh viễn, và chi phí lưu trữ CloudWatch của bạn sẽ tăng lên không giới hạn.
    
    **💡 Phân tích từ giao diện Console:**
    Như trong hình minh họa bên dưới, khi truy cập vào **CloudWatch > Log groups**, bạn sẽ thấy danh sách các Log Group đang hoạt động trong dự án:
    - **Log group của ECS** (`/ecs/publiast-staging`): Được cấu hình thời gian lưu trữ (Retention) là **1 week** (7 ngày) để tiết kiệm chi phí, đồng thời có đính kèm **1 Metric filter** (bộ lọc từ khóa lỗi mà chúng ta sẽ tìm hiểu ở phần sau).
    - **Các Log group của CodeBuild** (`/aws/codebuild/...`): Đây là các nhật ký (log) sinh ra trong quá trình hệ thống CI/CD build mã nguồn frontend và backend. Mặc định chúng đang được để ở mức `Never expire` (lưu trữ vĩnh viễn).

    {{< img "images/Workshop/services/cloudwatch-logs.png" "AWS Console - CloudWatch Logs" >}}
    <p align="center"><i>AWS Console - CloudWatch Logs</i></p>

    **🔍 Xem chi tiết Log Stream (Nhật ký hoạt động):**
    Từ màn hình Log Groups, nếu bạn click vào `/ecs/publiast-staging` và chọn một **Log stream** đang hoạt động (ví dụ: `ecs/publicast-api/...`), bạn sẽ thấy toàn bộ nhật ký hệ thống (log events) mà ứng dụng in ra. 
    Điều này chứng minh ứng dụng ECS đang ghi log thành công lên CloudWatch và cũng là nơi mà **Metric Filter** sẽ tiến hành quét để tìm kiếm các từ khóa lỗi (như ERROR, Exception) để phát cảnh báo.

    {{< img "images/Workshop/services/cloudwatch-log-stream.png" "AWS Console - CloudWatch Log Stream" >}}
    <p align="center"><i>AWS Console - CloudWatch Log Stream</i></p>

*   `aws_cloudwatch_log_metric_filter` & `aws_cloudwatch_metric_alarm`: Tạo bộ lọc quét các từ khóa lỗi (như ERROR, Exception) trong log của ECS. Khi phát hiện lỗi, Metric Alarm sẽ được kích hoạt để thông báo.
    *   **Các Thuộc tính Được Cấu hình:**
        *   `pattern`: Từ khóa để quét (ví dụ: `?ERROR ?Error ?Exception`).
        *   `threshold` & `evaluation_periods`: Cấu hình ngưỡng báo động khi số lỗi vượt quá giới hạn.

    {{< img "images/Workshop/services/cloudwatch-alarm.png" "AWS Console - CloudWatch Alarm" >}}
    <p align="center"><i>AWS Console - CloudWatch Alarm</i></p>

*   `aws_sns_topic` & `aws_sns_topic_subscription`: Kênh nhận và phân phối thông báo cảnh báo từ CloudWatch Alarm.
    *   **Các Thuộc tính Được Cấu hình:**
        *   `protocol`: Giao thức gửi thông báo (chúng ta sử dụng `email`).
        *   `endpoint`: Địa chỉ email nhận thông báo cảnh báo. Cần phải xác nhận (Confirm Subscription) qua email sau khi chạy lệnh Terraform apply.

    {{< img "images/Workshop/services/sns-topic.png" "AWS Console - SNS Topic" >}}
    <p align="center"><i>AWS Console - SNS Topic</i></p>
    


📁 **Source Code:** Mở file `terraform/modules/cloudwatch/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.



## CI/CD Pipeline IAM (`ci`)
**Mục đích:** Cung cấp các quyền hạn cần thiết cho hệ thống Tích hợp Liên tục và Triển khai Liên tục (CI/CD) để tự động hóa các bản triển khai một cách an toàn mà không cần cấp cho nó toàn quyền quản trị viên.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_iam_user` & `aws_iam_access_key`: Cấp phát một người dùng IAM dành riêng (hoạt động qua API) cho bot CI/CD.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name`: Đặt tên người dùng cho bot (ví dụ: `publicast-ci-bot`).
        *   Tài nguyên `aws_iam_access_key` sẽ tạo ra Access Key ID cùng Secret Access Key. *(Lưu ý: Mặc dù dự án của chúng ta dùng AWS Native CodePipeline vốn không thực sự cần key tĩnh, tài nguyên này hoạt động như một phương án dự phòng hoặc được dùng nếu bạn thích các công cụ CI bên ngoài như GitHub Actions).*
    
    {{< img "images/Workshop/services/iam-user.png" "AWS Console - CI IAM User" >}}
    <p align="center"><i>AWS Console - CI IAM User</i></p>
    
*   `aws_iam_policy`: Chúng ta tạo ra một chính sách JSON nghiêm ngặt, *chỉ* cấp quyền cho đúng những hành động cần thiết cho việc triển khai. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `policy`: Định nghĩa các hành động được phép. Bot chỉ được phép làm những việc như `ecr:GetAuthorizationToken` (đăng nhập vào Docker), `ecr:BatchPushImage` (tải code mới lên), và `ecs:UpdateService` (bảo ECS sử dụng code mới). Nhờ làm vậy, ngay cả khi các Key CI/CD bị hacker đánh cắp, kẻ tấn công cũng không thể xóa database hoặc sửa đổi VPC, giữ cho phạm vi ảnh hưởng ở mức cực kỳ nhỏ.
    
    {{< img "images/Workshop/services/iam-ci-policy.png" "AWS Console - CI IAM Policy" >}}
    <p align="center"><i>AWS Console - CI IAM Policy</i></p>

📁 **Source Code:** Mở file `terraform/modules/ci/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.
