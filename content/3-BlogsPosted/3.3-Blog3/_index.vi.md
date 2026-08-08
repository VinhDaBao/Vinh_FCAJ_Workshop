---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---


# CI/CD TRÊN AWS – KHI VIỆC DEPLOY KHÔNG CÒN LÀ “ĐĂNG NHẬP SERVER RỒI COPY CODE”

Trong quá trình phát triển dự án, có lẽ điều mà nhóm mình gặp nhiều nhất không phải là viết tính năng mới mà là việc triển khai sản phẩm. Mỗi lần có một thay đổi nhỏ, cả nhóm đều phải build lại ứng dụng, đăng nhập vào máy chủ, cập nhật source code, khởi động lại service rồi kiểm tra xem mọi thứ có hoạt động bình thường hay không. Chỉ cần quên một bước hoặc hai thành viên cùng thao tác lên một môi trường là rất dễ phát sinh lỗi.

Sau khi tìm hiểu về DevOps và các dịch vụ trên AWS, nhóm mình quyết định xây dựng một quy trình **CI/CD** để tự động hóa toàn bộ quá trình này.

## Bắt đầu với GitHub và Monorepo

Điểm xuất phát là đưa toàn bộ mã nguồn lên GitHub theo mô hình **monorepo**, trong đó backend và frontend được quản lý trong cùng một repository nhưng có hai quy trình build độc lập. Việc này giúp việc quản lý phiên bản trở nên thuận tiện hơn, đồng thời giảm công sức khi đồng bộ thay đổi giữa hai phần của hệ thống.

Mỗi khi có một commit mới được đẩy lên nhánh triển khai, **AWS CodeStar Connection** sẽ tạo kết nối an toàn giữa GitHub và AWS để kích hoạt pipeline. Đây là thành phần giúp AWS có thể theo dõi thay đổi trên repository mà không cần phải cấu hình webhook thủ công.

## CodePipeline và Infrastructure as Code

Sau khi nhận được source code, **AWS CodePipeline** sẽ bắt đầu thực hiện workflow đã được định nghĩa bằng Terraform. Thay vì phải tạo từng dịch vụ bằng giao diện AWS Console, toàn bộ hạ tầng được mô tả dưới dạng mã nguồn (**Infrastructure as Code**).

Điều này mang lại khá nhiều lợi ích như dễ kiểm soát phiên bản, dễ tái sử dụng và đặc biệt là có thể triển khai lại toàn bộ hạ tầng chỉ với một vài lệnh Terraform.

Trong pipeline của nhóm, backend và frontend được tách thành hai pipeline riêng biệt. Cách tổ chức này giúp mỗi thành phần có thể triển khai độc lập. Nếu chỉ thay đổi giao diện thì frontend sẽ được build và deploy mà không ảnh hưởng đến backend. Ngược lại, khi backend cập nhật API hoặc xử lý nghiệp vụ thì frontend vẫn không cần build lại.

## Build Backend với CodeBuild và ECR

Ở bước Build, **AWS CodeBuild** chịu trách nhiệm biên dịch source code. Backend sử dụng môi trường Docker nên CodeBuild được cấu hình ở chế độ *privileged mode* để có thể build Docker image.

Sau khi image được tạo thành công, image sẽ được đẩy lên **Amazon ECR**. Các biến môi trường như tên repository, AWS Account ID, ECS Cluster hay tên các ECS Service đều được truyền vào thông qua Terraform, giúp `buildspec` không phải chứa các giá trị cố định.

## Deploy Backend với Amazon ECS

Sau khi image mới xuất hiện trên ECR, pipeline tiếp tục cập nhật các service đang chạy trên **Amazon ECS**.

Trong dự án của nhóm, backend được chia thành nhiều service như:

* API
* Worker Light
* Worker Heavy

Nhờ đó, từng thành phần có thể mở rộng độc lập tùy theo tải của hệ thống. Khi có phiên bản mới, ECS sẽ thực hiện **rolling update** để thay thế container cũ bằng container mới mà không cần dừng toàn bộ hệ thống.

## Deploy Frontend với S3 và CloudFront

Đối với frontend, quy trình đơn giản hơn một chút. Sau khi CodeBuild hoàn thành việc build ứng dụng, toàn bộ file tĩnh sẽ được upload lên **Amazon S3**.

Tiếp theo, pipeline thực hiện **invalidate Amazon CloudFront** để xóa cache, giúp người dùng nhận được phiên bản mới ngay khi truy cập website mà không cần chờ cache hết hạn.

## Pipeline Artifacts

Một chi tiết nhỏ nhưng khá quan trọng là CodePipeline cần một nơi để lưu trữ các artifact tạm thời giữa các stage.

Vì vậy nhóm mình sử dụng một **S3 Bucket riêng dành cho Pipeline Artifacts**. Mặc dù người dùng gần như không bao giờ nhìn thấy bucket này, nhưng nó đóng vai trò là nơi lưu trữ các gói source code và kết quả build để các stage tiếp theo có thể sử dụng.

## Quản lý IAM bằng Terraform

Toàn bộ **IAM Role** và **IAM Policy** cho CodePipeline cũng như CodeBuild đều được định nghĩa bằng Terraform. Điều này giúp quyền truy cập được quản lý tập trung thay vì cấp thủ công trên AWS Console.

Khi cần triển khai sang một môi trường khác như staging hoặc production, chỉ cần thay đổi giá trị của các biến như `project`, `environment` hoặc `branch` là có thể tái sử dụng gần như toàn bộ module.

## Quy trình Deploy sau khi tự động hóa

Điều mình thấy giá trị nhất sau khi hoàn thành hệ thống này không phải là tốc độ build nhanh hơn, mà là sự ổn định trong quá trình triển khai.

Trước đây mỗi lần release đều phải kiểm tra rất nhiều bước thủ công. Còn bây giờ, quy trình gần như chỉ còn là:

1. Push code lên GitHub.
2. CodePipeline tự lấy source.
3. CodeBuild tự build.
4. Docker image được cập nhật lên ECR.
5. ECS triển khai phiên bản mới.
6. Frontend được đồng bộ lên S3 và CloudFront tự làm mới cache.

Nhờ vậy, thời gian triển khai giảm đáng kể và mọi thành viên trong nhóm đều sử dụng chung một quy trình. Điều này cũng giúp việc truy vết lỗi dễ dàng hơn vì mỗi lần build đều có log, trạng thái và lịch sử thực thi được lưu lại trên AWS.

## Bài học về Infrastructure as Code

Qua quá trình xây dựng hệ thống, nhóm mình cũng hiểu rõ hơn vai trò của **Infrastructure as Code**.

Thay vì xem Terraform chỉ là công cụ tạo tài nguyên AWS, nhóm bắt đầu coi toàn bộ hạ tầng như một phần của source code. Mọi thay đổi đều được commit, review và lưu trữ giống như quá trình phát triển phần mềm.

Nếu đang tìm hiểu về CI/CD trên AWS, mình thấy đây là một mô hình khá phù hợp để bắt đầu vì vừa tận dụng được các dịch vụ managed của AWS, vừa giúp làm quen với cách tổ chức pipeline trong các dự án thực tế.

Các điểm chính cần nắm:

* Hiểu cách xây dựng CI/CD pipeline trên AWS.
* Biết cách kết nối GitHub với AWS CodePipeline.
* Hiểu vai trò của CodeBuild, ECR, ECS, S3 và CloudFront trong quá trình deploy.
* Biết cách tách pipeline backend và frontend để triển khai độc lập.
* Hiểu cách sử dụng Terraform để quản lý Infrastructure as Code.
* Giảm phụ thuộc vào các thao tác deploy thủ công.
* Dễ dàng theo dõi lịch sử, log và trạng thái của từng lần triển khai.