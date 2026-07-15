---
title: "Week 6 Worklog"
date: 2026-05-04
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Establish multi-region networks and highly resilient system architectures.
* Connect advanced microservices infrastructure using managed container engines and CI/CD pipelines.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Centralize data protection policies using **AWS Backup** <br> - Learn complex networking with **VPC Peering** and **AWS Transit Gateway** | 2026-05-04 | 2026-05-04 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Implement asynchronous microservice message routers with **Amazon SQS** and **SNS** <br> - Set up shared volume storage fabrics via **Amazon EBS Multi-Attach** | 2026-05-05 | 2026-05-05 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Analyze high availability for enterprise databases (WSFC and SQL Server HA on AWS 2019/2022) <br> - Review **Cost Savings (Savings Plans / Reserved Instances)** and **Cost Visualization & Analytics** | 2026-05-06 | 2026-05-06 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Perform big-data analytics profiling via **AWS Glue** and **Amazon Athena** queries <br> - Build microservices foundations: **Docker** containerization and **Amazon ECS** | 2026-05-07 | 2026-05-08 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Provision infrastructure for ECS with CDK <br> - Deploy complete pipelines: **AWS CodePipeline**, Automated Deployments, and DevOps practices | 2026-05-08 | 2026-05-08 | <https://cloudjourney.awsstudygroup.com/> |

### Week 6 Achievements:

* Centralized Enterprise Transit Architecture: Designed and deployed a hub-and-spoke network architecture using AWS Transit Gateway. This centralized network routing across distinct VPC environments, eliminating the complexity of managing chaotic mesh-based VPC Peering connections.

* Asynchronous Microservices Decoupling: Implemented decoupled application architectures using Amazon SQS (First-In, First-Out queue settings) and Amazon SNS. Built a fan-out messaging pattern where a single event notification triggers multiple parallel backend processing systems simultaneously.

* Serverless Big Data ETL Extraction: Built a serverless data analytics pipeline. Used AWS Glue Crawlers to automatically discover data schemas in S3, cataloging them so they could be queried instantly with standard SQL via Amazon Athena.

* Containerized Infrastructure Deployment: Developed multi-stage Dockerfiles to package application code into lightweight, efficient runtime containers. Published these images to Amazon Elastic Container Registry (ECR) and deployed them using Amazon ECS.

* DevOps Pipeline Automation with CDK: Used the AWS CDK to define programmatic infrastructure configurations for an ECS microservice stack. Managed the deployment using AWS CodePipeline, creating a fully automated CI/CD pipeline that pulls source updates, builds container images, and deploys updates to production without downtime.