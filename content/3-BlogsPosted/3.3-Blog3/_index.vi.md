---
title: "Blog 3"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
includeInReport: false
---
# On-Demand vs Provisioned Mode trong Amazon DynamoDB – Nên chọn mô hình nào?

Chào mọi người, trong quá trình tìm hiểu về các dịch vụ của AWS, mình thấy rất hứng thú với DynamoDB và việc lựa chọn giữa hai phương thức tính phí là **On-Demand** và **Provisioned Mode**.

Một trong những quyết định quan trọng nhất khi thiết kế cơ sở dữ liệu NoSQL với Amazon DynamoDB là lựa chọn **Read/Write Capacity Mode** (Chế độ tính phí và quản lý năng lực). Lựa chọn đúng không chỉ giúp hệ thống chạy mượt mà mà còn **tiết kiệm lên đến 70% chi phí**.

Mình xin chia sẻ bài so sánh giữa 2 mô hình: **Provisioned Mode** và **On-Demand Mode**, kèm theo các cơ chế ngầm (Burst Capacity, Partition Scaling) và ví dụ áp dụng thực tế.

---

## 1. Provisioned Capacity Mode (Chế độ đặt trước)

**Cơ chế hoạt động:** Bạn chủ động khai báo trước số lượng **RCU** (Read Capacity Unit) và **WCU** (Write Capacity Unit) mà table của bạn cần phục vụ trong mỗi giây.

### Điểm đặc biệt: Cơ chế "Burst Capacity"

Nhiều người nghĩ rằng nếu đặt trước 100 WCU mà chỉ dùng 20 WCU thì 80 WCU còn lại sẽ bị "mất trắng". Tuy nhiên, DynamoDB có một cơ chế gọi là **Burst Capacity**:

- DynamoDB tự động tích lũy các RCU/WCU chưa sử dụng trong những phút thấp điểm và lưu trữ lại trong một **"quỹ dự trữ"**.
- Quỹ này có thể lưu giữ tối đa tương đương **5 phút (300 giây)** dung lượng capacity đặt trước.
- **Tác dụng:** Khi hệ thống đột ngột có một đợt bùng nổ truy cập ngắn (Burst Traffic) vượt quá ngưỡng Provisioned, DynamoDB sẽ rút WCU/RCU từ "quỹ dự trữ" này ra để phục vụ ngay lập tức mà **KHÔNG bị Throttling** và **KHÔNG tốn thêm chi phí**.

### Ưu điểm

- **Chi phí cực kỳ tối ưu (Cost-Effective):** Nếu tải hệ thống của bạn ổn định hoặc có tính chu kỳ dự đoán được (ví dụ: giờ hành chính cao, đêm thấp), Provisioned mode rẻ hơn nhiều so với On-Demand.
- **Giảm chi phí sâu hơn với Reserved Capacity:** Có thể cam kết mua trước 1 năm / 3 năm để nhận discount lên tới hơn **70%**.

### Nhược điểm

- **Nguy cơ bị Throttling Exception** (`ProvisionedThroughputExceededException`): Nếu lượng truy cập tăng đột biến duy trì quá 5 phút (vượt quá Burst Capacity) và Auto Scaling chưa kịp scale-out (có thể mất từ 1–5 phút để Auto Scaling phản ứng).

---

## 2. On-Demand Capacity Mode (Phục vụ theo yêu cầu)

**Cơ chế hoạt động:** Bạn không cần khai báo RCU/WCU. DynamoDB tự động quản lý tài nguyên và bạn chỉ trả tiền đúng cho số lượng request thực tế phát sinh (tính theo **RRU/WRU** — Request Units).

### Điểm đặc biệt: Hiện tượng Lag / Throttling khi Spike Up đột ngột

Nhiều bạn lầm tưởng On-Demand mode có thể scale từ 0 lên vô tận ngay lập tức trong 1 millisecond. Thực tế không hoàn toàn như vậy:

- **Cơ chế scaling của On-Demand:** DynamoDB On-Demand tự động phục vụ được lượng traffic gấp **2 lần (2x)** mức đỉnh (peak traffic) từng ghi nhận trước đó.
- **Vấn đề khi Traffic Spike Up quá nhanh** (ví dụ: từ 500 req/s vọt lên 50,000 req/s trong 2 giây): Khi lưu lượng tăng vọt quá nhanh vượt xa mức đỉnh cũ, DynamoDB cần thời gian để tự động phân tách Partition (**Partition Splitting**) và allocate thêm máy chủ vật lý ở background.
- Trong vài giây đến vài phút diễn ra quá trình "warm-up" / partition split này, các request vượt ngưỡng có thể bị tăng **Latency** (trễ) hoặc tạm thời bị **Throttling**.

### Ưu điểm

- **Zero-Management:** Không cần cấu hình WCU/RCU, không lo quản lý Auto Scaling rules.
- **Pay-per-request:** Nếu ứng dụng không có ai dùng (ví dụ môi trường Dev/Staging hoặc ứng dụng bán hàng ban đêm không có khách), chi phí bằng **$0**.

### Nhược điểm

- **Chi phí đắt hơn trên mỗi request:** Đơn giá trên mỗi 1 triệu Request Unit ở On-Demand đắt hơn khoảng **70%** so với giá 1 RCU/WCU chạy liên tục ở Provisioned mode với cùng điều kiện.
- **Phải cẩn thận với Spike Traffic cực lớn:** Vẫn cần warm-up trước nếu biết chắc chắn sẽ có sự kiện lớn như săn deal, flash-sale.

---

## 3. Ví dụ thực tế: Khi nào chọn mô hình nào?

### Nên chọn **On-Demand** Mode khi:

- **Mới ra mắt sản phẩm:** Chưa có dữ liệu lịch sử để biết người dùng truy cập ra sao.
- **Ứng dụng có tải nghỉ dài:** Hệ thống nhận phản ánh từ khách hàng rải rác, đêm hầu như không có ai dùng.
- **Môi trường Serverless:** Các ứng dụng xây dựng trên kiến trúc Event-driven với lượng request trồi sụt thất thường.

### Nên chọn **Provisioned** Mode khi:

- **Ứng dụng có lượng truy cập ổn định, dự đoán được.**
- **Đã chạy On-Demand một thời gian và có CloudWatch Metrics:** Sau khi theo dõi bảng CloudWatch 1–2 tháng, bạn xác định được mức baseline traffic → Chuyển sang Provisioned + Auto Scaling để giảm chi phí.
- **Cần tối ưu ngân sách nghiêm ngặt (Cost Optimization):** Kết hợp Provisioned Mode với DynamoDB Reserved Capacity.

---

## Nguồn tài liệu tham khảo

- [Read/Write Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/capacity-mode.html)
- [Provisioned Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/provisioned-capacity-mode.html)
- [Burst & Adaptive Capacity](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/burst-adaptive-capacity.html)
- [On-Demand Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/on-demand-capacity-mode.html)
- [DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)
- [Bài viết trên Facebook](https://www.facebook.com/share/p/18qTVu8i7b/)