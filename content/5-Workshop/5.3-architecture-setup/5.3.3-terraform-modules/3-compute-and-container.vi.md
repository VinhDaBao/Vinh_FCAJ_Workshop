---
title: "5.3.3.3. Lớp Tính toán & Container"
weight: 3
---

Lớp này chứa môi trường thực thi thực sự cho các vi dịch vụ (API, Worker Light, Worker Heavy) đã được tách rời của chúng ta. Thay vì phải quản lý các máy ảo truyền thống, chúng ta sử dụng công nghệ container không máy chủ (serverless). Điều này có nghĩa là AWS sẽ lo toàn bộ việc bảo trì máy chủ bên dưới, và chúng chỉ phải trả tiền chính xác cho lượng CPU và bộ nhớ mà các container của chúng ta tiêu thụ trong lúc chạy.

## Elastic Container Registry (`ecr`)
**Mục đích:** Một kho lưu trữ container Docker được quản lý hoàn toàn. Hãy coi nó như một phiên bản riêng tư, bảo mật của Docker Hub hoặc GitHub, nhưng được thiết kế chuyên biệt để lưu trữ các bản build image ứng dụng của bạn.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_ecr_repository`: Tạo một kho lưu trữ riêng (private) cho mỗi vi dịch vụ.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name`: Tên của kho lưu trữ (ví dụ: `publicast-api`).
        *   `image_scanning_configuration`: Chứa khối `scan_on_push = true`. Nhờ vậy, mỗi khi một lập trình viên đẩy một Docker image mới lên, AWS sẽ tự động quét nó nhằm phát hiện các lỗ hổng hệ điều hành và thư viện, đảm bảo chúng ta không triển khai code độc hại.
        *   `force_delete = true`: Cho phép Terraform phá hủy kho lưu trữ ngay cả khi bên trong vẫn còn chứa image (rất tiện lợi cho môi trường workshop).
    
    {{< img "images/Workshop/services/ecr.png" "AWS Console - ECR Repository" >}}
    <p align="center"><i>AWS Console - ECR Repository</i></p>
    
    {{< img "images/Workshop/services/ecr-repo.png" "AWS Console - ECR Repository" >}}
    <p align="center"><i>AWS Console - ECR Images</i></p>
    
*   `aws_ecr_lifecycle_policy`: Mỗi lần chúng ta cập nhật code qua hệ thống CI/CD, một image mới sẽ được lưu vào ECR. Để ngăn chi phí lưu trữ phình to theo thời gian, chúng ta định nghĩa một khối chính sách vòng đời JSON.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `repository`: Gắn chính sách này vào kho lưu trữ vừa tạo.
        *   `policy`: Một chuỗi JSON định nghĩa luật giữ lại image. Cụ thể, chúng ta đặt `countType = "imageCountMoreThan"` và `countNumber = 10`. Nó hướng dẫn ECR chỉ giữ lại 10 image mới nhất và tự động xóa các image cũ hơn, giữ cho kho lưu trữ luôn sạch sẽ và tiết kiệm chi phí.
    
    {{< img "images/Workshop/services/ecr-lifecycle.png" "AWS Console - ECR Lifecycle Policy" >}}
    <p align="center"><i>AWS Console - ECR Lifecycle Policy</i></p>

📁 **Source Code:** Mở file `terraform/modules/ecr/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## ECS Cluster (`ecs`)
**Mục đích:** Một nhóm logic của các task (tác vụ) hoặc service (dịch vụ). Nó đóng vai trò là ranh giới quản lý cho các ứng dụng container của chúng ta. Nếu ECR là nơi *lưu trữ* container, thì ECS Cluster là khu vực được chỉ định để chúng thực sự *chạy*.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_ecs_cluster`: Tạo không gian tên (namespace) và mặt phẳng quản lý (management plane).
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name`: Tên của cluster (ví dụ: `publicast-staging-cluster`). Đây đóng vai trò là trung tâm quản lý cho các dịch vụ serverless của chúng ta.
    
    {{< img "images/Workshop/services/ecs-cluster-main.png" "AWS Console - ECS Cluster" >}}
    <p align="center"><i>AWS Console - ECS Cluster</i></p>

📁 **Source Code:** Mở file `terraform/modules/ecs/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## ECS Task Definitions (`ecs_task_definition`)
**Mục đích:** Hoạt động như một bản thiết kế (blueprint) mô tả chính xác cách một Docker container nên được khởi chạy. Nếu bạn đã quen với Docker, thì Task Definition rất giống với một file `docker-compose.yml`, nhưng được thiết kế riêng cho AWS.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_ecs_task_definition`: Bản thiết kế tổng thể.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `network_mode = "awsvpc"`: Yêu cầu bắt buộc đối với Fargate để mỗi container nhận được một địa chỉ IP riêng.
        *   `requires_compatibilities = ["FARGATE"]`: Xác thực rằng task này được xây dựng cho môi trường serverless.
        *   `cpu` & `memory`: Định nghĩa tổng giới hạn tài nguyên cần thiết cho toàn bộ task để ngăn nó tiêu thụ vô hạn (ví dụ: `256` CPU units, `512` MB memory).
        *   `execution_role_arn` & `task_role_arn`: Gắn các vai trò IAM cụ thể mà chúng ta đã xây dựng ở Lớp 1 vào task này.
    
    {{< img "images/Workshop/services/taskdef-main.png" "AWS Console - Task Definition" >}}
    <p align="center"><i>AWS Console - Task Definition</i></p>
    
    {{< img "images/Workshop/services/taskdef-config.png" "AWS Console - Task Definition" >}}
    <p align="center"><i>AWS Console - Task Definition Configure</i></p>
    
*   `container_definitions` (Định dạng JSON): Đây là trái tim của cấu hình.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `image`: URL trỏ đến kho ECR.
        *   `portMappings`: Mở cổng mạng cụ thể (ví dụ: `containerPort: 8080`) để load balancer có thể kết nối.
        *   `logConfiguration`: Định tuyến các dòng lệnh `console.log` của ứng dụng về CloudWatch Logs.
        *   Mảng `secrets`: Bên trong định nghĩa container, chúng ta đối mặt với một thách thức bảo mật: làm sao để cung cấp mật khẩu database cho container mà không phải viết chúng ra dưới dạng plain text? Thay vì dùng mảng `environment` tiêu chuẩn, chúng ta dùng mảng `secrets` để ánh xạ các biến môi trường trực tiếp đến các ARN của các đối tượng trong AWS Secrets Manager. AWS sẽ tự động tiêm mật khẩu một cách an toàn vào chính xác thời điểm container khởi động.
    
    {{< img "images/Workshop/services/taskdef-json.png" "AWS Console - Task Def JSON" >}}
    <p align="center"><i>AWS Console - Task Def JSON</i></p>

📁 **Source Code:** Mở file `terraform/modules/ecs_task_definition/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## ECS Services (`ecs_service`)
**Mục đích:** Duy trì một số lượng cụ thể các bản sao (instances) của một Task Definition chạy đồng thời. Nó đóng vai trò như một người giám sát—nếu một container bị sập, ECS Service sẽ tự động khởi động một container mới để thay thế. Nó cũng xử lý việc kết nối các container với Bộ cân bằng tải.

**Tài nguyên Terraform chính:** 
**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_ecs_service`: Kết nối bản thiết kế Task Definition của bạn với Cluster. Như hiển thị trong giao diện tổng quan, chúng ta đã tách rời ứng dụng thành một bộ 3 dịch vụ riêng biệt chạy đồng thời:
    
    {{< img "images/Workshop/services/ecs-service.png" "AWS Console - ECS Services Overview" >}}
    <p align="center"><i>AWS Console - Tổng quan ECS Services</i></p>

    Hãy cùng phân tích vai trò cụ thể của bộ 3 này:

    1.  **Dịch vụ API (`api-service`)**: Core backend xử lý các yêu cầu HTTP trực tiếp từ người dùng. Điểm đặc biệt của service này là có thêm khối `load_balancer` để tự động đăng ký với ALB Target Group nhằm nhận lưu lượng web.
        {{< img "images/Workshop/services/ecs-service-api.png" "AWS Console - ECS API Service" >}}
        <p align="center"><i>AWS Console - ECS API Service</i></p>

    2.  **Dịch vụ Worker Nhẹ (`worker-light-service`)**: Chạy các công việc nền nhẹ nhàng (như gửi email, dọn dẹp cơ sở dữ liệu) thông qua BullMQ. Nó hoạt động hoàn toàn độc lập và bảo mật, không cần kết nối với Load Balancer.
        {{< img "images/Workshop/services/ecs-service-light.png" "AWS Console - ECS Worker Light Service" >}}
        <p align="center"><i>AWS Console - ECS Worker Light Service</i></p>

    3.  **Dịch vụ Worker Nặng (`worker-heavy-service`)**: Chuyên trị các tác vụ tiêu tốn nhiều CPU và RAM (như mã hóa và chia nhỏ video). Việc tách riêng service này đảm bảo các tiến trình nặng sẽ không bao giờ làm sập hay làm chậm API chính của chúng ta.
        {{< img "images/Workshop/services/ecs-service-heavy.png" "AWS Console - ECS Worker Heavy Service" >}}
        <p align="center"><i>AWS Console - ECS Worker Heavy Service</i></p>

    *   **Các Thuộc tính Cấu hình Chung:** 
        *   `launch_type = "FARGATE"`: Chỉ đạo service sử dụng công nghệ serverless.
        *   `desired_count`: Số lượng container bản sao chúng ta mong muốn chạy.
        *   Khối `network_configuration`: Đặt một cách an toàn các container đang chạy vào trong các `subnets` (Private Subnet) mà chúng ta đã tạo ở Lớp 1, gắn tường lửa `security_groups`, và thiết lập `assign_public_ip = false` để đảm bảo an ninh tối đa.
    
📁 **Source Code:** Mở file `terraform/modules/ecs_service/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.
