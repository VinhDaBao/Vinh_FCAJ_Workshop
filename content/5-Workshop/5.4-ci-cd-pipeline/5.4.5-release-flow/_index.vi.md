---
title : "5.4.5. CodePipelines"
weight : 5
---

## Điều phối bằng AWS CodePipeline

**Mục đích:** AWS CodePipeline điều phối toàn bộ quy trình phát hành (release process). Nó hoạt động như một loại "chất keo" kết nối kho lưu trữ GitHub (Giai đoạn Source) và các dự án CodeBuild (Giai đoạn Build) thành một luồng phân phối liên tục (CD), tự động hóa hoàn toàn và thống nhất.

**Tài nguyên Terraform chính:** 
Chúng ta sử dụng tài nguyên `aws_codepipeline` để định nghĩa hai đường ống song song cho mô hình monorepo:

*   **Backend Pipeline & Frontend Pipeline:**
    *   **Git Triggers (Chiến lược Monorepo):** Vì dự án sử dụng kiến trúc Monorepo, việc thiết lập `pipeline_type = "V2"` cho phép chúng ta sử dụng khối `trigger` cực kỳ thông minh. Nó lọc sự kiện theo đường dẫn (`file_paths`): Backend Pipeline chỉ chạy khi có thay đổi trong `backend/**`, và Frontend Pipeline chỉ chạy khi có thay đổi trong `frontend/**`. Điều này giúp tiết kiệm tối đa thời gian và chi phí build. *(Lưu ý: Bắt buộc phải tắt `DetectChanges = false` ở cấu hình Source để nhường quyền điều khiển cho khối trigger này).*
    *   **Artifact Store:** Cả hai pipeline đều định nghĩa một khối `artifact_store` trỏ tới S3 bucket mà chúng ta đã tạo ở bước 5.4.1. Đây chính là cách các file được truyền qua lại một cách bảo mật giữa các giai đoạn.
    *   **Giai đoạn 1 (Source):** Sử dụng provider `CodeStarSourceConnection`. Nó trỏ tới kho lưu trữ GitHub monorepo và một nhánh cụ thể. Khối này sẽ xuất ra một file zip `source_output`.
    *   **Giai đoạn 2 (Build):** Lấy file zip `source_output` làm đầu vào (`input_artifact`) và truyền nó vào CodeBuild tương ứng.
*   **IAM Roles:** Chúng ta gắn một `aws_iam_role` cụ thể cho các pipeline, cấp cho chúng quyền đọc/ghi vào bucket S3 lưu trữ artifact và quyền kích hoạt các lần thực thi của CodeBuild.

📁 **Source Code:** Mở file `terraform/modules/ci/cd/main.tf` trong IDE của bạn để xem toàn bộ cấu hình của CodePipeline.

> [!IMPORTANT]
> Sau khi quá trình triển khai Terraform kết thúc và bạn đã hoàn tất việc cấp quyền (authorize) cho kết nối CodeStar (như được trình bày chi tiết ở mục 5.4.3), bạn sẽ thấy cả hai pipeline tự động kích hoạt và chạy thành công trên AWS Console.

**Kết quả:** Sau khi chạy lệnh `terraform apply` và đợi quá trình cấp phát hoàn tất, giao diện trên AWS Console sẽ hiển thị như sau:

{{< img "images/Workshop/services/codepipeline.png" "AWS Console - CodePipeline" >}}

---

### Khám phá bên trong CodePipeline

Tương tự như CodeBuild, bạn có thể click trực tiếp vào tên của một pipeline (ví dụ: `publiast-staging-backend-pipeline`) để xem chi tiết luồng chạy của nó. 

Tại đây, bạn sẽ thấy trực quan các giai đoạn (Stages) nối tiếp nhau:
1. **Source:** Lấy code từ GitHub (bạn có thể thấy cả mã hash của commit, nội dung commit message như `feat: change logo text to StreamHubbb for pipeline testing`).
2. **Build:** Kích hoạt dự án CodeBuild tương ứng để biên dịch code và đóng gói. Nếu bạn bấm vào chữ *AWS CodeBuild* trong ô này, nó sẽ dẫn bạn thẳng đến màn hình chi tiết Build logs.

Giao diện này cho phép bạn theo dõi mã nguồn của mình đang đi đến bước nào và nếu có lỗi xảy ra, nó đang bị kẹt ở giai đoạn (Stage) nào.

{{< img "images/Workshop/services/codepipeline-details.png" "AWS Console - CodePipeline Details" >}}
<p align="center"><i>Chi tiết các giai đoạn bên trong CodePipeline</i></p>