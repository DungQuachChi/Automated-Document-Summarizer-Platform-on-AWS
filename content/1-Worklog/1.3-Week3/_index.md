---
title: "Week 3 Worklog"
date: 2026-04-13
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Explore NoSQL databases, caching mechanisms, and enterprise hybrid networking environments.
* Master Serverless computing basics and advanced management operations.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Learn NoSQL Database Essentials with **Amazon DynamoDB** <br> - Learn In-Memory Caching using **Amazon ElastiCache** | 2026-04-13 | 2026-04-13 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Participate in the **Networking on AWS Workshop** <br> - Study Windows Workloads on AWS and **AWS Managed Microsoft AD** | 2026-04-14 | 2026-04-14 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Review architectures for **Building Highly Available Web Applications** <br> - Study Serverless Automation with **AWS Lambda** | 2026-04-15 | 2026-04-15 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Attend **CloudWatch Advanced Workshop** <br> - Configure Advanced Monitoring with **CloudWatch and Grafana** Integration | 2026-04-16 | 2026-04-17 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Resource Organization with **Tags and Resource Groups** <br> - Implement Access Control using **IAM and Resource Tags** <br> - Practice Systems Management with **AWS Systems Manager (SSM)** & **Session Manager** | 2026-04-17 | 2026-04-17 | <https://cloudjourney.awsstudygroup.com/> |

### Week 3 Achievements:

* NoSQL Datastore & Caching Configuration:** Provisioned Amazon DynamoDB tables, configuring Partition Keys and Sort Keys for optimized data queries. Linked a Redis-backed Amazon ElastiCache cluster to offload read intensive operations, reducing backend query latency.

* Enterprise Windows Identity Deployment: Explored enterprise cloud architecture patterns by deploying AWS Managed Microsoft AD inside a private subnet array to facilitate centralized object authentication and secure management of Windows workloads.

* Serverless Event-Driven Logic Automation: Coded and deployed serverless operational processes using AWS Lambda. Set up automated execution logic triggered by object uploads to S3, parsing events without paying for idle server time.

* Advanced Metrics Dashboard Federation: Completed the Advanced CloudWatch workshop. Connected external Grafana visualization platforms directly to Amazon CloudWatch metrics using IAM access roles, creating centralized performance dashboards.

* Attribute-Based Access Control (ABAC) Governance: Enforced organization-wide tag standards via AWS Resource Groups Authored advanced IAM policies that evaluated environment tags (`Env: Production`), restricting modification access based on resource key criteria.

* Secure Bastion-less Systems Management: Integrated AWS Systems Manager (SSM) across running server clusters. Leveraged Session Manager to execute terminal commands on private subnet EC2 servers, completely eliminating the need for open SSH ports or inbound bastion host routes.