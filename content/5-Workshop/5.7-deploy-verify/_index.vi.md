---
title : "Triển khai & Kiểm tra"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

Sau khi tất cả các file cấu hình Terraform (`provider.tf`, `vpc.tf`, `security_groups.tf`, `iam.tf`, `s3.tf`, `waf.tf`, `amplify.tf`, `apigateway.tf`, `rds.tf`, `redis.tf`, `dynamodb.tf`, `ec2.tf`, `alb.tf`, `cloudwatch.tf`, `outputs.tf`) đã sẵn sàng, tiến hành khởi tạo toàn bộ hạ tầng và kiểm tra hoạt động của các dịch vụ.

---

#### Bước 1: Khởi tạo Terraform & Xem trước kế hoạch

1. Truy cập vào thư mục repository Terraform:
   ```bash
   cd Terraform_
   ```

2. Khởi tạo các provider plugins và modules:
   ```bash
   terraform init
   ```

3. Kiểm tra danh sách tài nguyên sẽ được tạo:
   ```bash
   terraform plan
   ```

![terraform plan](../../../images/workshop/5.7/1.png)

{{% notice info %}}
Lệnh `terraform plan` so sánh code HCL của bạn với trạng thái thực tế trên cloud và hiển thị toàn bộ tài nguyên chuẩn bị khởi tạo (hơn 50+ tài nguyên AWS).
{{% /notice %}}

---

#### Bước 2: Triển khai Hạ tầng (`terraform apply`)

Chạy lệnh apply và gõ `yes` khi được hỏi xác nhận:

```bash
terraform apply
```

Sau khi hoàn tất thành công, Terraform sẽ in ra tất cả các URL endpoint và thông tin tài nguyên:

![terraform apply outputs](../../../images/workshop/5.7/2.png)

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

![backend health](../../../images/workshop/5.7/3.png)

---

#### Bước 4: Kiểm tra Ứng dụng Frontend trên AWS Amplify

Lấy URL tên miền AWS Amplify:

```bash
# Lấy URL frontend Amplify
AMPLIFY_URL=$(terraform output -raw frontend_amplify_url)
echo $AMPLIFY_URL
```

Mở URL trên trình duyệt web. Bạn sẽ thấy màn hình đăng nhập của hệ thống TTNT LMS chạy trên HTTPS mà không gặp bất kỳ lỗi Mixed Content nào.

![frontend login](../../../images/workshop/5.7/4.png)

---

#### Bước 5: Kết nối vào EC2 qua AWS SSM Session Manager

Kiểm tra trạng thái máy chủ EC2 và các tiến trình Node.js / PM2 mà không cần mở port SSH 22:

1. Lấy Instance ID của máy chủ đang chạy trong Auto Scaling Group:
   ```bash
   INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=app-server-asg" "Name=instance-state-name,Values=running" \
     --query "Reservations[0].Instances[0].InstanceId" --output text)
   echo $INSTANCE_ID
   ```

2. Bắt đầu phiên SSM Session Manager:
   ```bash
   aws ssm start-session --target $INSTANCE_ID
   ```

3. Trong terminal EC2, kiểm tra phiên bản Node.js và ứng dụng PM2:
   ```bash
   node --version
   pm2 list
   cat /var/log/user_data.log
   ```

![ssm session](../../../images/workshop/5.7/5.png)

---

#### Bước 6: Khởi tạo Dữ liệu Mẫu (Tùy chọn)

Nạp dữ liệu mẫu cho khóa học, người dùng và diễn đàn vào RDS MySQL và DynamoDB:

```bash
cd ../TTNT-backend
node scripts/seed_all_data.sh
```

Kiểm tra endpoint API trả về dữ liệu JSON mẫu:
```bash
curl -s $API_URL/api/classes | jq
```

![seed data](../../../images/workshop/5.7/6.png)

---

#### Tóm tắt kết quả

- Toàn bộ hạ tầng được tạo tự động 100% bằng **Terraform Infrastructure as Code**.
- **AWS WAF v2** bảo vệ ALB, trong khi **AWS API Gateway** cung cấp endpoint HTTPS quản lý.
- **AWS Amplify** phục vụ frontend an toàn qua HTTPS không bị lỗi Mixed Content.
- **AWS SSM Session Manager** giúp truy cập máy chủ an toàn mà không cần mở port SSH 22 ra internet.
