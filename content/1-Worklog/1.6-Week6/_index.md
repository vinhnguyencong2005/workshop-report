---
title: "Worklog Week 6"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives

- Deploy the entire LMS application to AWS infrastructure using Terraform
- Set up a reliable CI/CD pipeline to automate backend and frontend deployments

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Run `terraform apply` to provision initial infrastructure: VPC, public subnets, EC2 instances <br> - SSH into instances, install Node.js 20 via nvm, clone backend source code manually <br> - Configure `.env` with database credentials, JWT secret, Redis and AWS region <br> - Install PM2 process manager, start backend on port 3000, verify via public IP <br> - Identify early issues: hardcoded config values, no auto-restart on crash, public exposure of EC2 | 20/07/2026 | 20/07/2026 | |
| Tue | - Migrate frontend hosting to **AWS Amplify Hosting** (`amplify.tf`) for global CDN edge distribution and automatic HTTPS SSL <br> - Configure SPA client-side rewrite rules in `amplify.tf` for index.html navigation <br> - Verify full-stack connectivity: frontend loads from Amplify over HTTPS <br> - Security review: EC2 directly on public internet is a major risk — begin researching private subnet architecture <br> - Add ALB, private subnets, and NAT Gateway to Terraform configuration | 21/07/2026 | 21/07/2026 | |
| Wed | - Refactor Terraform: move EC2 to private subnets, route outbound traffic through NAT Gateway <br> - Provision Application Load Balancer in public subnets forwarding to EC2 target group on port 3000 <br> - Provision **Regional AWS WAF v2** (`waf.tf`) attached to ALB with AWS Managed Rules (OWASP Top 10, Bad Inputs, IP Reputation) and IP Rate Limiting <br> - Solve deployment challenge: private instances have no public IP — design S3 + SSM strategy <br> - Provision **API Gateway HTTP API** (`apigateway.tf`) to provide managed HTTPS for ALB and solve browser Mixed Content errors | 22/07/2026 | 22/07/2026 | |
| Thu | - Improve deploy architecture: replace SSM push with user_data pull-on-boot for immutable deployments <br> - Write `user_data.tftpl` bootstrap script: install AWS CLI v2, nvm + Node.js 20, PM2, pull backend.zip from S3, unzip, `npm ci --production`, start app via PM2 <br> - Provision RDS MySQL (Multi-AZ) and ElastiCache Redis (2-node cluster) via Terraform <br> - Configure DynamoDB tables (ClassContent, ForumData, QuizContent, StudentSchedule, CourseAssign) on AWS <br> - Secure S3 frontend bucket by disabling public website access and turning on full public access block | 23/07/2026 | 23/07/2026 | |
| Fri | - Set up GitHub Actions CI/CD workflows: frontend via AWS Amplify CLI API (`create-deployment` + upload + `start-deployment`) and backend via S3 + ASG Rolling Refresh <br> - Write IAM policies for EC2 instance role: S3 read/write, DynamoDB CRUD, Secrets Manager, SSM managed instance <br> - Configure security groups with least-privilege: EC2 ingress only from ALB, RDS ingress only from EC2, Redis only from EC2 <br> - Configure Regional WAF v2 rate limiting rule (2,000 req/5min) and API Gateway CORS settings | 24/07/2026 | 24/07/2026 | |
| Sat | - Run full automated deployment test: push code → GitHub Actions triggers → frontend deploys to Amplify → ASG instance refresh rolls out backend instances → health checks pass <br> - Debug and fix Mixed Content security errors: route frontend API calls through API Gateway HTTPS endpoint (`https://<api-id>.execute-api.us-east-1.amazonaws.com`) <br> - Fine-tune ASG rolling refresh: set `InstanceWarmup: 0` and polling sleep interval to 10s for fast CI/CD execution <br> - Test ASG auto-scaling: manually trigger scale-out, verify new instances pull latest code and register with ALB <br> - Document all issues encountered and their resolutions | 25/07/2026 | 25/07/2026 | |
| Sun | - Reorganize Terraform project structure: split monolithic config into separate files (`waf.tf`, `amplify.tf`, `apigateway.tf`, etc.) <br> - Extract hardcoded values into variables (region, instance types, CIDR blocks) with `terraform.tfvars` <br> - Replace personal domain defaults with generic domain `lms.uni` (`app.lms.uni`) <br> - Write infrastructure README and Hugo workshop documentation with architecture diagram and deployment guide <br> - Final production smoke test: user registration → login → browse courses → take quiz → submit assignment → view grades <br> - Push all Terraform and application code to GitHub, verify CI/CD triggers on push | 26/07/2026 | 26/07/2026 | |

### Week 6 Results

- Fully automated AWS infrastructure provisioned via Terraform: VPC, ALB, Regional AWS WAF v2, AWS Amplify Hosting, API Gateway HTTP API, EC2 ASG in private subnets, RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB (5 tables), S3 (private buckets)
- Immutable deployment strategy established: backend via S3 artifact + ASG rolling instance refresh, frontend via AWS Amplify 3-step deployment API
- CI/CD pipeline operational on GitHub Actions: `git push main` triggers automated frontend release to Amplify and zero-downtime backend deployment
- Defense-in-depth security baseline implemented: Regional WAF v2 (OWASP Top 10, IP reputation, rate limiting), API Gateway HTTPS, private subnets, least-privilege security groups and IAM roles
- Browser Mixed Content security completely resolved using API Gateway HTTPS proxying to ALB
- Infrastructure documentation, Hugo workshop guides, and variable templates ready for team collaboration

