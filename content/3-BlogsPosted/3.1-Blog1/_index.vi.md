---
title: 'Blog 1'
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
includeInReport: false
---
# "CẶP ĐÔI SONG TRÙNG" - AWS GLUE & AMAZON ATHENA: KHI ETL GẶP QUERY ENGINE TRÊN DATA LAKE.

> *"Dữ liệu nằm trên S3 mà không query được thì cũng như kho vàng mà không có chìa khóa."*

**Xin chào mọi người,**
Nếu bạn đang xây dựng một Data Lake trên AWS, chắc hẳn bạn đã từng nghe đến hai cái tên: **AWS Glue** và **Amazon Athena**. Tách riêng ra, mỗi service đều mạnh mẽ theo cách riêng. Nhưng khi kết hợp lại, chúng tạo thành một "cặp đôi song trùng" - bổ trợ hoàn hảo cho nhau trong bài toán xử lý và phân tích dữ liệu quy mô lớn mà không cần quản lý bất kỳ server nào.

---

## AWS Glue - "Người dọn dẹp và sắp xếp dữ liệu"

**AWS Glue** là dịch vụ ETL (Extract - Transform - Load) serverless được quản lý hoàn toàn bởi AWS. Nói đơn giản, Glue giúp bạn khám phá cấu trúc dữ liệu, chuyển đổi dữ liệu thô sang dạng sạch và tối ưu, rồi tải vào nơi lưu trữ đích.

![AWS Glue là gì?](/images/3-BlogsPosted/3.1-Blog1/aws-glue-la-gi.jpg)

## Amazon Athena - "Người truy vấn không cần server"

**Amazon Athena** là dịch vụ interactive query serverless cho phép bạn phân tích dữ liệu trực tiếp trên Amazon S3 bằng **SQL chuẩn**, dựa trên engine **Trino** (trước đây gọi là Presto).

![Kiến trúc Amazon Athena](/images/3-BlogsPosted/3.1-Blog1/athena.png)

## Tại sao là "Cặp đôi song trùng"?

Hãy tưởng tượng: **AWS Glue** là người thủ thư - phân loại, gắn nhãn, sắp xếp sách lên kệ. **Amazon Athena** là người đọc - đến thư viện, mở catalog, tìm đúng cuốn sách cần đọc.

Mối liên kết cốt lõi chính là **Glue Data Catalog**: Glue tạo ra nó, Athena đọc từ nó.

### Luồng dữ liệu end-to-end

1. **Ingest**: Dữ liệu thô từ nhiều nguồn (app logs, databases, IoT) được đưa vào **Raw Zone** trên S3.
2. **Crawl**: Glue Crawler quét Raw Zone, tạo bảng trong Data Catalog.
3. **Transform**: Glue ETL Job đọc dữ liệu thô, làm sạch, chuyển sang Parquet có partition → ghi vào **Curated Zone**.
4. **Crawl lại**: Crawler quét Curated Zone, cập nhật Data Catalog.
5. **Query**: Athena đọc metadata từ Data Catalog, query trực tiếp dữ liệu trên S3 bằng SQL.

> **Tóm lại:** Glue lo "hậu trường" (crawl, catalog, transform), Athena lo "sân khấu" (query, phân tích). Cả hai đều serverless, cả hai đều pay-per-use, và cả hai đều chia sẻ chung một Data Catalog.

---

## Best Practices

### Tối ưu chi phí Athena

1. **Dùng Parquet / ORC** thay vì CSV / JSON - format columnar giúp giảm **30–90%** dữ liệu scan.
2. **Partition dữ liệu** theo thời gian hoặc key phổ biến - Athena chỉ scan partition liên quan.
3. **Dùng compression** (Snappy, GZIP, ZSTD) - giảm kích thước file trên S3.
4. **SELECT cột cụ thể**, tránh `SELECT *` - columnar format chỉ đọc cột cần thiết.
5. **Dùng CTAS** (Create Table As Select) - lưu kết quả dưới dạng bảng mới, tái sử dụng.

### Tối ưu Glue ETL

1. **Chọn đúng Worker Type**: `G.1X` cho workload nhẹ, `G.2X` cho heavy transform.
2. **Enable Job Bookmarks** - tránh xử lý lại dữ liệu đã transform.
3. **Compact small files** - nhiều file nhỏ = chậm. Gộp lại thành file **128MB–512MB**.
4. **Enable Auto Scaling** - Glue 3.0+ hỗ trợ auto scaling worker, tiết kiệm chi phí.

---

## So sánh với các giải pháp khác

| Tiêu chí | Glue + Athena | Amazon Redshift | Amazon EMR |
|---|---|---|---|
| **Serverless** | ✅ Hoàn toàn | ❌ Cần quản lý cluster | ❌ Cần quản lý cluster |
| **Chi phí khởi đầu** | Gần $0 | Cao | Cao |
| **Phù hợp** | Ad-hoc query, Data Lake | Data Warehouse, BI nặng | ML, Spark workload lớn |
| **Latency** | Vài giây | Mili-giây | Phút |
| **Độ phức tạp** | Thấp | Trung bình | Cao |

> **Khi nào chọn Glue + Athena?** Khi bạn mới bắt đầu xây Data Lake, workload chủ yếu là ad-hoc analysis và batch ETL, muốn pay-per-use, team size nhỏ, và không muốn quản lý infrastructure.

---

## Use Cases thực tế

1. **Log Analytics**: Glue crawl & transform log files sang Parquet, Athena query tìm error patterns và tạo dashboard.
2. **E-commerce Data Lake**: Glue ETL dữ liệu đơn hàng từ RDS → S3, Athena phân tích revenue, top products, customer segmentation.
3. **IoT Data Processing**: Glue Streaming ETL đọc dữ liệu sensor từ Kinesis, Athena phân tích historical data và phát hiện anomaly.
4. **Data Mesh / Data Sharing**: Glue Data Catalog + AWS Lake Formation quản lý quyền truy cập cross-account, Athena cho mỗi team query trong phạm vi được cấp.

---

## Bảo mật

Cả hai service đều hỗ trợ:
- **Encryption at rest**: S3 SSE, KMS
- **Encryption in transit**: TLS/SSL
- **Access control**: IAM Policies + AWS Lake Formation (chi tiết đến mức column/row)
- **Audit logging**: AWS CloudTrail
- **Network isolation**: VPC Endpoints (PrivateLink)

---

## Kết luận

AWS Glue và Amazon Athena thực sự xứng đáng là "cặp đôi song trùng" trong hệ sinh thái Data Analytics trên AWS. Khi kết hợp, bạn có một Data Lake pipeline hoàn chỉnh, serverless, pay-per-use mà không cần quản lý bất kỳ server nào. Đây là lựa chọn lý tưởng cho bất kỳ tổ chức nào muốn bắt đầu nhanh với chi phí thấp và scale lên khi cần.

> *"Đừng xây Data Warehouse khi bạn chỉ cần Data Lake. Và khi bạn cần Data Lake - hãy bắt đầu với Glue + Athena."*

---

## Nguồn tham khảo

- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html)
- [Amazon Athena Documentation](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
- [Bài viết trên Facebook](https://www.facebook.com/share/p/14iYXw6trwj/)