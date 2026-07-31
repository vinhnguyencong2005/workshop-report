---
title : "Clean Up"
date: 2026-07-30
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

To avoid ongoing AWS charges after completing your workshop demonstration, it is essential to tear down all provisioned cloud infrastructure.

Because everything was provisioned declaratively using **Terraform Infrastructure as Code**, you can destroy all 50+ AWS resources cleanly with a single command.

---

#### Step 1: Destroy Infrastructure (`terraform destroy`)

1. Open your terminal and navigate to the Terraform directory:
   ```bash
   cd Terraform_
   ```

2. Run the destroy command:
   ```bash
   terraform destroy
   ```

3. Terraform will display a list of all resources to be deleted. Type **`yes`** when prompted to confirm:

![terraform destroy](/images/workshop/5.9/1.png)

##### Expected Teardown Timeline:
- **0–2 min**: WAF Web ACL, API Gateway HTTP API, ALB listeners, Security Groups, IAM instance profiles
- **5-10 min**: EC2 Auto Scaling Group, Launch Template, S3 buckets, DynamoDB tables, AWS Amplify App
- **5–12 min**: RDS MySQL Multi-AZ DB Instance and ElastiCache Redis Replication Group
- **12–15 min**: NAT Gateways, Elastic IPs, VPC subnets, Internet Gateway, VPC

Total teardown time is approximately **20–25 minutes**.