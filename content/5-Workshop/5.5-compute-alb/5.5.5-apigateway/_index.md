---
title : "API Gateway HTTP API"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5.5. </b> "
---

Modern web browsers enforce **Mixed Content** policies: when a web frontend is served securely over `https://` (such as on AWS Amplify), browsers block any insecure `http://` API calls.

To provide a managed HTTPS endpoint for our backend Application Load Balancer without requiring custom SSL certificate domain validation, we provision an **AWS API Gateway HTTP API** (`apigateway.tf`).

#### 1. API Gateway HTTP API Resource (`apigateway.tf`)

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

#### 2. Key Architecture Benefits

- **Instant Managed HTTPS**: API Gateway automatically provides an SSL-secured endpoint (`https://<api-id>.execute-api.us-east-1.amazonaws.com`).
- **CORS Management**: Configures cross-origin resource sharing headers for incoming frontend requests.
- **Transparent Proxying**: Forwards all paths (`/auth/login`, `/api/*`, `/health`) and HTTP verbs directly to the underlying ALB.
