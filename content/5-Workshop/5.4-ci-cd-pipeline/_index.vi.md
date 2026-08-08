---
title: "5.4. CI/CD Pipeline (AWS Native)"
weight: 4
---


Quy trình Tích hợp và Triển khai Liên tục (CI/CD) cho ứng dụng PubliCast được tự động hóa hoàn toàn bằng các công cụ dành cho nhà phát triển bản địa của AWS (AWS-native). Thay vì phụ thuộc vào các trình chạy CI/CD của bên thứ ba như GitHub Actions, chúng tôi đã cấp phát trực tiếp **AWS CodePipeline** và **AWS CodeBuild** thông qua Terraform.

## Quy trình Tích hợp & Triển khai Liên tục

Quy trình này được kích hoạt tự động mỗi khi một nhà phát triển đẩy (push) các thay đổi mã nguồn lên nhánh (branch) được chỉ định trong GitHub Monorepo của chúng ta. Bởi vì đây là một monorepo, chúng tôi đã chia tự động hóa này thành hai đường ống riêng biệt và độc lập: một cho Backend và một cho Frontend.

## Phân tích Kiến trúc Pipeline

Module Terraform `ci/cd` của chúng tôi cấp phát các thành phần cốt lõi sau để hiện thực hóa tự động hóa này:

1.  **CodeStar Connection (`aws_codestarconnections_connection`)**: 
    Thành phần này hoạt động như một cây cầu bảo mật giữa môi trường AWS và kho lưu trữ GitHub của chúng ta. Nó lắng nghe các sự kiện webhook một cách an toàn (như các đợt push code) mà không cần phải lưu trữ các token truy cập cá nhân dài hạn (Personal Access Tokens) bên trong mã Terraform.

2.  **Artifact Store (`aws_s3_bucket`)**: 
    Một bucket S3 chuyên dụng được CodePipeline sử dụng nội bộ để lưu trữ các file nén zip tạm thời (source artifacts) khi chúng được truyền từ giai đoạn Source (GitHub) sang giai đoạn Build.

3.  **Backend Pipeline (`aws_codepipeline.backend`)**:
    *   **Giai đoạn Source (Nguồn)**: Kéo mã nguồn mới nhất từ GitHub thông qua kết nối CodeStar.
    *   **Giai đoạn Build (CodeBuild)**: Chạy một container Linux được cấu hình với `privileged_mode = true` (đây là yêu cầu bắt buộc để có thể chạy Docker-in-Docker). Nó thực thi các lệnh từ file `backend/buildspec.yml`: tiến hành build 3 Docker images cho vi dịch vụ (API, Worker Light, Worker Heavy), đẩy chúng lên Amazon ECR, và kích hoạt quy trình cập nhật cuốn chiếu (rolling update) trên các dịch vụ ECS Fargate.

4.  **Frontend Pipeline (`aws_codepipeline.frontend`)**:
    *   **Giai đoạn Source (Nguồn)**: Kéo mã nguồn mới nhất từ cùng một GitHub monorepo.
    *   **Giai đoạn Build (CodeBuild)**: Thực thi các lệnh từ file `frontend/buildspec.yml`. Nó biên dịch các tài nguyên frontend (ví dụ: React/Vite) và đồng bộ hóa (sync) các file tĩnh đã biên dịch trực tiếp vào bucket S3 dành cho frontend. Sau đó, nó tự động xóa cache (invalidate) của CloudFront CDN để người dùng cuối có thể nhìn thấy các bản cập nhật ngay lập tức.

5.  **Chính sách IAM Nghiêm ngặt**:
    Để đảm bảo bảo mật, Terraform cấp phát các khối `aws_iam_role` và `aws_iam_role_policy` cụ thể. Chúng đảm bảo rằng trình chạy Backend CodeBuild chỉ có quyền chạm vào ECR và ECS, trong khi trình chạy Frontend CodeBuild chỉ có quyền tải dữ liệu lên S3 và xóa cache CloudFront.
