---
title: "Sự kiện 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---
# Bài thu hoạch "Agentic AI Build Week & One Team Community Day: Hành trình sáng tạo và làm chủ giải pháp AI trên AWS"

### Mục đích của sự kiện

- **Tiếp cận bài toán thực tế của AI Agent:** Tiếp thu phương pháp phát triển tác nhân AI tự động (Agentic AI) để giải quyết các thách thức vận hành trong doanh nghiệp và đời sống.
- **Học hỏi tư duy thiết kế kiến trúc Đa kênh (Multi-channel):** Tìm hiểu mô hình tích hợp trực tiếp AI Agent vào các ứng dụng nhắn tin quen thuộc như Zalo, Messenger mà không bắt người dùng cài app mới.
- **Trải nghiệm thực chiến 24h Hackathon:** Lắng nghe bài học kinh nghiệm, áp lực tiến độ và câu chuyện thực tế từ các đội thi tại Agentic AI Build Week (AABW).
- **Ứng dụng AI vào quy trình kỹ thuật chuyên sâu:** Khám phá giải pháp AI Native hỗ trợ Kiến trúc sư giải pháp (Solutions Architect) tự động hóa bóc tách tài liệu, vẽ sơ đồ hạ tầng và lập dự toán chi phí AWS.
- **Phân tích dữ liệu & Tín hiệu doanh nghiệp:** Nghiên cứu kiến trúc Multi-Agent phối hợp Web Crawler để theo dõi, phân tích và đưa ra cảnh báo sớm về tái cấu trúc doanh nghiệp.

### Danh sách diễn giả & Các đội ngũ chia sẻ

- **Dự án KFC Bot Agent (Đội One Team):** Anh Duy, Trần Đông, Đoàn Trung, Minh Việt, Anshul Roy.
- **Dự án S.H.E.P.H.E.R.D (Đội 3KA):** Huỳnh An Khương, Nguyễn Quốc Huy, Ngô Quang Khôi, Hoàng Lê Thành Đức, Đặng Nguyễn Phước Lộc.
- **Dự án SA Professional AI Native App (Đội Plan V):** Phạm Tiến Thuận, Phát Huỳnh, Hoàng Long, Lê Minh Nghĩa, Trần Đại Vĩ, Nguyễn An.
- **Dự án Signal Scout (Đội Dream AI Team):** Lê Tấn Lực, Đỗ Hoàng Hiếu, Triệu Quốc Hào, Nguyễn Văn Duy Khiêm, Nguyễn Công Minh, Nguyễn Trần Minh Quân.

---

### Nội dung nổi bật

#### 1. KFC Bot Agent – Trợ lý AI đặt hàng đa kênh (Đội One Team)

- **Bối cảnh & Vấn đề thực tế:** Bài toán đặt đồ ăn qua giọng nói/tin nhắn rất phức tạp. Thực tế từ dự án AI Drive-thru của McDonald's cho thấy: câu từ người dùng mang tính ngẫu nhiên nhưng các quy tắc bán hàng (khuyến mãi, tùy chọn món, trạng thái giỏ hàng) đòi hỏi độ chính xác tuyệt đối. Việc ép khách hàng thoát ứng dụng nhắn tin (Zalo, Messenger) để tải app đặt đồ ăn làm tăng tỷ lệ bỏ dở đơn hàng (Lost Order).
- **Giải pháp & Giá trị cốt lõi:** KFC Bot Agent mang đến trải nghiệm **"Đặt món trực tiếp trong khung chat"**. Khách hàng không cần đăng ký tài khoản mới hay chuyển app, giúp giảm tải công việc cho tổng đài viên.
- **Cơ chế 5 bước hành động của Agent:**
  1. *Mục tiêu (Goal):* Xác định chính xác ý định đặt món của người dùng.
  2. *Kế hoạch (Plan):* Xây dựng trình tự các bước thực hiện.
  3. *Công cụ (Tools):* Tra cứu dữ liệu thực đơn và giá bán đáng tin cậy.
  4. *Hành động (Act):* Thêm món vào giỏ hàng và áp dụng mã giảm giá.
  5. *Xác thực (Verify):* Kiểm tra tính chính xác của giỏ hàng trước khi chốt đơn.
- **Kiến trúc "Design Once | Deploy Everywhere":**
  * *Luồng đa kênh:* Tin nhắn từ Zalo, WhatsApp, Telegram... được Channel Adapters chuẩn hóa thành `Normalized Message`, sau đó chuyển tới **One Agent Platform** để xử lý suy luận, bộ lọc an toàn và bộ nhớ.
  * *Hạ tầng AWS:* Kết hợp WAF, API Gateway, Lambda, SQS, AgentCore Gateway, Amazon Bedrock AgentCore, DynamoDB, OpenSearch, ElastiCache, S3 cùng các cổng thanh toán và giao hàng.
- **Bốn con số hiệu suất ấn tượng:**
  * **$0.006 / đơn hàng:** Chi phí vận hành tối ưu (tính trên 500 đơn/ngày).
  * **$88 / tháng:** Tổng chi phí duy trì hạ tầng (Amazon Bedrock chiếm 75%).
  * **3 - 5 giây:** Thời gian phản hồi tin nhắn hoàn chỉnh.
  * **Giảm 60% Infra Code:** Nhờ ứng dụng nền tảng AgentCore thay thế hạ tầng truyền thống.
- **Thông điệp dự án:** *"Khách hàng KFC Việt Nam đã có mặt sẵn trên Zalo. KFC AI giúp họ không cần phải chuyển sang nền tảng nào khác."*

---

#### 2. Dự án S.H.E.P.H.E.RD – Giám sát & Điều phối An ninh/Lưu lượng Thông minh (Đội 3KA)

- **Hành trình 24h Hackathon:** Kỷ niệm thực chiến của đội qua các giai đoạn từ ngợp áp lực ban đầu đến tập trung cao độ và tự hào. Những trải nghiệm đáng nhớ lúc 3h sáng: tiêu thụ 5 lon Redbull, sự cố đẩy nhầm file `.env` chứa secret key lên GitHub và cùng nhau tranh luận để chốt sản phẩm.
- **Bài toán & Giải pháp:** Việc giám sát thủ công tại các lối vào sự kiện dễ gây bỏ sót điểm nghẽn ùn tắc. S.H.E.P.H.E.R.D kết hợp camera giám sát với mô hình AI để phân tích mật độ đám đông, dự báo thời gian xếp hàng và phát cảnh báo sớm.
- **2 tính năng AI Agentic cốt lõi:**
  * *Autonomous Monitor:* Giám sát tự động liên tục 24/7.
  * *Operator Copilot:* Trợ lý tương tác bằng ngôn ngữ tự nhiên giúp nhân viên an ninh nhận đề xuất hướng xử lý sự cố.
- **Công nghệ & Hạ tầng AWS:** Sử dụng YOLO + ByteTrack, Amazon SageMaker, Amazon Bedrock AgentCore + Strands Agent và React Dashboard. Hạ tầng gồm Kinesis Video Streams, ECS Fargate, Lambda, API Gateway và DynamoDB.

---

#### 3. SA Professional AI Native App – Trợ lý tự động hóa cho Kiến trúc sư Giải pháp (Đội Plan V)

- **Vấn đề của Solution Architect (SA):** Kiến trúc sư giải pháp mất nhiều thời gian bóc tách yêu cầu, vẽ sơ đồ hạ tầng và tính toán chi phí AWS thủ công mỗi khi nhận đề bài từ khách hàng.
- **Giải pháp & Tính năng:** Ứng dụng AI Native phân tích yêu cầu dạng văn bản hoặc giọng nói → Tự động tạo phương án kiến trúc → Xuất sơ đồ AWS / Draw.io có thể chỉnh sửa → Ước tính chi phí AWS (region `ap-southeast-1`) → Tự động phát hiện điểm thiếu sót trong yêu cầu.
- **Kiến trúc hệ thống:** Giao diện Chat → App Server → Knowledge Base (bảng giá AWS), Amazon Bedrock, Draw.io MCP, AWS Pricing MCP. Triển khai bằng Terraform trên AWS (CloudFront, Cognito, ALB, ECS Fargate, PostgreSQL, S3, EFS).
- **Tác động đột phá:** Giúp SA hoàn thiện bản thảo kiến trúc, mã IaC và bảng dự toán chi phí AWS trong vài phút thay vì phải xây dựng thủ công từ đầu.

---

#### 4. Signal Scout – Nền tảng Phân tích & Cảnh báo Tín hiệu Tái cấu trúc Doanh nghiệp (Đội Dream AI Team)

- **Mô hình Value Creation:** Xâu chuỗi các nguồn tin tức, dữ liệu tài chính và biến động nhân sự thành báo cáo phân tích tổng quan có bằng chứng, hỗ trợ lãnh đạo đưa ra quyết định chiến lược.
- **Kiến trúc AgentCore & AWS:** Xây dựng trên Route53, Amplify, Cognito, API Gateway, Lambda, DynamoDB. Hệ thống gồm *Crawler Subagent* (Apify, TinyFish) và *Analysis Subagent* (Bedrock Guardrails, Strands Agent, Langfuse).
- **Trải nghiệm demo:** Giao diện web cho phép nhập URL hoặc tên công ty để hệ thống tự động thu thập dữ liệu, phân tích biến động tài chính và hiển thị biểu đồ xu hướng theo thời gian.

---

### Những gì học được

- **Agentic AI Architecture:** Một tác nhân AI hoàn chỉnh không chỉ dừng lại ở tính năng hỏi đáp mà cần tuân thủ chu trình 5 bước: Goal → Plan → Tools → Act → Verify.
- **Triết lý "Design Once | Deploy Everywhere":** Chuẩn hóa kiến trúc thông qua các lớp Adapter giúp dễ dàng mở rộng ứng dụng sang nhiều kênh nhắn tin mà không cần viết lại mã nguồn.
- **Tối ưu hóa chi phí Cloud với Bedrock & AgentCore:** Tận dụng AgentCore giúp giảm 60% Infra code và duy trì chi phí vận hành ở mức rất thấp trên mỗi giao dịch.
- **Bài học thực chiến Hackathon:** tinh thần *"Sản phẩm nhỏ vận hành ổn định giá trị hơn ý tưởng lớn dở dang"* và tầm quan trọng của việc phối hợp đồng đội dưới áp lực thời gian.

---

### Ứng dụng vào công việc

- **Áp dụng mô hình Đa tác nhân (Multi-Agent):** Phân chia công việc cho các Agent nhỏ chuyên biệt (Thu thập dữ liệu, Phân tích, Kiểm duyệt) kết hợp với Agent điều phối để duy trì độ chính xác và tránh nhiễu ngữ cảnh.
- **Ứng dụng MCP (Model Context Protocol):** Tích hợp giao thức MCP (như AWS Pricing MCP hay Draw.io MCP) để tác nhân AI trực tiếp thao tác và tạo tài liệu, sơ đồ kỹ thuật tự động.
- **Tối ưu hóa quy trình thiết kế Cloud:** Vận dụng công cụ AI Native vào việc phác thảo kiến trúc hệ thống và tính toán ngân sách AWS nhanh chóng cho các dự án thực tế.

### Hình ảnh minh chứng

![Event 4](/images/4-EventParticipated/4.4-Event4/Event4_.png)�u đồ thời gian xu hướng.

---

### Những gì học được

- **Agentic AI Architecture:** Một Agent hoàn chỉnh không chỉ dừng lại ở Chatbot mà phải có đủ 5 bước: Goal → Plan → Tools → Act → Verify.
- **Triết lý "Design Once | Deploy Everywhere":** Việc thiết kế kiến trúc chuẩn hóa qua các lớp Adapter/Connector giúp dễ dàng mở rộng sản phẩm lên bất kỳ kênh giao tiếp nào mà không cần viết lại mã nguồn.
- **Tối ưu hóa chi phí Cloud với Bedrock & AgentCore:** Việc ứng dụng AgentCore giúp giảm 60% Infra code và tối ưu chi phí vận hành xuống chỉ vài cent/đơn hàng.
- **Bài học thực chiến Hackathon:** *"Có mặt đã là chiến thắng một nửa"*, *"Sản phẩm nhỏ chạy được còn hơn ý tưởng lớn mà hỏng"*, và *"Những đồng đội bạn gặp quan trọng hơn cả giải thưởng"*.

---

### Ứng dụng vào công việc

- **Áp dụng mô hình Đa tác nhân (Multi-Agent):** Phân chia nhiệm vụ cho các Agent nhỏ chuyên biệt (Crawler, Analysis, Guardrails) kết hợp với Agent chính để tránh loãng ngữ cảnh và tăng độ chính xác.
- **Ứng dụng MCP (Model Context Protocol):** Tích hợp MCP (như AWS Pricing MCP hay Draw.io MCP) để cho phép AI Agent trực tiếp tương tác và sinh ra tài liệu/sơ đồ kỹ thuật tự động.
- **Tối ưu hóa quy trình thiết kế Cloud:** Áp dụng tư duy AI Native vào việc phác thảo kiến trúc hệ thống và dự toán ngân sách AWS nhanh chóng cho các dự án thực tế.

### Hình ảnh minh chứng

![Event 4](/images/4-EventParticipated/4.4-Event4/Event4_.png)