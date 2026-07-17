---
title : "Prerequiste"
date : 2024-01-01 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### Create a Personal IAM User with MFA

Never use the root user for regular work. Create a personal administrator user instead.

1. Open the **IAM** console.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_console.jpeg)
2. In the left menu, click **Users** → **Create user**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_create_user.jpeg)
3. User name: your name or a consistent identifier, e.g. phatnguyen. Check Provide user access to the AWS Management Console.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_name_and_check.jpeg)
4. Click **Next**.Select **Attach policies directly** → check **AdministratorAccess**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_attach.jpeg)
5. Click **Next** → **Create user**.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_click_create.jpeg)
6. Click into the new user → **Security credentials** tab → **Assign MFA device**.
![overview](/images/5-Workshop/5.2-Prerequisite/MFA.jpeg)
7. Enter name and choose authenticator app
![overview](/images/5-Workshop/5.2-Prerequisite/Name_Auth.jpeg)
8. Choose **Authenticator app**, scan the QR code with an app such as Google Authenticator or Authy, and enter two consecutive codes to confirm.
![overview](/images/5-Workshop/5.2-Prerequisite/Scan_QR.jpeg)
9. Sign out of the root user. From now on, sign in as this IAM user for all work in this workshop.

#### Install and Configure the AWS CLI

1. Install the AWS CLI from [aws.amazon.com/cli](https://aws.amazon.com/cli).
2. Verify the install:
```bash
   aws --version
```
3. Create an access key for your IAM user: IAM console → your user → 
![overview](/images/5-Workshop/5.2-Prerequisite/click_new_user.jpeg)
4. **Security credentials** tab → **Create access key** → 
![overview](/images/5-Workshop/5.2-Prerequisite/sercurity_crea.jpeg)
![overview](/images/5-Workshop/5.2-Prerequisite/access_key.jpeg)
5. choose **Command Line Interface (CLI)** → confirm → **Create access key**. 
![overview](/images/5-Workshop/5.2-Prerequisite/checked_CLI.jpeg)
![overview](images/5-Workshop/5.2-Prerequisite/clickCreate_AccessKey.jpeg)
6. Copy the **Access Key ID** and **Secret Access Key** — the secret is only shown once.
![overview](/images/5-Workshop/5.2-Prerequisite/download_key.jpeg)
7. Configure a named profile (avoid the default profile, so multiple AWS accounts/users don't conflict on the same machine):
```bash
   aws configure --profile yourname
```
8. Enter:
   - **AWS Access Key ID** — from step 3
   - **AWS Secret Access Key** — from step 3
   - **Default region** — \ap-southeast-1\
   - **Default output format** — \json\
9. Verify the profile works:
```bash
   aws sts get-caller-identity --profile yourname
```
   This returns your account ID and user ARN.

#### Install Terraform, Python, and Git

1. Install Terraform from [terraform.io/downloads](https://www.terraform.io/downloads). Verify:
```bash
   terraform -version
```
2. Install Python 3.12 from [python.org](https://python.org). Verify:
```bash
   python3 --version
```
3. Install Git from [git-scm.com](https://git-scm.com). Verify:
```bash
   git --version
```


#### Clone the Repository

1. Clone the project:
```bash
   git clone https://github.com/your-org/Project_Abstract.git
   cd Project_Abstract
```
2. This workshop provisions named, globally-unique resources (S3 bucket names, Cognito domain prefixes). If following this guide with your own AWS account, change these values before running Terraform to avoid collisions with other readers:
   - S3 bucket names (must be globally unique across all of AWS)
   - Cognito Hosted UI domain prefix (must be globally unique per region)
   - Any hardcoded account ID references — replace with your own account ID from aws sts get-caller-identity