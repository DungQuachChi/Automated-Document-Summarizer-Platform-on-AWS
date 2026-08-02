---
title : "Prerequiste"
date : 2026-07-14
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
3. User name: your name or a consistent identifier, e.g. workshop-admin. Check Provide user access to the AWS Management Console.
![overview](/images/5-Workshop/5.2-Prerequisite/IAM_name_and_check.jpeg)
4. Click **Next** then select **Attach policies directly** → check **AdministratorAccess**.
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
3. Create an access key for your IAM user: IAM console → Your user 
![overview](/images/5-Workshop/5.2-Prerequisite/click_new_user.jpeg)
4. **Security credentials** tab → **Create access key**  
![overview](/images/5-Workshop/5.2-Prerequisite/sercurity_crea.jpeg)
![overview](/images/5-Workshop/5.2-Prerequisite/access_key.jpeg)
5. choose **Command Line Interface (CLI)** → Confirm → **Create access key**. 
![overview](/images/5-Workshop/5.2-Prerequisite/checked_CLI.jpeg)
![overview](/images/5-Workshop/5.2-Prerequisite/clickCreate_AccessKey.jpeg)
6. Copy the **Access Key ID** and **Secret Access Key**(the secret is only shown once).
![overview](/images/5-Workshop/5.2-Prerequisite/download_key.jpeg)
7. Configure a named profile (avoid the default profile, so multiple AWS accounts/users don't conflict on the same machine):
```bash
   aws configure --profile yourname
```
8. Enter:
   - **AWS Access Key ID**: From step 3.
   - **AWS Secret Access Key**: From step 3.
   - **Default region**: ap-southeast-1.
   - **Default output format**: json.
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
   git clone https://github.com/pHTSOm/Project_Abstract.git
   cd Project_Abstract
```
2. The workshop provides the specific resource names that are uniquely available around the world (S3 bucket names and Cognito domain prefixes). If you use your account to follow the instructions in the tutorial, please change the mentioned names before executing Terraform commands:
   - S3 bucket names: always globally unique in the AWS environment.
   - Cognito Hosted UI domain prefix: globally unique in each region.
   - Replace account ID references with your own from aws sts get-caller-identity.