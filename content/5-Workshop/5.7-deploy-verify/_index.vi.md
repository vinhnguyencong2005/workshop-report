---
title : "Triển khai & Kiểm tra"
date: 2026-07-30
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

Sau khi tất cả các tệp cấu hình Terraform (`provider.tf`, `vpc.tf`, `security_groups.tf`, `iam.tf`, `s3.tf`, `waf.tf`, `amplify.tf`, `apigateway.tf`, `rds.tf`, `redis.tf`, `dynamodb.tf`, `ec2.tf`, `alb.tf`, `cloudwatch.tf`, `outputs.tf`) đã sẵn sàng, tiến hành khởi tạo toàn bộ hạ tầng và kiểm tra hoạt động của các dịch vụ.

---

#### Bước 1: Khởi tạo Terraform & Xem trước Kế hoạch

1. Truy cập vào thư mục repository Terraform:
   ```bash
   cd TTNT-IaC
   ```

2. Khởi tạo các provider plugins và modules:
   ```bash
   terraform init
   ```

![terraform init](/images/workshop/5.7/1.png)

3. Kiểm tra và xem trước danh sách tài nguyên sẽ được tạo:
   ```bash
   terraform plan
   ```

![terraform plan](/images/workshop/5.7/2.png)

{{% notice info %}}
Lệnh `terraform plan` so sánh mã HCL của bạn với trạng thái thực tế trên cloud và hiển thị toàn bộ tài nguyên chuẩn bị khởi tạo (hơn 50+ tài nguyên AWS).
{{% /notice %}}

---

#### Bước 2: Triển khai Hạ tầng (`terraform apply`)

Chạy lệnh apply và nhập `yes` khi được hỏi xác nhận:

```bash
terraform apply
```

Sau khi hoàn tất thành công, Terraform sẽ in ra tất cả các URL endpoint và thông tin tài nguyên:

![terraform apply outputs](/images/workshop/5.7/3.png)

{{% notice tip %}}
**Lưu lại các thông tin Output này!**
Hãy sao chép và lưu lại các giá trị Output ở cuối lệnh `terraform apply` (đặc biệt là `backend_api_url`, `frontend_amplify_url`, và `amplify_app_id`). Bạn sẽ cần các endpoint này để kiểm tra và cấu hình GitHub Actions CI/CD ở các bước tiếp theo.
{{% /notice %}}

##### Thời gian triển khai dự kiến:
- **0–2 phút**: VPC, subnets, IGW, route tables, security groups, IAM roles, WAF v2 Web ACL, API Gateway HTTP API
- **2–5 phút**: NAT Gateways, VPC Endpoints, S3 Buckets, DynamoDB Tables, AWS Amplify App
- **5–15 phút**: RDS MySQL Multi-AZ Instance & ElastiCache Redis Replication Group
- **15–20 phút**: EC2 Launch Template & Auto Scaling Group khởi động máy chủ

Tổng thời gian triển khai khoảng **15–20 phút**.

---

#### Bước 3: Kiểm tra Health Check của Backend API

Lấy endpoint HTTPS API Gateway từ output của Terraform và chạy kiểm tra:

```bash
# Lấy URL backend API
API_URL=$(terraform output -raw backend_api_url)
echo $API_URL

# Kiểm tra health check
curl -s $API_URL/health
```

Kết quả trả về dự kiến:
```json
{"status":"ok"}
```

![backend health](/images/workshop/5.7/4.png)

---

#### Bước 4: Khởi tạo Dữ liệu Mẫu (Tùy chọn)

Nạp dữ liệu mẫu cho khóa học, người dùng và diễn đàn vào RDS MySQL và DynamoDB để kiểm thử:

```bash
cd ../TTNT-backend
chmod +x src/config/seed_sample_data.sh
./src/config/seed_sample_data.sh AWS
```

![seed data](/images/workshop/5.7/5.png)

---

#### Tóm tắt kết quả

- Toàn bộ hạ tầng được tạo tự động 100% bằng **Terraform Infrastructure as Code**.
- **AWS WAF v2** bảo vệ ALB, trong khi **AWS API Gateway** cung cấp endpoint HTTPS quản lý.
- **AWS Amplify** phục vụ frontend an toàn qua HTTPS không bị lỗi Mixed Content.
- Tất cả tài nguyên cloud đã sẵn sàng cho bước tự động hóa quy trình CI/CD tiếp theo.
