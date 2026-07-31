---
title : "Các bước chuẩn bị"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Các công cụ cần thiết

Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt các công cụ sau. Chạy lệnh xác minh để kiểm tra:

| Công cụ | Phiên bản | Hướng dẫn cài đặt | Kiểm tra |
|------|---------|---------|--------|
| **Terraform** | ≥ 1.5 | [Tải về](https://developer.hashicorp.com/terraform/downloads) | `terraform version` |
| **AWS CLI** | v2 | [Hướng dẫn cài đặt](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) | `aws --version` |
| **Git** | Bất kỳ | [Tải về](https://git-scm.com/downloads) | `git --version` |
| **Node.js** | ≥ 18 | [Tải về](https://nodejs.org/) | `node --version` |

{{% notice tip %}}
Nếu bất kỳ lệnh nào báo lỗi "command not found", hãy làm theo liên kết cài đặt tương ứng với hệ điều hành của bạn trước khi tiếp tục.
{{% /notice %}}

##### 1. Tạo Access Keys

1. Nhấp vào tên người dùng ở góc trên bên trái $\rightarrow$ chọn tab **Security credentials**.
2. Cuộn xuống phần **Access keys** $\rightarrow$ nhấp **Create access key**.

![](/images/workshop/5.2/1.png)

3. Chọn **Command Line Interface (CLI)** $\rightarrow$ tích chọn ô xác nhận $\rightarrow$ nhấp **Next**.

![](/images/workshop/5.2/2.png)

4. Nhấp **Create access key**.

![](/images/workshop/5.2/3.png)

5. **Lưu lại cả hai giá trị** (tải file .csv hoặc sao chép trực tiếp):

| Khóa | Định dạng mẫu |
|-----|------------|
| Access Key ID | `YOUR_ACCESS_KEY` |
| Secret Access Key | `YOUR_SECRET_ACCESS_KEY` |

{{% notice warning %}}
Secret Access Key chỉ hiển thị **duy nhất một lần**. Hãy lưu lại ngay bây giờ. Không bao giờ commit khóa này lên Git hoặc chia sẻ công khai.
{{% /notice %}}

![access keys](/images/workshop/5.2/4.png)

##### 2. Cấu hình AWS CLI {#configure-aws-cli}

```bash
aws configure
```

Nhập các giá trị đã lưu ở bước trước:

```
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

![](/images/workshop/5.2/5.png)

#### Clone Repository

```bash
git clone https://github.com/vinhnguyencong2005/TTNT-IaC
cd TTNT-IaC
```
