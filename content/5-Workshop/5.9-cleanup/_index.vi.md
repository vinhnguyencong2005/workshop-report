---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

Để tránh phát sinh chi phí ngoài ý muốn trên tài khoản AWS sau khi hoàn tất buổi workshop, việc dọn dẹp toàn bộ hạ tầng cloud đã tạo là vô cùng quan trọng.

Do mọi tài nguyên đã được khai báo tập trung bằng **Terraform Infrastructure as Code**, bạn có thể xóa toàn bộ hơn 50+ tài nguyên AWS một cách sạch sẽ chỉ bằng một lệnh duy nhất.

---

#### Bước 1: Xóa Hạ tầng (`terraform destroy`)

1. Mở terminal và truy cập vào thư mục Terraform:
   ```bash
   cd Terraform_
   ```

2. Chạy lệnh xóa tài nguyên:
   ```bash
   terraform destroy
   ```

3. Terraform sẽ liệt kê toàn bộ các tài nguyên sắp bị xóa. Gõ **`yes`** khi được hỏi để xác nhận:

![terraform destroy](/images/workshop/5.9/1.png)

##### Thời gian dọn dẹp dự kiến:
- **0–2 phút**: WAF Web ACL, API Gateway HTTP API, ALB listeners, Security Groups, IAM instance profiles
- **2–5 phút**: EC2 Auto Scaling Group, Launch Template, S3 buckets, DynamoDB tables, AWS Amplify App
- **5–12 phút**: RDS MySQL Multi-AZ Instance và ElastiCache Redis Replication Group
- **12–15 phút**: NAT Gateways, Elastic IPs, VPC subnets, Internet Gateway, VPC

Tổng thời gian dọn dẹp kéo dài khoảng **10–15 phút**.

---

#### Bước 2: Xử lý sự cố Dọn dẹp Thủ công (Nếu có)

Trong một số trường hợp hiếm hoi tiến trình xóa bị tắc nghẽn:

- **S3 Buckets**: Các bucket trong code đã có tùy chọn `force_destroy = true`, giúp Terraform tự động xóa toàn bộ đối tượng trước khi xóa bucket. Nếu cần dọn thủ công:
  ```bash
  aws s3 rm s3://<tên-bucket> --recursive
  aws s3 rb s3://<tên-bucket> --force
  ```

- **RDS Final Snapshot**: Cấu hình `rds.tf` đã đặt `skip_final_snapshot = true` để hỗ trợ việc xóa nhanh sau workshop.

---

#### Bước 3: Xác minh Dọn dẹp Tài nguyên

Xác nhận lại rằng toàn bộ tài nguyên đã được xóa sạch khỏi tài khoản AWS của bạn:

```bash
# Kiểm tra xem VPC chính đã được xóa chưa
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=main" \
  --query "Vpcs[*].VpcId"
```

Kết quả trả về dự kiến:
```json
[]
```

![verify cleanup](/images/workshop/5.9/2.png)

---

#### Tổng kết Workshop 🎉

Chúc mừng! Bạn đã hoàn thành xuất sắc:
- Khởi tạo toàn bộ **Learning Management System (LMS) 3 tầng** trên AWS bằng **Terraform**.
- Bảo mật hạ tầng backend với **Regional AWS WAF v2** và **AWS API Gateway HTTP API**.
- Phục vụ ứng dụng frontend qua chuẩn HTTPS bằng **AWS Amplify Hosting**.
- Tự động hóa quá trình deploy với **GitHub Actions** CI/CD pipelines.
- Dọn dẹp sạch sẽ toàn bộ tài nguyên bằng lệnh **`terraform destroy`**.
