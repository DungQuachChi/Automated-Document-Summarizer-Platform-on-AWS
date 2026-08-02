---
title: "Week 13 Worklog"
date: 2026-06-22
weight: 13
chapter: false
pre: " <b> 1.13. </b> "
---

### Week 13 Objectives:
* Expand document processing capabilities to support real-world file formats (PDF/DOCX) using Amazon S3 Event Triggers and AWS Lambda Layers.
* Implement a comprehensive system monitoring, performance analysis, and distributed tracing solution using **AWS X-Ray** and **CloudWatch Alarms**.
* Develop the Web Frontend User Interface and deploy it statically via **Amazon S3** & **Amazon CloudFront CDN**.

### Daily Tasks:
| Day | Task | Start Date | Completion Date | References |
| --- | --- | --- | --- | --- |
| Mon | - Create an S3 Bucket for input document storage (documents-upload-bucket) <br> Package an **AWS Lambda Layer** containing text extraction libraries (pypdf / python-docx) | 2026-06-22 | 2026-06-22 | AWS Lambda Layers Guide |
| Tue | Program an asynchronous S3 Event Notification pipeline to trigger Lambda for document extraction <br> Convert uploaded attachment formats into raw string text before passing to Amazon Bedrock | 2026-06-23 | 2026-06-23 | Amazon S3 Event Notifications |
| Wed | Enable **AWS X-Ray Tracing** across API Gateway, Lambda, and DynamoDB <br> Configure **CloudWatch Alarms** to monitor HTTP 5xx errors and AI model latency thresholds | 2026-06-24 | 2026-06-24 | AWS X-Ray Developer Guide |
| Thu | Build a responsive Web UI (HTML5/TailwindCSS/JS ES6) featuring File Uploads, Cognito Authentication, and summary output rendering <br> Configure Cross-Origin Resource Sharing (CORS) on API Gateway | 2026-06-25 | 2026-06-25 | Amazon API Gateway CORS |
| Fri | Deploy Frontend assets to S3 Static Website Hosting integrated with **Amazon CloudFront** <br> Conduct End-to-End (E2E) testing (User Sign-up -> Login -> File Upload -> AI Summary -> History Retrieval) | 2026-06-26 | 2026-06-26 | Amazon CloudFront Docs |

### Key Achievements:
* **Lambda Layer & Advanced Document Extraction:** Successfully packaged a custom Lambda Layer containing pypdf and python-docx to extend summary capabilities from plain text to uploaded PDF/DOCX files.
* **Automated Async File Processing Pipeline:** Configured Amazon S3 Event Notifications to automatically trigger processing Lambda functions upon file upload, safely extracting raw text before pushing to the Bedrock LLM workflow.
* **Distributed System Observability:** Activated **AWS X-Ray** across the request path (API Gateway -> Lambda -> Bedrock -> DynamoDB) to generate Service Maps and trace latency. Configured CloudWatch Alarms to issue alerts if backend error rates exceed 5%.
* **Frontend Web Application & Authentication Security:** Built a responsive Web UI that enables users to authenticate via Cognito Hosted UI, secure ID Tokens, upload files directly, and view saved summary histories.
* **Global Content Delivery via CloudFront:** Distributed Frontend assets using **Amazon CloudFront** paired with an S3 Bucket Origin, configuring CORS rules on API Gateway to guarantee secure cross-domain communication.
* **End-to-End (E2E) Integration Testing:** Completed comprehensive testing from user sign-up to PDF upload, receiving Claude 3 Haiku summaries in under 3 seconds and querying historical records from DynamoDB.