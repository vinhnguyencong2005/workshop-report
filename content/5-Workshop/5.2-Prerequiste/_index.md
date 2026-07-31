---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Tools

Before starting, make sure you have these installed. Run the verify command to confirm:

| Tool | Version | Install | Verify |
|------|---------|---------|--------|
| **Terraform** | ≥ 1.5 | [Download](https://developer.hashicorp.com/terraform/downloads) | `terraform version` |
| **AWS CLI** | v2 | [Install guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) | `aws --version` |
| **Git** | Any | [Download](https://git-scm.com/downloads) | `git --version` |
| **Node.js** | ≥ 18 | [Download](https://nodejs.org/) | `node --version` |

{{% notice tip %}}
If any command returns "command not found", follow the install link for your OS before continuing.
{{% /notice %}}

##### 1. Generate Access Keys

1. Click the top left user name → **Security credentials** tab
2. Scroll to **Access keys** → **Create access key**

![](/images/workshop/5.2/1.png)

3. Choose **Command Line Interface (CLI)** → check the confirmation box → **Next**

![](/images/workshop/5.2/2.png)

4. Click **Create access key**

![](/images/workshop/5.2/3.png)

5. **Save both values** (download .csv or copy them):

| Key | Looks Like |
|-----|------------|
| Access Key ID | `YOUR_ACCESS_KEY` |
| Secret Access Key | `YOUR_SECRET_ACCESS_KEY` |

{{% notice warning %}}
The Secret Access Key is shown **only once**. Save it now. Never commit it to Git or share it publicly.
{{% /notice %}}

![access keys](/images/workshop/5.2/4.png)

##### 2. Configure AWS CLI {#configure-aws-cli}

```bash
aws configure
```

Enter the values from step 3:

```
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

![](/images/workshop/5.2/5.png)

#### Clone the Repository

```bash
# Terraform IaC
git clone https://github.com/vinhnguyencong2005/TTNT-IaC
# Backend source code
git clone https://github.com/vinhnguyencong2005/TTNT-backend
# Fronend source code
git clone https://github.com/vinhnguyencong2005/TTNT-frontend
```
