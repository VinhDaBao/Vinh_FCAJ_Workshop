---
title : "5.4.4. CodeBuild Projects"
weight : 4
---

## Tích hợp AWS CodeBuild

**Mục đích:** AWS CodeBuild là một dịch vụ tích hợp liên tục (CI) được quản lý hoàn toàn, thực hiện việc biên dịch mã nguồn, chạy thử nghiệm (tests) và tạo ra các gói phần mềm sẵn sàng để triển khai (như các Docker image hoặc các file tĩnh React đã biên dịch).

**Tài nguyên Terraform chính:** 
Trong mã Terraform của chúng ta, chúng tôi cấp phát hai tài nguyên `aws_codebuild_project` riêng biệt để xử lý các yêu cầu hoàn toàn khác nhau của frontend và backend:

*   **Backend Build Project:** 
    *   Chúng ta cấu hình cho nó sử dụng một image Amazon Linux 2 tiêu chuẩn.
    *   Điểm mấu chốt, chúng ta phải thiết lập `privileged_mode = true` trong khối environment. Điều này là yêu cầu bắt buộc để có thể chạy Docker daemon bên trong container của CodeBuild, cho phép chúng ta build các Docker image vi dịch vụ.
    *   Chúng ta tiêm (inject) các biến môi trường lúc chạy (như `IMAGE_REPO_NAME` và `ECS_CLUSTER_NAME`) để file script `backend/buildspec.yml` biết chính xác nơi cần đẩy (push) image lên và cập nhật dịch vụ nào.
*   **Frontend Build Project:** 
    *   Chúng ta thiết lập `privileged_mode = false` bởi vì chúng ta chỉ cần chạy các lệnh Node.js `npm run build` thông thường, chứ không dùng đến Docker.
    *   Chúng ta tiêm các biến `S3_BUCKET_NAME` và `CLOUDFRONT_DIST_ID` để file script `frontend/buildspec.yml` có thể tải lên (upload) các file tĩnh HTML/JS/CSS và xóa cache (invalidate) CDN.
*   **IAM Roles:** Cả hai dự án đều dùng chung một `aws_iam_role` cực kỳ nghiêm ngặt, giới hạn việc chúng chỉ được phép tương tác với ECR, ECS, S3 và CloudFront.

📁 **Source Code:** Mở file `terraform/modules/ci/cd/main.tf` trong IDE của bạn để xem toàn bộ cấu hình của CodeBuild.

**Kết quả:** Sau khi chạy lệnh `terraform apply` và đợi quá trình cấp phát hoàn tất, giao diện trên AWS Console sẽ hiển thị như sau:

{{< img "images/Workshop/services/codebuild.png" "AWS Console - CodeBuild Projects" >}}

---

### Khám phá bên trong quá trình Build

Để thấy rõ hơn sức mạnh của AWS CodeBuild, bạn hãy nhấp chuột vào tên của một project (ví dụ: `publiast-staging-backend-build`), sau đó chuyển sang thẻ **Build logs** (hoặc **Phase details**) của một lần chạy (Build run) bất kỳ. 

Tại đây, bạn sẽ thấy một giao diện Terminal thực thụ, nơi hiển thị chi tiết từng dòng lệnh đang được hệ thống thực thi tự động (như `docker build`, `docker push`, hoặc `npm run build`). Đây là nơi cực kỳ hữu ích để debug nếu Pipeline của bạn gặp lỗi.

{{< img "images/Workshop/services/codebuild-logs.png" "AWS Console - CodeBuild Logs" >}}
<p align="center"><i>Chi tiết nhật ký chạy code bên trong CodeBuild</i></p>
