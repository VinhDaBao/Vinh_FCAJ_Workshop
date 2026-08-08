---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---



# MỘT BÀI HỌC 30 USD CHỈ VÌ BỎ QUA MỘT EMAIL TỪ AWS

## Trong quá trình triển khai dự án

Trong quá trình triển khai dự án trên AWS, nhóm mình từng gặp một sự cố khá đáng nhớ liên quan đến Amazon RDS. Mặc dù hệ thống vẫn hoạt động bình thường và không xảy ra lỗi nào về mặt kỹ thuật, nhưng chỉ vì không để ý đến một email cảnh báo từ AWS mà nhóm đã phải trả thêm khoảng **30 USD** cho một khoản phí hoàn toàn có thể tránh được.

Khi đó, cơ sở dữ liệu của dự án đang sử dụng Amazon RDS MySQL phiên bản 8.0. AWS đã gửi thông báo đến tài khoản rằng MySQL 8.0 sắp kết thúc thời gian Standard Support và khuyến nghị người dùng chủ động nâng cấp lên phiên bản mới hơn trước thời hạn. Theo thông báo, sau khi MySQL 8.0 hết vòng đời hỗ trợ từ cộng đồng, Amazon RDS cũng sẽ kết thúc Standard Support và chuyển các cơ sở dữ liệu chưa được nâng cấp sang Amazon RDS Extended Support. Chế độ này vẫn giúp cơ sở dữ liệu tiếp tục được cập nhật các bản vá bảo mật quan trọng, nhưng đồng thời sẽ phát sinh thêm chi phí theo thời gian sử dụng nếu người dùng chưa nâng cấp lên phiên bản mới [1][2][7].

Thời điểm nhận được email, nhóm đang tập trung hoàn thiện các chức năng cuối cùng của hệ thống nên chỉ đọc lướt qua tiêu đề rồi bỏ qua. Vì cơ sở dữ liệu vẫn hoạt động ổn định, mọi người đều nghĩ rằng đây chỉ là một email mang tính chất nhắc nhở và có thể xử lý sau. Chính sự chủ quan này đã dẫn đến việc khi hết thời hạn hỗ trợ, AWS tự động đưa phiên bản MySQL 8.0 của nhóm sang Extended Support. Hệ thống vẫn vận hành bình thường nên không ai phát hiện ra rằng tài khoản đã bắt đầu phát sinh thêm chi phí mỗi giờ.

## Phát hiện khoản phí bất ngờ

Chỉ đến khi kiểm tra bảng Billing để đối chiếu chi phí triển khai, nhóm mới nhận thấy xuất hiện một khoản phí mới liên quan đến Amazon RDS Extended Support. Sau khi kiểm tra AWS Health Dashboard và đọc lại email thông báo trước đó, mọi người mới hiểu rằng khoản phí này xuất phát từ việc chưa nâng cấp cơ sở dữ liệu lên phiên bản MySQL 8.4 đúng thời điểm.

Ngay sau đó, nhóm quyết định thực hiện nâng cấp phiên bản theo hướng dẫn của AWS. Tuy nhiên, trong quá trình cấu hình nâng cấp lại xảy ra thêm một sai sót khác. Thay vì chọn thực hiện nâng cấp ngay lập tức (**Upgrade immediately**), nhóm giữ nguyên tùy chọn mặc định là áp dụng trong kỳ bảo trì tiếp theo (**Apply during the next maintenance window**).

Điều này đồng nghĩa với việc yêu cầu nâng cấp chỉ được lên lịch, còn cơ sở dữ liệu vẫn tiếp tục chạy trên MySQL 8.0 trong chế độ Extended Support cho đến khi cửa sổ bảo trì diễn ra. Trong khoảng thời gian chờ đợi đó, hệ thống vẫn tiếp tục bị tính phí Extended Support như bình thường [6].

Khi quá trình nâng cấp hoàn tất và cơ sở dữ liệu chuyển sang MySQL 8.4, nhóm kiểm tra lại hóa đơn thì tổng khoản phí phát sinh do Extended Support đã vào khoảng **30 USD**.

Mặc dù đây không phải là một con số quá lớn đối với các hệ thống doanh nghiệp, nhưng đối với một dự án học tập sử dụng ngân sách giới hạn thì đây là một khoản chi phí khá đáng tiếc, nhất là khi nguyên nhân không xuất phát từ lỗi kỹ thuật mà hoàn toàn đến từ việc quản lý và vận hành dịch vụ.

## Bài học về quản lý AWS

Sau sự việc này, nhóm nhận ra rằng việc sử dụng dịch vụ Cloud không chỉ dừng lại ở việc triển khai đúng kiến trúc hay viết mã nguồn ổn định. Người quản trị còn cần thường xuyên theo dõi các thông báo trong AWS Health Dashboard, đọc kỹ các email từ AWS và chủ động kiểm tra Billing Dashboard để phát hiện sớm những khoản phí bất thường.

Những thông báo về vòng đời dịch vụ (**Lifecycle Events**) tuy có vẻ chỉ là thông tin tham khảo nhưng thực tế lại có thể ảnh hưởng trực tiếp đến chi phí vận hành của hệ thống.

Đồng thời, khi thực hiện các thao tác nâng cấp dịch vụ, cần đọc kỹ từng tùy chọn mà AWS cung cấp. Trong trường hợp cần chấm dứt việc phát sinh phí Extended Support càng sớm càng tốt, việc lựa chọn nâng cấp ngay lập tức sẽ phù hợp hơn so với việc chờ đến kỳ bảo trì tiếp theo, miễn là có thể chấp nhận khoảng thời gian gián đoạn dịch vụ.

AWS cũng khuyến nghị tạo một bản snapshot trước khi nâng cấp để có thể khôi phục khi cần thiết, hoặc sử dụng Blue/Green Deployment nhằm giảm thiểu thời gian gián đoạn đối với các hệ thống đang phục vụ người dùng [4][5].

## Nhìn lại

Đây là một bài học thực tế giúp nhóm hiểu rằng trên môi trường Cloud, đôi khi một email bị bỏ qua hoặc một tùy chọn mặc định không được chú ý cũng có thể dẫn đến những khoản chi phí phát sinh ngoài dự kiến.

Chỉ với hai sai sót nhỏ là không đọc thông báo từ AWS và không thực hiện nâng cấp ngay lập tức, nhóm đã phải trả thêm khoảng **30 USD**.

Từ đó, nhóm xây dựng thói quen thường xuyên kiểm tra AWS Health Dashboard, theo dõi Billing Dashboard và xử lý các cảnh báo ngay khi nhận được để tránh lặp lại những tình huống tương tự trong các dự án sau.

Các điểm chính cần nắm:

* Luôn đọc và xử lý các cảnh báo từ AWS.
* Theo dõi thường xuyên AWS Health Dashboard và Billing Dashboard.
* Chủ động nâng cấp dịch vụ trước khi hết thời gian hỗ trợ.
* Kiểm tra kỹ các tùy chọn khi thực hiện nâng cấp.
* Hiểu rằng các cấu hình mặc định có thể gây phát sinh chi phí.
