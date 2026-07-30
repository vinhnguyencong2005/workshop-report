---
title : "API Gateway HTTP API"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

Các trình duyệt web hiện đại áp dụng chính sách bảo mật **Mixed Content**: khi ứng dụng frontend được tải qua kết nối an toàn `https://` (như trên AWS Amplify), trình duyệt sẽ **chặn hoàn toàn** bất kỳ lệnh gọi API không an toàn `http://` nào.

Để cung cấp một endpoint HTTPS quản lý cho Application Load Balancer mà không yêu cầu xác minh tên miền hoặc chứng chỉ SSL thủ công, chúng ta khởi tạo một **AWS API Gateway HTTP API** (`apigateway.tf`).

#### 1. Tài nguyên API Gateway HTTP API (`apigateway.tf`)

```hcl
resource "aws_apigatewayv2_api" "alb_https" {
  name          = "alb-https-proxy"
  protocol_type = "HTTP"
  target        = "http://${aws_lb.main.dns_name}"

  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH"]
    allow_headers = ["*"]
    max_age       = 300
  }

  tags = {
    Name    = "alb-https-proxy"
    Purpose = "https-proxy-for-alb"
  }
}
```

#### 2. Lợi ích kiến trúc chính

- **HTTPS Tự động & Tức thì**: API Gateway tự động cấp endpoint bảo mật SSL (`https://<api-id>.execute-api.us-east-1.amazonaws.com`).
- **Quản lý CORS**: Cấu hình các headers Cross-Origin Resource Sharing cho các request đến từ frontend.
- **Proxy Tự động**: Chuyển tiếp toàn bộ đường dẫn (`/auth/login`, `/api/*`, `/health`) và phương thức HTTP (POST, GET, PUT, v.v.) trực tiếp đến ALB phía sau.
