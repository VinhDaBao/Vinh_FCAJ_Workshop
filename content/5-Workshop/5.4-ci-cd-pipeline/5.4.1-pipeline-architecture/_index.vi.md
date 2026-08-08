---
title : "5.4.1. Sơ đồ Kiến trúc Pipeline"
weight : 1
---

## Trực quan hóa Quy trình CI/CD

Trước khi đi sâu vào các tài nguyên Terraform cụ thể dùng để xây dựng pipeline của chúng ta, hãy dành một khoảnh khắc để hiểu về kiến trúc tổng thể của cách thức mã nguồn (code) di chuyển từ máy tính cá nhân của một nhà phát triển vào môi trường sản xuất (production) trên AWS.

Như được minh họa bên dưới, chúng tôi sử dụng một kiến trúc ống dẫn kép (dual-pipeline) (một dành cho các dịch vụ Backend ECS, và một dành cho ứng dụng Frontend S3/CloudFront). Cả hai đường ống này đều được kích hoạt bởi một kết nối CodeStar duy nhất nối với kho lưu trữ GitHub monorepo của chúng ta.

{{< img "images/Workshop/services/ci_cd.drawio.png" "CI/CD Pipeline Architecture" >}}
