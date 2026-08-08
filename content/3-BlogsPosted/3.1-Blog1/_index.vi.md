---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# ROUTE 53 TRONG DỰ ÁN QUẢN LÝ NỘI DUNG CỦA NHÓM

Dự án của nhóm mình là một nền tảng quản lý nội dung, cho phép người dùng tạo bài viết một lần và đăng lên nhiều mạng xã hội khác nhau như Facebook, LinkedIn hay X (Twitter) thông qua các API chính thức. Để thực hiện được điều này, hệ thống phải tích hợp cơ chế OAuth của từng nền tảng nhằm cho phép người dùng liên kết tài khoản một cách an toàn.

Mỗi nền tảng đều yêu cầu khai báo **Redirect URI (Callback URL)** cố định trước khi ứng dụng được phép hoạt động. Vì vậy, việc sở hữu một tên miền ổn định là rất quan trọng. Thay vì sử dụng địa chỉ IP hoặc URL tạm thời trong quá trình phát triển, nhóm mình cấu hình subdomain như `api.example.com` thông qua Route 53 để làm địa chỉ callback cho toàn bộ hệ thống.

Nhờ đó, khi người dùng nhấn nút **"Kết nối tài khoản"**, các nền tảng sẽ chuyển hướng về đúng endpoint của backend sau khi quá trình xác thực hoàn tất. Sau này, dù nhóm có thay đổi hạ tầng, chuyển sang Load Balancer khác hoặc triển khai lại ECS thì Callback URL vẫn giữ nguyên. Chỉ cần cập nhật bản ghi trong Route 53 là toàn bộ hệ thống tiếp tục hoạt động mà không cần cấu hình lại trên từng nền tảng.

Đây cũng là lúc tụi mình nhận ra một tên miền không chỉ phục vụ việc truy cập website, mà còn là thành phần quan trọng giúp các dịch vụ bên ngoài nhận diện và tin tưởng ứng dụng của mình. Việc quản lý DNS bằng Route 53 đã giúp quá trình tích hợp với nhiều mạng xã hội trở nên ổn định và dễ bảo trì hơn rất nhiều.

## Bắt đầu từ việc quản lý tên miền

Ban đầu, nhóm đã mua một tên miền để phục vụ cho dự án. Thay vì sử dụng trực tiếp DNS của nhà cung cấp tên miền, tụi mình quyết định quản lý toàn bộ DNS bằng Amazon Route 53. Lý do khá đơn giản: các dịch vụ AWS được tích hợp rất tốt với Route 53, giúp việc cấu hình sau này thuận tiện hơn rất nhiều.

Sau khi tạo Hosted Zone, nhóm nhận được bốn Name Server (NS) do AWS cung cấp. Công việc tiếp theo là cập nhật Name Server này tại nhà đăng ký tên miền. Đây là bước khá dễ bỏ sót vì nếu chưa đổi Name Server, mọi thay đổi trong Route 53 sẽ hoàn toàn không có hiệu lực.

Sau vài phút đến vài giờ để DNS được cập nhật toàn cầu, nhóm bắt đầu tạo các bản ghi đầu tiên:

* **A hoặc Alias Record** để trỏ tên miền chính đến Application Load Balancer.
* **CNAME Record** cho các subdomain như `api.example.com`.
* **MX Record** nếu cần sử dụng email.
* **TXT Record** phục vụ xác thực với nhiều dịch vụ khác nhau.

Lúc này tụi mình mới hiểu rằng Route 53 không chỉ đơn thuần là nơi "trỏ domain", mà còn là nơi quản lý gần như toàn bộ thông tin liên quan đến tên miền.

## Lần đầu xác thực tên miền bằng TXT Record

Trong quá trình tích hợp các dịch vụ bên thứ ba như Google OAuth, TikTok, Facebook hay một số nền tảng gửi email, nhóm liên tục gặp yêu cầu:

> Please verify your domain.

Lúc đầu, tụi mình cứ nghĩ việc xác thực sẽ phải upload một file HTML lên website giống như Google Search Console. Nhưng sau khi đọc tài liệu mới biết phần lớn các dịch vụ hiện nay đều sử dụng **TXT Record** để xác minh quyền sở hữu tên miền.

Ví dụ, một dịch vụ sẽ yêu cầu tạo bản ghi như:

```text
Type: TXT
Name: _verification.example.com
Value: x7d91abdf8c....
```

Sau khi thêm bản ghi này vào Route 53, dịch vụ sẽ kiểm tra DNS toàn cầu. Nếu tìm thấy đúng giá trị thì sẽ xác nhận rằng nhóm thực sự sở hữu tên miền đó.

Đây là một trải nghiệm khá thú vị vì tụi mình nhận ra rất nhiều hệ thống trên Internet hoạt động dựa trên DNS chứ không phải chỉ có việc truy cập website.

Từ đó về sau, gần như mọi dịch vụ mới đều yêu cầu thêm ít nhất một TXT Record để xác minh.

## Route 53 và AWS Certificate Manager

Một trong những lần khiến tụi mình "vỡ òa" là khi tạo chứng chỉ HTTPS.

Ban đầu nhóm nghĩ rằng muốn có HTTPS sẽ phải mua SSL Certificate giống như trước đây.

Nhưng **AWS Certificate Manager (ACM)** cho phép cấp chứng chỉ hoàn toàn miễn phí.

Điều kiện là phải chứng minh mình sở hữu tên miền.

Lại một lần nữa DNS xuất hiện.

Sau khi yêu cầu cấp chứng chỉ, ACM sinh ra một CNAME Record như:

```text
Name:
_abcd123.example.com

Value:
_xyz.acm-validations.aws
```

Chỉ cần thêm bản ghi này vào Route 53.

Vài phút sau, trạng thái của chứng chỉ chuyển từ:

```text
Pending Validation
```

sang:

```text
Issued
```

Nhóm gần như không cần làm thêm bất kỳ thao tác nào khác.

Điều tuyệt vời là nếu sử dụng Route 53 và ACM cùng một tài khoản AWS, nhiều trường hợp AWS còn tự động tạo bản ghi xác thực giúp nhóm.

Sau khi gắn chứng chỉ vào Application Load Balancer, website đã có HTTPS hoàn chỉnh.

* Không cần mua SSL.
* Không cần cài đặt Let's Encrypt.
* Không cần tự gia hạn.

ACM sẽ tự động gia hạn chứng chỉ trước khi hết hạn miễn là bản ghi xác thực vẫn còn tồn tại.

Đây có lẽ là một trong những tính năng mà tụi mình thích nhất khi sử dụng AWS.

## Học được cách phân chia subdomain

Ban đầu, mọi thành phần đều chạy trên tên miền chính.

Sau một thời gian, nhóm bắt đầu tách thành nhiều subdomain khác nhau.

Ví dụ:

```text
www.example.com    → Website giới thiệu
api.example.com    → Backend
pub.example.com    → Website tĩnh
docs.example.com   → Tài liệu
cdn.example.com    → Tài nguyên tĩnh
```

Nhờ Route 53, việc quản lý các subdomain rất trực quan.

Nếu sau này thay đổi kiến trúc thì chỉ cần cập nhật Record tương ứng mà không ảnh hưởng tới các dịch vụ khác.

## Alias Record giúp đơn giản hơn rất nhiều

Một điều khá hay mà tụi mình học được là trên AWS thường không cần sử dụng CNAME cho tên miền gốc.

Thay vào đó có thể dùng **Alias Record**.

Alias có thể trỏ trực tiếp đến:

* Application Load Balancer
* CloudFront
* S3 Static Website
* API Gateway

Ưu điểm là không cần quan tâm IP thật của dịch vụ.

Nếu AWS thay đổi hạ tầng phía sau, Route 53 vẫn tự xử lý.

Đây là điều mà DNS truyền thống không làm được.

## Những bài học "đắt giá"

Trong quá trình sử dụng Route 53, nhóm cũng gặp không ít lỗi.

Có lần tụi mình tạo Hosted Zone là:

```text
pub.example.com
```

trong khi đáng lẽ phải là:

```text
example.com
```

Kết quả là toàn bộ bản ghi đều không hoạt động như mong muốn.

Một lần khác, nhóm thêm TXT Record đúng giá trị nhưng xác thực vẫn thất bại.

Sau khi kiểm tra mới phát hiện Name Server của tên miền vẫn đang trỏ về nhà cung cấp cũ thay vì Route 53.

Cũng có lúc cả nhóm tưởng Route 53 bị lỗi vì vừa thêm bản ghi xong nhưng hệ thống vẫn báo chưa tìm thấy.

Sau đó mới hiểu DNS cần thời gian để cập nhật và còn phụ thuộc vào TTL cũng như bộ nhớ đệm DNS của từng nhà mạng.

Những lỗi này tuy nhỏ nhưng giúp tụi mình hiểu rõ hơn cách DNS hoạt động thay vì chỉ làm theo hướng dẫn.

## Nhìn lại

Sau nhiều tháng sử dụng, tụi mình nhận ra Amazon Route 53 không đơn thuần là một dịch vụ quản lý DNS. Nó là nền tảng kết nối rất nhiều thành phần của hệ thống lại với nhau. Từ việc trỏ tên miền đến Application Load Balancer, xác thực quyền sở hữu tên miền bằng TXT Record, cấp chứng chỉ HTTPS thông qua AWS Certificate Manager, cho đến cấu hình Callback URL cho các dịch vụ OAuth, thanh toán và webhook, gần như mọi bước triển khai đều ít nhiều liên quan đến Route 53.

Điều mà nhóm học được không chỉ là cách tạo một bản ghi DNS, mà còn là tư duy quản lý tên miền một cách bài bản. Khi mọi cấu hình DNS được tổ chức rõ ràng, việc mở rộng hệ thống, thay đổi hạ tầng hay tích hợp thêm dịch vụ mới đều trở nên đơn giản hơn rất nhiều.

Đối với một nhóm sinh viên lần đầu xây dựng sản phẩm trên nền tảng cloud, Route 53 là một trong những dịch vụ mang lại nhiều bài học thực tế nhất. Nó giúp tụi mình hiểu rằng một tên miền không chỉ dùng để truy cập website, mà còn là "chìa khóa" để xác minh danh tính của hệ thống, thiết lập kết nối an toàn và tích hợp với rất nhiều dịch vụ hiện đại trên Internet.

Các điểm chính cần nắm:

* Hiểu cách sử dụng Amazon Route 53 để quản lý Hosted Zone và DNS Record.
* Hiểu vai trò của domain cố định trong OAuth Redirect URI/Callback URL.
* Biết cách sử dụng A, Alias, CNAME, MX và TXT Record trong các trường hợp thực tế.
* Hiểu cách TXT Record được sử dụng để xác minh quyền sở hữu domain.
* Hiểu cách Route 53 kết hợp với AWS Certificate Manager để xác thực chứng chỉ HTTPS.
* Nhận thức được tầm quan trọng của DNS trong việc giảm phụ thuộc vào hạ tầng backend cụ thể.
* Hiểu rằng việc phân chia domain và subdomain giúp hệ thống dễ quản lý, mở rộng và bảo trì hơn.

[Link Blog 1](https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2230396147725345)