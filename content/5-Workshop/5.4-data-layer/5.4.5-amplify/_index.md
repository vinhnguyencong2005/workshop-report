---
title : "AWS Amplify Hosting"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

AWS Amplify Hosting provides global CDN distribution, automatic HTTPS SSL certificates, and Single Page Application (SPA) rewrite rules for hosting web frontend applications (React, Vue, Vite).

In our architecture, AWS Amplify hosts the static frontend application and injects the `VITE_API_BASE_URL` environment variable pointing to the backend API Gateway HTTPS endpoint.

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

  # Injects API Gateway HTTPS URL for build and client runtime
  environment_variables = {
    VITE_API_BASE_URL   = aws_apigatewayv2_api.alb_https.api_endpoint
    AMPLIFY_DIFF_DEPLOY = "true"
  }

  # SPA Rewrite/Redirect rule (200 rewrite to index.html for client-side routing)
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
Amplify provides a default domain (`https://main.<app-id>.amplifyapp.com`) with automated SSL certificate provisioning out of the box.
{{% /notice %}}
