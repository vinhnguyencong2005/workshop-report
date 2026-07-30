---
title : "Tự động hóa CI/CD với GitHub Actions"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

Tự động hóa giúp các cập nhật code được triển khai mượt mà lên môi trường sản xuất. Trong phần này, chúng ta cấu hình các GitHub Repository Secrets và tìm hiểu cách kích hoạt quy trình CI/CD tự động cho cả **Frontend (AWS Amplify)** và **Backend (EC2 Auto Scaling Group)**.

---

#### Bước 1: Cấu hình GitHub Repository Secrets

Trước khi chạy các workflow, hãy thêm AWS Credentials và các endpoint output từ Terraform vào GitHub:

1. Mở các repository GitHub của bạn (`TTNT-frontend` và `TTNT-backend`).
2. Vào **Settings** $\rightarrow$ **Secrets and variables** $\rightarrow$ **Actions** $\rightarrow$ **New repository secret**.
3. Thêm các secrets sau:

| Tên Secret | Mô tả / Nguồn giá trị |
| :--- | :--- |
| `AWS_ACCESS_KEY_ID` | IAM User Access Key ID |
| `AWS_SECRET_ACCESS_KEY` | IAM User Secret Access Key |
| `AWS_REGION` | Region AWS (`us-east-1`) |
| `AMPLIFY_APP_ID` | Giá trị lấy từ `terraform output amplify_app_id` |
| `VITE_API_BASE_URL` | Giá trị lấy từ `terraform output backend_api_url` (API Gateway HTTPS) |

![github secrets](../../../images/workshop/5.8/1.png)

{{% notice tip %}}
Đảm bảo `VITE_API_BASE_URL` trỏ chính xác đến **URL HTTPS API Gateway** (`https://<api-id>.execute-api.us-east-1.amazonaws.com`) để tránh lỗi Mixed Content trên trình duyệt.
{{% /notice %}}

---

#### Bước 2: Nguyên lý hoạt động của các Workflow Deployment

- **Workflow Frontend (`frontend-deploy.yml`)**:
  Biên dịch ứng dụng web React/Vite, đóng gói sản phẩm build, tải lên AWS Amplify thông qua presigned upload URL và kích hoạt phát hành toàn cầu trên CDN.

- **Workflow Backend (`backend-deploy.yml`)**:
  Đóng gói mã nguồn API Node.js, tải tệp zip lên S3 deployment bucket riêng tư, và kích hoạt tính năng **EC2 Auto Scaling Group Rolling Instance Refresh** tự động để cập nhật máy chủ không làm gián đoạn dịch vụ.

---

#### Bước 3: Cách kích hoạt Deployment

Bạn có thể kích hoạt các tiến trình deploy theo hai cách:

##### Cách A: Tự động kích hoạt khi Git Push (Quy trình tiêu chuẩn)

Bất kỳ khi nào bạn push code mới lên nhánh `main`, GitHub Actions sẽ tự động khởi chạy workflow:

```bash
# 1. Stage các thay đổi
git add .

# 2. Commit code
git commit -m "Update application code"

# 3. Push lên GitHub
git push origin main
```

![git push trigger](../../../images/workshop/5.8/2.png)

---

##### Cách B: Kích hoạt thủ công trên giao diện GitHub (`workflow_dispatch`)

Bạn cũng có thể khởi chạy deploy thủ công bất kỳ lúc nào trực tiếp trên trình duyệt web:

1. Mở repository GitHub trên trình duyệt.
2. Nhấp vào tab **Actions** phía trên.
3. Chọn workflow ở cột bên trái:
   - Cho frontend: **Frontend Deploy to AWS Amplify**
   - Cho backend: **Backend Deploy via S3 & ASG Rolling Refresh**
4. Nhấp vào nút **Run workflow** bên phải.
5. Chọn nhánh **`main`** và nhấp **Run workflow**.

![manual trigger](../../../images/workshop/5.8/3.png)

---

#### Bước 4: Theo dõi và Kiểm tra Deploy

1. Nhấp vào lượt chạy workflow đang hoạt động trong tab **Actions** để xem chi tiết log thời gian thực.
2. Khi tiến trình hoàn tất với màu xanh (✅ **Success**), kiểm tra ứng dụng:
   - Mở app frontend: `https://main.<app-id>.amplifyapp.com`
   - Kiểm tra API health: `curl -s https://<api-id>.execute-api.us-east-1.amazonaws.com/health`

![workflow success](../../../images/workshop/5.8/4.png)

---

#### Tóm tắt bài học

- **Phát hành Frontend Tự động**: AWS Amplify build và deploy bản cập nhật tự động mỗi khi `git push`.
- **Zero-Downtime Rolling Update**: ASG thay thế các máy chủ backend êm ái mà không gây gián đoạn.
- **Kích hoạt linh hoạt**: Hỗ trợ cả kích hoạt tự động qua `push` và kích hoạt thủ công qua `workflow_dispatch`.
