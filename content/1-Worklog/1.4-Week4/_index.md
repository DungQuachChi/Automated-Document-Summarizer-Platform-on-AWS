---
title: "Week 4 Worklog"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Master Infrastructure as Code (IaC) architectures.
* Explore system optimization, workload migration mechanisms, and storage automation.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Learn **AWS CloudFormation** essentials <br> - Deep dive into **AWS CDK Essentials** and **AWS CDK Advanced** concepts | 2026-04-20 | 2026-04-20 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Complete the **Infrastructure as Code Workshop Series** <br> - Configure local development environment using **AWS Toolkit for VS Code** | 2026-04-21 | 2026-04-21 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Study Migrate to AWS: **VM Migration via AWS VM Import/Export** <br> - Study **AWS DMS (Database Migration Service)** and **SCT (Schema Conversion Tool)** | 2026-04-22 | 2026-04-22 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Configure **Disaster Recovery with AWS Elastic Disaster Recovery (DRS)** <br> - Perform Right-Sizing using **EC2 Resource Optimization** <br> - Set up Network Monitoring via **VPC Flow Logs** | 2026-04-23 | 2026-04-24 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Learn Billing Console Delegation and **Service Quotas** management <br> - Automate snapshot management with **Amazon EBS Data Lifecycle Manager** <br> - Implement **Anomaly Detection for EBS Backups** | 2026-04-24 | 2026-04-24 | <https://cloudjourney.awsstudygroup.com/> |

### Week 4 Achievements:

* Programmatic Infrastructure (IaC) Synthesis: Developed and executed reusable infrastructure templates via AWS CloudFormation (YAML) and programmatic code constructs using the AWS Cloud Development Kit (CDK). Successfully ran synthesis (`cdk synth`) and deployment (`cdk deploy`) tasks directly from a customized local environment via AWS Toolkit for VS Code.

* Enterprise Migration Strategy Architecture: Modeled complex migration pipelines using the AWS Schema Conversion Tool (SCT) to convert engine schemas, alongside AWS Database Migration Service (DMS) blueprints to configure continuous data replication tasks from on-premises to the cloud.

* Continuous Disaster Recovery Infrastructure: Configured non-disruptive continuous block-level data replication using AWS Elastic Disaster Recovery (DRS), enabling automated cloud failover readiness with low recovery time and recovery point objectives (RTO/RPO).

* Network Traffic Analytics & Optimization: Enabled VPC Flow Logs on target network interfaces, sending stream data directly into CloudWatch Logs. Used CloudWatch Logs Insights to analyze network payload patterns and detect dropped packets or unauthorized connection requests.

* Automated Data Protection Lifecycle Management: Automated block storage backup regimes using Amazon EBS Data Lifecycle Manager (DLM). Configured backup retention policies and layered Anomaly Detection for EBS Backups to flag unusual volume sizes or missing snapshots, protecting against ransomware.
