---
title: "Worklog Week 6"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives

- Deploy the entire application to AWS infrastructure using Terraform
- Set up CI/CD pipeline to automate backend and frontend deployments

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Run `terraform apply` to deploy EC2 on public subnet <br> - Manually install Node.js, PM2, clone backend source to the instance <br> - Configure environment variables (.env) and verify backend via public IP | 20/07/2026 | 20/07/2026 | |
| Tue | - Build and deploy frontend: `npm run build` → upload to S3 static website hosting <br> - Verify frontend connects to backend via public IP <br> - Begin researching how to move EC2 from public to private subnet for better security | 21/07/2026 | 21/07/2026 | |
| Wed | - Refactor Terraform: move EC2 to private subnet, configure NAT Gateway <br> - Experiment with deploy strategy: upload code to S3 → send SSM Run Command so instances pull code from S3 <br> - Attempt to sign up for Cloudflare CDN but account was rejected, decide to move on without it | 22/07/2026 | 22/07/2026 | |
| Thu | - Optimize deploy strategy: replace SSM trigger with instance pulling code from S3 on boot (user_data) <br> - Write `user_data.tftpl` script: install AWS CLI, Node.js, PM2, pull backend.zip, unzip, start app <br> - Configure DynamoDB on AWS and verify backend connectivity | 23/07/2026 | 23/07/2026 | |
| Fri | - Set up GitHub Actions workflow for backend: zip source → upload S3 → trigger ASG instance refresh <br> - Build deploy script for frontend: `aws s3 sync dist/` to S3 bucket <br> - Write IAM policies for EC2 (S3, DynamoDB, Secrets Manager, SSM permissions) | 24/07/2026 | 24/07/2026 | |
| Sat | - Test full automated deploy flow: push code → GitHub Actions run → ASG refresh → app live <br> - Log and fix issues: CORS, health check timeout, missing security group rules <br> - Fine-tune ALB target group health check to reduce unhealthy instance detection time | 25/07/2026 | 25/07/2026 | |
| Sun | - Clean up and reorganize Terraform directory structure <br> - Write documentation for infrastructure architecture and deployment process <br> - Run final end-to-end test: login, browse courses, take quiz, submit assignment on production | 26/07/2026 | 26/07/2026 | |

### Week 6 Results

- Complete AWS infrastructure: EC2 in private subnet, public ALB, RDS, ElastiCache Redis, DynamoDB
- Stable deployment strategy: backend via S3 + ASG instance refresh, frontend via S3 sync
- CI/CD pipeline with GitHub Actions operational, auto-deploying on push to `main` branch
- Abandoned Cloudflare after account rejection, using ALB DNS and S3 website endpoint directly
- Infrastructure architecture and deployment documentation written, ready for operations phase
