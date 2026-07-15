---
title: "Week 2 Worklog"
date: 2026-04-06
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Explore cloud development environments, storage services, and databases.
* Understand high availability, content delivery networks, and scaling architectures.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Learn Cloud Development with **AWS Cloud9** <br> - Study **Amazon S3** static website hosting fundamentals | 2026-04-06 | 2026-04-06 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Learn Database Essentials with **Amazon RDS** <br> - Understand database subnet groups and multi-AZ deployments | 2026-04-07 | 2026-04-07 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Learn **Amazon Lightsail** for simplified computing <br> - Deep dive into **Amazon Lightsail Containers** deployment | 2026-04-08 | 2026-04-08 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Learn system scaling using **EC2 Auto Scaling** and **Elastic Load Balancing** <br> - Set up fundamental system monitoring using **Amazon CloudWatch** | 2026-04-09 | 2026-04-10 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Learn Hybrid DNS Management with **Amazon Route 53** <br> - Set up Content Delivery with **Amazon CloudFront** & **Lambda@Edge** | 2026-04-10 | 2026-04-10 | <https://cloudjourney.awsstudygroup.com/> |                                                                                   | 08/15/2025 | 08/15/2025      | <https://cloudjourney.awsstudygroup.com/> |


### Week 2 Achievements:

* Cloud Development Environment Deployment: Configured an AWS Cloud9 cloud IDE, creating a pre-authenticated terminal environment to streamline code editing and resource management directly from a browser interface.

* S3 Static Web Hosting & Distribution: Configured an Amazon S3 bucket to serve a static website. Modified S3 Bucket Policies to allow public read access, and integrated the origin with Amazon CloudFront to distribute web assets across global edge locations, lowering latency.

* Relational Database Administration: Provisioned an Amazon RDS database instance within isolated private database subnet groups. Enabled Multi-AZ deployment configurations to establish automated failover safety replication mechanisms.

* Simplified Container Deployments: Used Amazon Lightsail Containers to quickly deploy lightweight web images, avoiding the overhead of configuring underlying virtual servers or target orchestrators.

* Auto-Scaling & Load Balancing Architecture:** Built a high-availability compute tier using Elastic Load Balancing (ALB) and EC2 Auto Scaling. Configured dynamic scaling policies driven by Amazon CloudWatch metrics to automatically launch or terminate instances based on CPU usage limits.

* Edge Computing Logic Implementation: Configured custom DNS routing records inside Amazon Route 53 and introduced Lambda@Edge processing to modify request headers on-the-fly at regional edge caches, improving edge security routing.