---
title : "AWS Amplify Hosting"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

AWS Amplify Hosting cung cấp phân phối CDN toàn cầu, chứng chỉ HTTPS SSL tự động, và các quy tắc rewrite cho ứng dụng Single Page Application (SPA) khi hosting ứng dụng web frontend (React, Vue, Vite).

Trong kiến trúc này, AWS Amplify host ứng dụng frontend tĩnh và truyền biến môi trường `VITE_API_BASE_URL` trỏ đến endpoint HTTPS API Gateway backend.

#### 1. Amplify App (`amplify.tf`)

```hcl
resource "aws_amplify_app" "frontend" {
  name       = var.amplify_app_name
  repository = var.amplify_repository_url != "" ? var.amplify_repository_url : null

  build_spec = <<-EOT
    version: 1
    frontend:
      phases:
        preBuild:
          commands:
            - npm ci
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: dist
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
  EOT

  # Truyền URL HTTPS API Gateway cho quá trình build và runtime client
  environment_variables = {
    VITE_API_BASE_URL   = aws_apigatewayv2_api.alb_https.api_endpoint
    AMPLIFY_DIFF_DEPLOY = "true"
  }

  # Quy tắc Rewrite/Redirect SPA (điều hướng 200 rewrite về index.html cho client-side routing)
  custom_rule {
    source = "</^[^.]+$|\\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json)$)([^.]+$)/>"
    target = "/index.html"
    status = "200"
  }

  tags = {
    Name    = var.amplify_app_name
    Purpose = "frontend-hosting"
  }
}
```

#### 2. Amplify Production Branch

```hcl
resource "aws_amplify_branch" "main" {
  app_id      = aws_amplify_app.frontend.id
  branch_name = var.amplify_branch_name

  framework = "Web"
  stage     = "PRODUCTION"

  environment_variables = {
    VITE_API_BASE_URL = aws_apigatewayv2_api.alb_https.api_endpoint
  }

  tags = {
    Name = "${var.amplify_app_name}-${var.amplify_branch_name}"
  }
}
```

{{% notice tip %}}
Amplify tự động cung cấp một tên miền mặc định (`https://main.<app-id>.amplifyapp.com`) cùng chứng chỉ SSL tự động gia hạn mà không cần cấu hình thủ công.
{{% /notice %}}
