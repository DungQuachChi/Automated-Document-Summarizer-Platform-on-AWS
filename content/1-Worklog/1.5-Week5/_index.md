---
title: "Week 5 Worklog"
date: 2026-04-27
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---


### Week 5 Objectives:

* Secure cloud resources across networking, identities, data layers, and environments.
* Deep dive into compliance governance, centralized firewalls, and automated threat detection.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Study Identity Federation via **AWS Single Sign-On (AWS IAM Identity Center)** <br> - Master **IAM Permission Boundaries** and advanced Policies with Conditions | 2026-04-27 | 2026-04-27 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Evaluate baseline posture via **AWS Security Hub** <br> - Lock down network perimeters using **Private Access to S3 with VPC Endpoints** | 2026-04-28 | 2026-04-28 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Protect public endpoints using **AWS WAF (Web Application Firewall)** <br> - Enforce data encryption mechanisms via **AWS KMS (Key Management Service)** | 2026-04-29 | 2026-04-29 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Discover sensitive data assets via **Amazon Macie** <br> - Store application keys securely via **AWS Secrets Manager** <br> - Centralize security policies via **AWS Firewall Manager** | 2026-04-30 | 2026-05-01 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Deploy continuous threat tracking via **AWS GuardDuty** <br> - Automate secure golden image pipelines using **EC2 Image Builder** <br> - Authenticate end-users via **Amazon Cognito** & review **S3 Security Best Practices** | 2026-05-01 | 2026-05-01 | <https://cloudjourney.awsstudygroup.com/> |

### Week 5 Achievements:

* Enterprise Identity & Privilege Isolation: Configured user access control structures using AWS IAM Identity Center (Single Sign-On). Enforced IAM Permission Boundaries on sub-admin identities to prevent privilege escalation, ensuring users cannot grant themselves unauthorized access permissions.

* Perimeter Hardening & Private Network Access: Provisioned a Gateway VPC Endpoint for Amazon S3, allowing traffic from internal private EC2 instances to communicate with storage storage nodes via the AWS internal backbone network, completely bypassing the public internet.

* Application Layer Edge Protection: Deployed AWS WAF rules directly in front of Application Load Balancers. Built custom inspection rulesets to filter out common web threats like SQL Injection (SQLi) and Cross-Site Scripting (XSS), keeping public endpoints safe.

* Cryptographic Data Security Governance: Configured automated customer-managed encryption keys using AWS KMS. Integrated Amazon Macie to scan S3 data lakes using machine learning, automatically identifying and alerting on exposed Personally Identifiable Information (PII).

* Automated Secrets Rotation & Threat Detection: Configured AWS Secrets Manager to automate database password rotation without causing application downtime. Activated AWS GuardDuty to continuously analyze CloudTrail and VPC logs for suspicious activities or compromised instances.

* Golden Image Pipeline Automation: Used EC2 Image Builder to build an automated operating system patching pipeline. This setup automates the creation, testing, and deployment of secure, hardened Amazon Machine Images (AMIs) across the entire AWS footprint.