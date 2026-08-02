---
title : "Các bước chuẩn bị"
date : 2026-07-14
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### Tạo người dùng IAM cá nhân kèm MFA

Không bao giờ sử dụng tài khoản root cho công việc hàng ngày. Thay vào đó, hãy tạo một người dùng quản trị (administrator) cá nhân.

1. Mở console **IAM**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_console.jpeg)
2. Ở menu bên trái, chọn **Users** → **Create user**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_create_user.jpeg)
3. User name: tên của bạn hoặc một định danh nhất quán, ví dụ workshop-admin. Tích chọn Provide user access to the AWS Management Console.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_name_and_check.jpeg)
4. Nhấn **Next** rồi chọn **Attach policies directly** → tích chọn **AdministratorAccess**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_attach.jpeg)
5. Nhấn **Next** → **Create user**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_click_create.jpeg)
6. Nhấn vào người dùng vừa tạo → tab **Security credentials** → **Assign MFA device**.
![overview](/images/5-Workshop/5.2-Prerequisite/MFA.jpeg)
7. Nhập tên và chọn authenticator app
![overview](/images/5-Workshop/5.2-Prerequisite/Name_Auth.jpeg)
8. Chọn **Authenticator app**, quét mã QR bằng một ứng dụng như Google Authenticator hoặc Authy, sau đó nhập hai mã liên tiếp để xác nhận.
![overview](/images/5-Workshop/5.2-Prerequisite/Scan_QR.jpeg)
9. Đăng xuất khỏi tài khoản root. Từ giờ trở đi, hãy đăng nhập bằng người dùng IAM này cho mọi thao tác trong workshop này.

#### Cài đặt và cấu hình AWS CLI

1. Cài đặt AWS CLI từ [aws.amazon.com/cli](https://aws.amazon.com/cli).
2. Kiểm tra việc cài đặt:
```bash
   aws --version
```
3. Tạo access key cho người dùng IAM của bạn: console IAM → người dùng của bạn  
![overview](/images/5-Workshop/5.2-Prerequisite/click_new_user.jpeg)
4. Tab **Security credentials** → **Create access key**  
![overview](/images/5-Workshop/5.2-Prerequisite/sercurity_crea.jpeg)
![overview](/images/5-Workshop/5.2-Prerequisite/access_key.jpeg)
5. chọn **Command Line Interface (CLI)** → xác nhận → **Create access key**. 
![overview](/images/5-Workshop/5.2-Prerequisite/checked_CLI.jpeg)
![overview](/images/5-Workshop/5.2-Prerequisite/clickCreate_AccessKey.jpeg)
6. Sao chép **Access Key ID** và **Secret Access Key**(secret chỉ hiển thị một lần duy nhất).
![overview](/images/5-Workshop/5.2-Prerequisite/download_key.jpeg)
7. Cấu hình một profile có tên riêng (tránh dùng profile mặc định, để nhiều tài khoản/người dùng AWS không bị xung đột trên cùng một máy):
```bash
   aws configure --profile yourname
```
8. Nhập:
   - **AWS Access Key ID**: Từ bước 3
   - **AWS Secret Access Key**: Từ bước 3
   - **Default region**: ap-southeast-1
   - **Default output format**: json
9. Kiểm tra profile đã hoạt động:
```bash
   aws sts get-caller-identity --profile yourname
```
   Lệnh này trả về Account ID và ARN của người dùng.

#### Cài đặt Terraform, Python và Git

1. Cài đặt Terraform từ [terraform.io/downloads](https://www.terraform.io/downloads). Kiểm tra:
```bash
   terraform -version
```
2. Cài đặt Python 3.12 từ [python.org](https://python.org). Kiểm tra:
```bash
   python3 --version
```
3. Cài đặt Git từ [git-scm.com](https://git-scm.com). Kiểm tra:
```bash
   git --version
```


#### Clone Repository

1. Clone dự án:
```bash
   git clone https://github.com/pHTSOm/Project_Abstract.git
   cd Project_Abstract
```
2. Workshop này khởi tạo các tài nguyên có tên duy nhất trên toàn cầu (tên bucket S3, tiền tố domain Cognito). Nếu bạn làm theo hướng dẫn này bằng tài khoản AWS của riêng mình, hãy thay đổi các giá trị sau trước khi chạy Terraform để tránh trùng lặp với những người đọc khác:
   - Tên bucket S3 (phải là duy nhất trên toàn bộ AWS)
   - Tiền tố domain của Cognito Hosted UI (phải là duy nhất trong mỗi region)
   - Bất kỳ tham chiếu account ID nào bị hardcode — hãy thay bằng account ID của chính bạn lấy từ aws sts get-caller-identity