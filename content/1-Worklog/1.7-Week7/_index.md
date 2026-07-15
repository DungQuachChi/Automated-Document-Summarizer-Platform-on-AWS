---
title: "Week 7 Worklog"
date: 2026-05-11
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---


### Week 7 Objectives:

* Master application modernization architectures through hands-on serverless workshops.
* Complete the DevAx and Serverless Book Store development series.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - **Serverless - DevAx Series:** Learn Monolith to Microservices Migration strategies <br> - Build microservices, set up CI/CD for release, and manage data restructuring | 2026-05-11 | 2026-05-11 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Serverless - DevAx Series (Cont.):** Implement Event-Driven Architectures <br> - Configure Single Page Application Authentication & integrate AWS AI Services | 2026-05-12 | 2026-05-12 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **Serverless - Book Store Series:** Set up serverless backend with Lambda, S3, and DynamoDB <br> - Develop frontends for Serverless APIs and automate deployment with **AWS SAM** | 2026-05-13 | 2026-05-13 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **Serverless - Book Store Series (Cont.):** Implement user login via **Amazon Cognito** <br> - Configure Custom Domains, SSL certificates, and Event Processing via SQS/SNS | 2026-05-14 | 2026-05-15 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Serverless - Book Store Series (Cont.):** Build out GraphQL APIs with **AWS AppSync** <br> - Finalize CI/CD and monitoring configurations for serverless applications | 2026-05-15 | 2026-05-15 | <https://cloudjourney.awsstudygroup.com/> |


### Week 7 Achievements:

* Monolith Refactoring Modernization: Analyzed legacy multi-tier web codebases to break down monolithic setups into decoupled microservices, implementing structural patterns to successfully isolate separate databases.

* Serverless Application Framework Delivery: Used the AWS Serverless Application Model (AWS SAM) to build and package a complete application stack. Authored structured `template.yaml` definitions to deploy AWS Lambda layers, S3 asset buckets, and DynamoDB storage instances using unified CloudFormation execution calls.

* Identity Management & Access Security: Secured a Single Page Application (SPA) frontend by implementing Amazon Cognito User Pools. This added secure, token-based registration and authentication workflows, complete with custom domain routing and managed SSL/TLS layer validations.

* Managed GraphQL API Layer Synthesis: Deployed managed, real-time data endpoints using AWS AppSync. Built efficient GraphQL resolvers that securely read and write data to Amazon DynamoDB backends, eliminating the need to manage traditional REST endpoint controllers.

* Event-Driven Event Pipeline Processing: Connected asynchronous processing logic across components using decoupled SQS and SNS queues. This setup captures transactional events in real-time, safely managing spikes in application traffic.