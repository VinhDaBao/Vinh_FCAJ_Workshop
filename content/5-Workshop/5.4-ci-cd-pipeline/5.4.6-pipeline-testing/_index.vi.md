---
title : "5.4.6. Kiểm thử Tự động hóa CI/CD"
weight : 6
---

Bây giờ khi các CodePipelines của chúng ta đã được triển khai và liên kết hoàn toàn với kho lưu trữ GitHub, đã đến lúc chứng kiến điều kỳ diệu của quy trình GitOps trên thực tế! 

Trong phần này, chúng ta sẽ thực hiện một thay đổi mã nguồn nhỏ trên máy tính cá nhân, đẩy (push) nó lên GitHub, và theo dõi quá trình AWS tự động build cũng như triển khai nó lên ứng dụng web trực tiếp của chúng ta mà không cần bất kỳ sự can thiệp thủ công nào.

## Bước 1: Thay đổi mã nguồn (Local)

Mở thư mục PubliCast monorepo trong IDE nội bộ của bạn (ví dụ: VSCode). Chúng ta sẽ mô phỏng một yêu cầu thay đổi giao diện nhỏ trên Frontend.
Cụ thể, hãy mở file `frontend/src/pages/auth/Login.jsx` và tìm đến phần logo `LeftPanel`, hãy thử sửa chữ **StreamHub** thành **StreamHubbb**:

```jsx
// frontend/src/pages/auth/Login.jsx
<div className="flex items-center gap-2 mb-auto">
  <div style={{ width: 32, height: 32, background: "#FFF", borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center" }}>
    <Wifi size={16} color="#0A0A0A" />
  </div>
  {/* Sửa chữ StreamHub thành StreamHubbb */}
  <span style={{ color: "#FFF", fontSize: 16, fontWeight: 500 }}>StreamHubb</span>
</div>
```

## Bước 2: Commit và Push lên GitHub

Mở terminal của bạn và sử dụng Git để commit sự thay đổi này, sau đó đẩy (push) lên nhánh mà CodePipeline đang theo dõi.

```bash
git add .
git commit -m "feat: change logo text to StreamHubbb for pipeline testing"
git push origin develop
```

## Bước 3: Quan sát Pipeline (In Progress)

Nhờ có webhook của CodeStar, AWS sẽ phát hiện ngay lập tức lệnh `git push` của bạn.
1. Hãy vào **AWS Console** -> **CodePipeline**.
2. Nhấp vào đường ống Frontend Pipeline của bạn.
3. Bạn sẽ thấy giai đoạn **Source** bắt đầu kéo mã nguồn mới về, và sau đó giai đoạn **Build** sẽ sáng lên với màu Xanh Dương (Trạng thái **In Progress** - Đang tiến hành build React).

{{< img "images/Workshop/services/pipeline-progress.png" "AWS Console - Pipeline In Progress" >}}

## Bước 4: Pipeline Thành công (Success)

Đợi vài phút để CodeBuild hoàn tất việc biên dịch mã nguồn giao diện (npm run build) và tải nó lên S3/CloudFront. Khi hoàn tất, tất cả các giai đoạn trong CodePipeline sẽ chuyển sang màu Xanh Lá (**Succeeded**).

{{< img "images/Workshop/services/pipeline-success.png" "AWS Console - Pipeline Success" >}}

## Bước 5: Xác minh thay đổi trên Ứng dụng thực tế

Bài kiểm tra cuối cùng! Mở một tab mới trong trình duyệt của bạn và truy cập vào trang Đăng nhập (Login) trên giao diện Frontend của bạn.

Bạn sẽ ngay lập tức nhìn thấy chữ **StreamHub** lúc đầu giờ đã được đổi thành chữ **StreamHubbb**! Mọi thứ diễn ra hoàn toàn tự động trên môi trường live internet! Bạn đã triển khai thành công một đường ống tự động hóa CI/CD đạt chuẩn.

{{< img "images/Workshop/services/login-streamhubbb.png" "Live Web App - StreamHubbb UI" >}}
