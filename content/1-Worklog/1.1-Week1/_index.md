---
title: "Week 1 Worklog"
date: 2026-03-30
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Connect and get acquainted with members of the First Cloud Journey (FCAJ).
* Understand basic AWS services, how to use the Management Console & AWS CLI.
* Grasp core concepts of IAM, VPC, and EC2.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - Get acquainted with FCAJ members <br> - Read and take note of internship unit rules and regulations                                                                                                   | 08/11/2025 | 08/11/2025      |
| 3   | - Learn about AWS and its types of services <br>&emsp; + Compute <br>&emsp; + Storage <br>&emsp; + Networking <br>&emsp; + Database <br>&emsp; + ... <br>                                              | 08/12/2025 | 08/12/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Create AWS Free Tier account <br> - Learn about AWS Console & AWS CLI <br> - **Practice:** <br>&emsp; + Create AWS account <br>&emsp; + Install & configure AWS CLI <br> &emsp; + How to use AWS CLI | 08/13/2025 | 08/13/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Learn basic EC2: <br>&emsp; + Instance types <br>&emsp; + AMI <br>&emsp; + EBS <br>&emsp; + ... <br> - SSH connection methods to EC2 <br> - Learn about Elastic IP   <br>                            | 08/14/2025 | 08/15/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Practice:** <br>&emsp; + Launch an EC2 instance <br>&emsp; + Connect via SSH <br>&emsp; + Attach an EBS volume                                                                                     | 08/15/2025 | 08/15/2025      | <https://cloudjourney.awsstudygroup.com/> |


### Week 1 Achievements:

* **AWS Cost & Account Security Setup:** Successfully initialised an AWS Free Tier account, configured multi-factor authentication (MFA) on the Root user, and established zero-spend threshold alarms via **AWS Budgets** to strictly monitor resource consumption.

* **AWS CLI Mastery:** Configured the local terminal with the AWS CLI tool using secure Access Keys. Mastered core command patterns (`aws ec2 describe-instances`, `aws iam list-users`) to read and control cloud resources programmatically without using the web UI.

* **VPC Infrastructure Implementation:** Designed and configured a basic Virtual Private Cloud (VPC) complete with Public Subnets, an Internet Gateway (IGW), and modified Route Tables to direct public-facing internet traffic properly.

* **Secure Compute Management:** Provisioned an Amazon EC2 Linux instance using customized Amazon Machine Images (AMIs). Managed network access via security groups by mapping inbound rules to restrict port access (Port 22 for SSH, Port 80 for HTTP).

* **Instance Profiling Execution:** Applied security best practices by creating an AWS IAM Role with specific Amazon S3 read-only permissions and attached it to an EC2 instance via an instance profile. Verified that applications running inside the instance could securely access S3 buckets without hardcoding cleartext AWS credentials inside the server code.
