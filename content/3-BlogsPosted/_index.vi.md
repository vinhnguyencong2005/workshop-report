---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
includeInReport: false
---

###  [Blog 1 - "CẶP ĐÔI SONG TRÙNG" - AWS GLUE & AMAZON ATHENA: KHI ETL GẶP QUERY ENGINE TRÊN DATA LAKE.](3.1-Blog1/)
Blog này giới thiệu sự kết hợp giữa AWS Glue và Amazon Athena - "cặp đôi song trùng" giúp xây dựng pipeline xử lý và phân tích dữ liệu serverless trên Amazon S3. Bằng cách kết hợp khả năng tự động crawl/transform dữ liệu của Glue với engine truy vấn SQL trực tiếp của Athena thông qua Glue Data Catalog chung, bài viết mang đến giải pháp tối ưu chi phí và hiệu năng cho bài toán Data Lake mà không cần quản lý bất kỳ hạ tầng nào.

### [Blog 2 - AWS ESTIMATED BILLING HIỂN THỊ HÓA ĐƠN HÀNG NGHÌN TỶ USD – ĐIỀU GÌ THỰC SỰ ĐÃ XẢY RA?](3.2-Blog2/)
Bài viết phân tích sự cố Estimated Billing của AWS khi hiển thị hóa đơn tính toán ước tính lên tới hàng tỷ USD. Qua đó giải thích cơ chế hoạt động độc lập giữa hệ thống Estimate và Invoice, khái niệm Unit Pricing, bài học về việc rollback code không đồng nghĩa rollback dữ liệu, và các nguyên tắc thiết kế hệ thống phân tán giúp giới hạn phạm vi ảnh hưởng (blast radius).

###  [Blog 3 - ON-DEMAND VS PROVISIONED MODE TRONG AMAZON DYNAMODB – NÊN CHỌN MÔ HÌNH NÀO?](3.3-Blog3/)
Bài viết so sánh hai chế độ tính phí của Amazon DynamoDB (Provisioned Mode và On-Demand Mode), giải thích các cơ chế vận hành ngầm như Burst Capacity hay Partition Scaling, đồng thời đưa ra lời khuyên cụ thể giúp lựa chọn mô hình tối ưu chi phí và hiệu năng cho từng bài toán thực tế.

###  [Blog 4 - 5 ĐIỀU MÌNH HỌC ĐƯỢC KHI TÌM HIỂU VỀ AMAZON S3](3.4-Blog4/)
Bài viết tổng hợp 5 bài học cốt lõi khi đào sâu vào kiến trúc lưu trữ Amazon S3: Flat Namespace không có thư mục vật lý, cơ chế tính phí ẩn khi xóa file (Multipart Uploads & Versioning), bài toán tối ưu chi phí Request/Băng thông với CloudFront, mô hình Upload tối ưu qua Pre-signed URLs, và những lưu ý thực tế khi áp dụng S3 Storage Classes & Intelligent-Tiering.