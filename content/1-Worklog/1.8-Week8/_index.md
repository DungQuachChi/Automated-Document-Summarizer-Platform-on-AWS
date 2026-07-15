---
title: "Week 8 Worklog"
date: 2026-05-18
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---


### Week 8 Objectives:

* Complete the Document Management System series, along with container management frameworks.
* Explore orchestrations across Elastic Beanstalk, Amazon ECS/Fargate, EKS, and ROSA enterprise environments.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - **Serverless DMS Series:** Build serverless CRUD with Lambda/DynamoDB <br> - Integrate **AWS Amplify** storage/auth and link frontend via API Gateway | 2026-05-18 | 2026-05-18 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Serverless DMS Series (Cont.):** Deploy using SAM, configure CloudFront, and add advanced search <br> - Setup DevOps pipelines and perform distributed tracing with **AWS X-Ray** | 2026-05-19 | 2026-05-19 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Complete **Serverless Web App Workshop** (Chat Application & API creation) <br> - Complete **Elastic Beanstalk Workshop**: Deploy Node.js apps with CDK Pipelines | 2026-05-20 | 2026-05-20 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Deploy high-availability **WordPress on AWS** architectures on Amazon EC2 <br> - Participate in **Amazon ECS Workshop**: Containerization with ECS and **AWS Fargate** | 2026-05-21 | 2026-05-22 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Participate in **Amazon EKS Workshop**: Getting Started with EKS and EKS Blueprints for CDK <br> - Review CI/CD for EKS and learn about Red Hat OpenShift Service on AWS (**ROSA**) | 2026-05-22 | 2026-05-22 | <https://cloudjourney.awsstudygroup.com/> |

### Week 8 Achievements:

* Full-Stack Serverless Storage Application: Built a complete Document Management System (DMS) featuring automated data handling via Amazon API Gateway and backend Lambda CRUD processes. Integrated AWS Amplify libraries on the frontend to manage file storage securely.

* Distributed Cloud Application Diagnostics: Implemented end-to-end performance tracing across serverless microservices using AWS X-Ray. Added tracing markers to Lambda execution blocks to pinpoint execution bottlenecks and track request context journeys across multiple AWS components.

* Serverless Container Fleet Management: Configured and launched scalable web microservices inside Amazon ECS using AWS Fargate. This setup runs containers directly on serverless infrastructure, removing the operational overhead of provisioning, patching, or scaling underlying EC2 cluster nodes.

* Enterprise Kubernetes Cluster Assembly: Completed advanced container management labs using the Amazon EKS Workshop. Programmatically deployed managed Kubernetes control planes using EKS Blueprints for AWS CDK, setting up core GitOps delivery pipelines to manage application workloads.

* Highly Available Enterprise Monolith Architecture: Engineered a resilient production configuration for WordPress on AWS. Used separated EC2 compute nodes across distinct Availability Zones, connected to a shared storage fabric and an RDS database backend to prevent single points of failure.