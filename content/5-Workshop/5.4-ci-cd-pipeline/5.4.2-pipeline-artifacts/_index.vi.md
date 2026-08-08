---
title : "5.4.2. Pipeline Artifacts (S3)"
weight : 2
---

## Bucket S3 cho Pipeline Artifacts

Khi AWS CodePipeline chạy, nó sẽ di chuyển dữ liệu giữa các giai đoạn khác nhau (ví dụ: từ giai đoạn **Source** sang giai đoạn **Build**). Để thực hiện việc này một cách an toàn và đáng tin cậy, CodePipeline cần một vị trí lưu trữ tạm thời.

Trong cấu hình Terraform của chúng tôi (`ci/cd/main.tf`), chúng tôi cấp phát một bucket Amazon S3 chuyên dụng dành riêng cho mục đích này:

```hcl
# ------------------------------------------------------------------------------
# 1. S3 BUCKET ĐỂ LƯU CODEPIPELINE ARTIFACTS
# ------------------------------------------------------------------------------
resource "aws_s3_bucket" "pipeline_artifacts" {
  bucket        = "${var.project}-${var.environment}-pipeline-artifacts"
  force_destroy = true
}
```

### Kiến trúc Dùng chung (Shared Architecture)
Cần lưu ý rằng bucket S3 duy nhất này được **dùng chung cho cả hai đường ống**. Cả Backend CodePipeline và Frontend CodePipeline đều sử dụng chính xác chung một bucket này làm `artifact_store` của chúng.

### Cách thức hoạt động:
1. Khi một nhà phát triển đẩy (push) code lên GitHub, giai đoạn **Source** của cả hai đường ống sẽ nén mã nguồn của kho lưu trữ (thành file zip) và tải nó lên bucket S3 này.
2. Giai đoạn **Build** (AWS CodeBuild) sẽ tải file zip này xuống từ bucket S3, giải nén nó, và bắt đầu thực thi các lệnh tương ứng trong file `buildspec.yml` cho frontend hoặc backend.

> [!NOTE]
> Chúng tôi thiết lập `force_destroy = true` trên bucket này. Lý do là vì các pipeline artifacts chỉ là dữ liệu tạm thời và sẽ được tạo lại trong mỗi lần chạy. Nếu chúng ta từng cần phá hủy (destroy) môi trường Terraform (giống như ở phần cuối của workshop này), chúng ta muốn Terraform tự động xóa bucket này ngay cả khi nó có chứa các file zip artifact rác còn sót lại.
