---
title : "AWS WAF v2 (Regional)"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.3.6. </b> "
---

AWS WAF (Web Application Firewall) kiểm tra lưu lượng truy cập web đầu vào để ngăn chặn các lỗ hổng phổ biến, tấn công SQL injection, cross-site scripting (XSS), bad user agents và giới hạn tần suất truy cập (rate-limiting DDoS) trước khi lưu lượng chạm tới các máy chủ backend.

Trong kiến trúc này, chúng ta cấu hình một **Regional WAF v2 Web ACL** và gắn trực tiếp vào Application Load Balancer (`aws_lb.main`).

#### 1. Regional WAF Web ACL (`waf.tf`)

```hcl
resource "aws_wafv2_web_acl" "alb_waf" {
  name        = "alb-regional-waf"
  description = "Regional WAF Web ACL protecting backend Application Load Balancer"
  scope       = "REGIONAL"

  default_action {
    allow {}
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "alb-regional-waf-metrics"
    sampled_requests_enabled   = true
  }

  # Luật 1: Common Rule Set (Bảo vệ các lỗ hổng OWASP Top 10)
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 10

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesCommonRuleSetMetric"
      sampled_requests_enabled   = true
    }
  }

  # Luật 2: Known Bad Inputs Rule Set
  rule {
    name     = "AWSManagedRulesKnownBadInputsRuleSet"
    priority = 20

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesKnownBadInputsRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesKnownBadInputsRuleSetMetric"
      sampled_requests_enabled   = true
    }
  }

  # Luật 3: Amazon IP Reputation List
  rule {
    name     = "AWSManagedRulesAmazonIpReputationList"
    priority = 30

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesAmazonIpReputationList"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesAmazonIpReputationListMetric"
      sampled_requests_enabled   = true
    }
  }

  # Luật 4: Giới hạn tần suất truy cập IP (2,000 requests / 5 phút)
  rule {
    name     = "RateLimitRule"
    priority = 40

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 2000
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRuleMetric"
      sampled_requests_enabled   = true
    }
  }

  tags = {
    Name = "alb-regional-waf"
  }
}
```

#### 2. Gắn WAF vào Application Load Balancer

```hcl
resource "aws_wafv2_web_acl_association" "alb_waf_assoc" {
  resource_arn = aws_lb.main.arn
  web_acl_arn  = aws_wafv2_web_acl.alb_waf.arn
}
```

{{% notice info %}}
Bằng cách sử dụng `scope = "REGIONAL"`, WAF v2 sẽ được gắn trực tiếp vào các tài nguyên ALB trong cùng region (`us-east-1`).
{{% /notice %}}
