---
title: "Blog 4"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---

# 5 Điều mình học được khi tìm hiểu về Amazon S3

Chào mọi người,

Gần đây mình có dành thời gian đọc tài liệu và tìm hiểu sâu hơn về kiến trúc lưu trữ trên AWS, đặc biệt là dịch vụ S3. Ban đầu, mình chỉ xem đây là một kho lưu trữ file đơn thuần, nhưng khi đi sâu vào tài liệu kỹ thuật, mình nhận ra có rất nhiều cơ chế ngầm mà nếu bỏ qua sẽ dễ dẫn đến sai lầm trong thiết kế và quản lý chi phí.

Dưới đây là 5 điều cốt lõi nhất mình đã đúc kết được sau khi tìm hiểu về S3, hi vọng sẽ hữu ích cho các bạn đang học và làm đồ án liên quan đến Cloud:

### 1. S3 không hề có hệ thống thư mục vật lý

Khi nhìn vào giao diện của S3, mình thấy các folder rất trực quan, nhưng thực chất S3 là một hệ thống Object Storage với kiến trúc **Flat Namespace**.

Điều mình học được là một đường dẫn như `images/2026/avatar.png` không phải là tệp `avatar.png` nằm trong hai cấp thư mục. Toàn bộ chuỗi đó là một khóa (Key) duy nhất. Các dấu `/` chỉ là tiền tố (Prefix) để giả lập giao diện thư mục. Hiểu được bản chất này giúp mình nhận ra rằng thao tác "đổi tên thư mục" trên S3 thực chất là hệ thống phải sao chép toàn bộ object sang một chuỗi key mới và xóa object cũ — một thao tác cực kỳ tốn kém tài nguyên nếu không quy hoạch tiền tố chuẩn xác từ đầu.

### 2. Nhấn Delete không đồng nghĩa với việc AWS ngừng tính phí

Một điểm khiến mình khá bất ngờ khi đọc tài liệu về quản lý chi phí (Billing) là thao tác xóa file đôi khi không thực sự ngắt chi phí lưu trữ:

* **Incomplete Multipart Uploads:** Khi hệ thống cho phép upload các file lớn, quá trình này được chia nhỏ. Nếu mạng gián đoạn, các phần đã tải lên sẽ kẹt lại và bị ẩn đi, nhưng AWS vẫn tính tiền. Cách giải quyết chuẩn là phải thiết lập Lifecycle Rule để dọn dẹp.
* **Versioning:** Nếu bucket đang bật tính năng Versioning, thao tác xóa chỉ gán một "Delete Marker" lên file chứ không xóa dữ liệu gốc. Bài học rút ra là luôn phải đi kèm Lifecycle Rule để dọn dẹp các phiên bản cũ sau một khoảng thời gian nhất định.

### 3. Phí lưu trữ rất rẻ, nhưng Request và Băng thông mới là bài toán khó

Khi lập bảng dự toán chi phí, mình thấy dung lượng lưu trữ của S3 Standard cực kỳ rẻ. Tuy nhiên, chỗ dễ gây thâm hụt ngân sách nhất lại nằm ở chỗ khác:

* **Request Cost:** S3 tính phí trên mỗi 1.000 API calls. Nếu hệ thống lưu trữ hàng triệu file tĩnh cực nhỏ và liên tục truy vấn, chi phí gọi API sẽ vượt xa phí lưu trữ.
* **Data Transfer Out:** Băng thông đẩy dữ liệu từ S3 ra Internet có giá khá cao.

Do đó, nguyên tắc kiến trúc rút ra là phải luôn đặt một mạng phân phối nội dung như **Amazon CloudFront** đứng trước S3 để cache file tĩnh, giúp chặn đứng lượng lớn request trực tiếp vào S3.

### 4. Kiến trúc Upload tối ưu: Không đẩy file qua Backend

Đây là phần mình thấy hay nhất về System Design. Ban đầu, tư duy logic của mình là: `Client` gửi file lên `Backend` → `Backend` dùng SDK đẩy lên `S3`.

Nhưng khi đọc về cách tối ưu, mình học được cách làm đó sẽ khiến Backend bị nghẽn nếu có quá nhiều người cùng upload. Thay vào đó, kiến trúc hiện đại chuộng dùng **Pre-signed URLs**. Backend chỉ đóng vai trò kiểm tra quyền và tạo ra một URL sống trong thời gian ngắn. Client sẽ dùng URL đó để tự đẩy luồng dữ liệu nặng thẳng lên máy chủ S3.

### 5. S3 có nhiều loại lưu trữ, đừng chỉ dùng Standard

Ban đầu mình nghĩ cứ upload file lên S3 là tất cả đều được lưu ở S3 Standard. Sau khi đọc tài liệu AWS mới biết S3 có nhiều Storage Class dành cho các nhu cầu khác nhau. Với dữ liệu ít được truy cập, có thể dùng Standard-IA hoặc các lớp Glacier để giảm đáng kể chi phí lưu trữ.

Điều mình thấy hay nhất là **S3 Intelligent-Tiering**. Chỉ cần chọn Storage Class này khi lưu object, S3 sẽ tự theo dõi tần suất truy cập và tự động chuyển object giữa các access tier để tối ưu chi phí mà không tính Retrieval Fee giữa các tier của Intelligent-Tiering. Tuy nhiên, bài học thực tế rút ra là đừng bật tính năng này một cách mù quáng:

* **File quá nhỏ:** Với những file dung lượng rất nhỏ, AWS vẫn thu thêm phí quản lý tự động cho từng file. Nếu hệ thống có hàng triệu file tí hon, chi phí phát sinh này thậm chí còn đắt hơn giữ ở S3 Standard.
* **Xóa file sớm:** Các lớp giá rẻ thường đi kèm cam kết thời gian lưu trữ tối thiểu. Nếu bạn ném file vào rồi xóa đi ngay sau đó, AWS vẫn sẽ tính tiền cho trọn cả chu kỳ.

---

Trên đây là những gì mình thấy hay sau khi đọc lý thuyết liên quan đến Amazon S3. Mọi người ở đây ai đã có kinh nghiệm "thực chiến", từng gặp lỗi hay có tips nào hay khi tối ưu S3 thì chia sẻ thêm cho mình và các bạn khác cùng học hỏi với nhé!

Cảm ơn mọi người đã xem!

### Nguồn tham khảo
- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Amazon S3 Pricing](https://aws.amazon.com/vi/s3/pricing)