---
title: "5.3.3.1. Lớp Mạng & Bảo mật"
weight: 1
---

Lớp này thiết lập nền tảng bảo mật vững chắc cho nền tảng PubliCast. Nó cô lập các tài nguyên của chúng ta khỏi internet công cộng và kiểm soát chặt chẽ các quyền truy cập. Hãy tưởng tượng lớp này giống như những bức tường, cánh cửa và nhân viên bảo vệ cho trung tâm dữ liệu ảo của bạn.

## Virtual Private Cloud (`vpc`)
**Mục đích:** Cung cấp một môi trường mạng riêng biệt, bị cô lập hoàn toàn trên AWS. Nó đóng vai trò là vành đai bảo vệ ngoài cùng cho toàn bộ hạ tầng của chúng ta. Nếu không có VPC, bạn không thể triển khai các tài nguyên AWS hiện đại một cách an toàn.

**Tài nguyên Terraform chính:** 
Để xây dựng một mạng có tính sẵn sàng cao và tiết kiệm chi phí, module này điều phối một số thành phần cốt lõi:

**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_vpc`: Hãy coi VPC là lớp vỏ bọc chính cho mạng lưới của bạn. Tài nguyên này định nghĩa ranh giới mạng và gán một dải địa chỉ IP chính (được gọi là khối IPv4 CIDR). Mọi thứ chúng ta xây dựng sẽ nằm gọn bên trong dải IP này.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `cidr_block`: Được cấp bởi biến của chúng ta (ví dụ: `10.0.0.0/16`), quyết định kích thước của mạng lưới.
        *   `tags`: Được hợp nhất (merge) động với chuỗi `${var.project}-${var.environment}-vpc` để dễ dàng nhận diện trên giao diện AWS.
    
    {{< img "images/Workshop/services/vpc.png" "AWS Console - Main VPC" >}}
    <p align="center"><i>AWS Console - Main VPC</i></p>
    
*   `aws_subnet`: Subnet (Mạng con) là những phần nhỏ hơn, dễ quản lý hơn được chia ra từ mạng VPC của bạn. Như bạn thấy trong hình bên dưới, Terraform tạo ra chính xác **4 subnet** cho dự án của chúng ta. Tại sao lại là 4?
    *   Thứ nhất, chúng ta chia theo **Khả năng truy cập**: chúng ta cần các **Public Subnets** (mạng công khai, dành cho các tài nguyên phải giao tiếp với internet như Load Balancer) và **Private Subnets** (mạng riêng tư, để giấu kín an toàn cơ sở dữ liệu và container ứng dụng khỏi thế giới bên ngoài).
    *   Thứ hai, chúng ta chia theo **Tính Sẵn sàng Cao (High Availability)**: AWS nhóm các trung tâm dữ liệu vật lý của họ thành các Availability Zones (AZ - Vùng tính sẵn sàng). Nếu toàn bộ một trung tâm dữ liệu bị mất điện, chúng ta không muốn ứng dụng của mình bị sập. Bằng cách dùng vòng lặp `count` của Terraform để triển khai vắt ngang qua **2 AZ khác nhau**, chúng ta có một ma trận: (1 Public + 1 Private) × 2 AZ = 4 Subnets. Điều này đảm bảo khả năng chịu lỗi (fault tolerance) tuyệt đối.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `availability_zone`: Sử dụng `data.aws_availability_zones` để gán động subnet vào nhiều trung tâm dữ liệu vật lý khác nhau.
        *   `map_public_ip_on_launch = true`: *Chỉ* áp dụng cho các Public subnets để các máy chủ bên trong tự động nhận được IP công cộng (Public IP).
    
    {{< img "images/Workshop/services/subnets.png" "AWS Console - Subnets" >}}
    <p align="center"><i>AWS Console - Subnets</i></p>
    
*   `aws_internet_gateway`: Đây chính là "cánh cửa" kết nối ra internet. Được gắn vào VPC và định tuyến qua Public Route Table, Internet Gateway cho phép các tài nguyên nằm trong Public Subnet (như Load Balancer của chúng ta) nhận được lưu lượng truy cập web từ người dùng của bạn.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `vpc_id`: Chỉ định rõ Gateway này thuộc về VPC nào mà chúng ta vừa tạo.
    
    {{< img "images/Workshop/services/igw.png" "AWS Console - Internet Gateway" >}}
    <p align="center"><i>AWS Console - Internet Gateway</i></p>
    
*   `aws_nat_gateway` & `aws_eip`: Sẽ ra sao nếu một máy chủ nằm trong Private Subnet cần tải xuống một bản cập nhật phần mềm hoặc gọi đến một API của bên thứ ba (như Stripe hay Meta)? Vì nó là mạng riêng tư, nó không thể dùng trực tiếp Internet Gateway. Chúng ta đặt một NAT Gateway ở Public Subnet để làm "người trung gian" an toàn. Nó nhận yêu cầu từ Private Subnet, lấy dữ liệu từ internet về và trả lại, tất cả quá trình này diễn ra mà không hề để lộ máy chủ riêng tư trước các cuộc tấn công từ internet.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `allocation_id`: Kết nối NAT Gateway với một Elastic IP (`aws_eip`) để nó có một địa chỉ IP tĩnh trên internet.
        *   `subnet_id`: Chỉ định rõ AWS phải đặt Gateway này ở bên trong *Public* subnet.
    
    {{< img "images/Workshop/services/natgw.png" "AWS Console - NAT Gateway" >}}
    <p align="center"><i>AWS Console - NAT Gateway</i></p>
    
*   `aws_vpc_endpoint`: Được cấu hình theo kiểu `Gateway` dành riêng cho Amazon S3. Thông thường, việc tải các file media dung lượng lớn từ S3 về máy chủ sẽ khiến dữ liệu phải đi vòng ra internet công cộng rồi mới quay lại, gây tốn rất nhiều tiền phí xử lý dữ liệu của NAT Gateway. VPC Endpoint tạo ra một "đường tắt" bí mật nối thẳng đến S3. Nó sửa đổi Private Route Table để mọi lưu lượng hướng tới S3 đều nằm hoàn toàn trong mạng nội bộ AWS, giúp chúng ta tiết kiệm rất nhiều tiền và tăng tốc độ tải.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `vpc_endpoint_type = "Gateway"`: Cụ thể hóa việc dùng Gateway endpoint (miễn phí) thay vì Interface endpoint.
        *   `service_name`: Trỏ động đến dịch vụ `com.amazonaws.${region}.s3`.
        *   `route_table_ids`: Tự động tiêm đường tắt này vào các Bảng Định tuyến Riêng tư (Private Route Tables) của chúng ta.
    
    {{< img "images/Workshop/services/endpoint.png" "AWS Console - VPC Endpoints" >}}
    <p align="center"><i>AWS Console - VPC Endpoints</i></p>

📁 **Source Code:** Mở file `terraform/modules/vpc/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## Security Groups (`security_group`)
**Mục đích:** Hoạt động như các tường lửa mạng ảo để cho phép hoặc từ chối lưu lượng truy cập ở cấp độ từng instance (máy chủ). Nếu VPC là bức tường bao quanh tòa nhà, thì Security Groups chính là những ổ khóa cửa của từng căn phòng cụ thể.

**Tài nguyên Terraform chính:** 
Chúng ta áp dụng kiến trúc "Zero Trust" (Không tin cậy bất kỳ ai) bên trong VPC. Điều này có nghĩa là ngay cả khi một tài nguyên nằm trong mạng riêng tư của chúng ta, nó cũng không được tin tưởng một cách mặc định.

**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_security_group`: Tạo ranh giới tường lửa logic. Chúng ta tạo ra các security group riêng biệt cho từng lớp của ứng dụng.
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `name` & `description`: Đặt tên và mô tả rõ ràng mục đích của tường lửa.
        *   `vpc_id`: Ràng buộc security group này với VPC tùy chỉnh của chúng ta, thay vì VPC mặc định.
    
    {{< img "images/Workshop/services/sg.png" "AWS Console - Security Groups List" >}}
    <p align="center"><i>AWS Console - Security Groups List</i></p>

Dưới đây chúng ta chia chủ đề `security_group` thành 4 nhóm cụ thể được sử dụng trong dự án PubliCast.

### 1) ALB Security Group (`alb-sg`)
**Mục đích:** Chịu trách nhiệm tiếp nhận lưu lượng (traffic) từ Internet và chuyển tiếp tới các ECS Tasks thông qua Application Load Balancer.
*   **Ingress (Vào):** Cho phép HTTP (80) và HTTPS (443) từ bất cứ đâu (`0.0.0.0/0` và IPv6 `::/0`).
*   **Egress (Ra):** Cho phép lưu lượng đi ra kết nối tới `ecs-tasks-sg` ở các cổng ứng dụng để truyền truy vấn, cũng như để thực hiện health checks.

{{< img "images/Workshop/services/alb-sg.png" "AWS Console - ALB Security Group" >}}
<p align="center"><i>AWS Console - ALB Security Group</i></p>

### 2) ECS Tasks Security Group (`ecs-tasks-sg`)
**Mục đích:** Được gắn trực tiếp lên các container backend đang chạy (Node.js API, Workers). Nó ẩn hoàn toàn ứng dụng khỏi internet công cộng.
*   **Ingress:** *Chỉ* cho phép lưu lượng đi vào từ `alb-sg` ở các cổng của ứng dụng.
*   **Egress:** Cho phép lưu lượng đi ra để kết nối tới S3 (qua VPC endpoints), Secrets Manager, RDS, và Redis.

{{< img "images/Workshop/services/ecs-sg.png" "AWS Console - ECS Tasks Security Group" >}}
<p align="center"><i>AWS Console - ECS Tasks Security Group</i></p>

### 3) Redis Security Group (`redis-sg`)
**Mục đích:** Bảo vệ cụm ElastiCache Redis, đảm bảo chỉ có các container ứng dụng nội bộ được cấp quyền mới có thể truy cập bộ đệm.
*   **Ingress:** *Chỉ* cho phép truy cập từ `ecs-tasks-sg` trên cổng `6379`.
*   **Egress:** Bị giới hạn hoàn toàn trong mạng nội bộ.

{{< img "images/Workshop/services/redis-sg.png" "AWS Console - Redis Security Group" >}}
<p align="center"><i>AWS Console - Redis Security Group</i></p>

### 4) RDS Security Group (`rds-sg`)
**Mục đích:** Bảo mật cơ sở dữ liệu quan hệ (MySQL). Đây là thành phần được canh gác nghiêm ngặt nhất vì nó chứa dữ liệu vĩnh viễn của chúng ta.
*   **Ingress:** *Chỉ* cho phép truy cập từ `ecs-tasks-sg` thông qua cổng database (ví dụ: `3306`).
*   **Egress:** Được giới hạn tối giản nhất có thể.

{{< img "images/Workshop/services/rds-sg.png" "AWS Console - RDS Security Group" >}}
<p align="center"><i>AWS Console - RDS Security Group</i></p>

> [!TIP]
> **Tham chiếu Security Group (Security Group Referencing):** Đây là tính năng bảo mật mạnh mẽ nhất của chúng ta! Hãy để ý đối với RDS và Redis, chúng ta *không* cho phép các dải IP rộng như `10.0.0.0/16`. Thay vào đó, chúng ta thiết lập thuộc tính `source_security_group_id` trỏ chính xác đến ID của ECS Tasks Security Group. Ngay cả khi hacker chiếm được một máy chủ khác trong cùng một subnet, chúng cũng không thể truy cập vào database vì chúng không hề "đeo huy hiệu" của nhóm `ecs-tasks-sg`.

📁 **Source Code:** Mở file `terraform/modules/security_group/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.

## Quản lý Định danh và Truy cập (`iam`)
**Mục đích:** Kiểm soát những định danh nào (người dùng, dịch vụ, hoặc role) có thể truy cập vào các API và tài nguyên AWS cụ thể. Nếu Security Groups kiểm soát truy cập *mạng*, thì IAM kiểm soát truy cập *API*.

**Tài nguyên Terraform chính:** 
Chúng ta tuân thủ nguyên tắc đặc quyền tối thiểu (least privilege) cho các ứng dụng container của mình, nghĩa là chúng ta chỉ cấp cho chúng chính xác những quyền chúng cần để làm việc, và không thừa một quyền nào khác.

**Kết quả:** Sau khi chạy lệnh `terraform apply`, bạn có thể xác minh các thành phần riêng lẻ này trên AWS Console:

*   `aws_iam_role`: Tạo các roles (vai trò). Hãy nghĩ về một role như một "chiếc mũ" mà một dịch vụ có thể đội lên để nhận được các quyền hạn tạm thời. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `assume_role_policy`: Chúng ta định nghĩa một khối JSON ở đây khẳng định rằng chỉ có dịch vụ AWS ECS (`ecs-tasks.amazonaws.com`) mới được phép đội chiếc mũ này.
    
    {{< img "images/Workshop/services/ecs-iam-role.png" "AWS Console - IAM Role" >}}
    <p align="center"><i>AWS Console - IAM Role</i></p>
    
*   **Task Execution Role (`aws_iam_role_policy_attachment`):** Chúng ta dùng tài nguyên này để đính kèm chính sách `AmazonECSTaskExecutionRolePolicy` do AWS quản lý. Role này dành cho tác nhân (agent) ẩn của ECS chạy trong nền. 
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `policy_arn`: Trỏ đến mã định danh chính sách (ARN) chính thức của AWS, cấp cho tác nhân này quyền tải Docker image an toàn từ ECR về và đẩy đầu ra `console.log` lên CloudWatch.
*   **Task Role (`aws_iam_role_policy`):** Chúng ta tạo các chính sách JSON tùy chỉnh, nội tuyến (inline) dành riêng cho các container ứng dụng Node.js thực tế của bạn. Như bạn thấy trên giao diện console, chúng ta đính kèm các chính sách cụ thể:
    *   **`backend-s3-access-policy`**: Cấp cho ứng dụng quyền `s3:PutObject`, `s3:GetObject` và `s3:DeleteObject` để người dùng có thể tải lên và quản lý file media trong S3 bucket của dự án.
    *   **`[project]-ecs-exec-policy`**: Cấp các quyền như `secretsmanager:GetSecretValue` (để ứng dụng lấy mật khẩu database một cách bảo mật) và `ssmmessages:CreateControlChannel` (cho phép quản trị viên mở một dòng lệnh terminal an toàn đi thẳng vào bên trong container để gỡ lỗi thông qua tính năng ECS Exec).
    *   **Các Thuộc tính Được Cấu hình:** 
        *   `policy`: Dùng hàm `jsonencode()` để nhúng an toàn các tài liệu chính sách JSON nghiêm ngặt này (gồm Effect, Action, Resource). Nếu ứng dụng của bạn không may bị chiếm quyền, hacker cũng chỉ có thể thực hiện chính xác những hành động này, giúp giới hạn tối đa phạm vi ảnh hưởng.
    
    {{< img "images/Workshop/services/iam-policy.png" "AWS Console - IAM Policy" >}}
    <p align="center"><i>AWS Console - IAM Policy</i></p>

📁 **Source Code:** Mở file `terraform/modules/iam/main.tf` trong IDE của bạn để xem toàn bộ cấu hình.
