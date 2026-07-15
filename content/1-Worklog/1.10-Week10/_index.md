---
title: "Week 10 Worklog"
date: 2026-06-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:

* Finalize the Capstone Project proposal.
* Define the functional scope, components, and exclusions of the serverless architecture.
* Complete initial local environment preparations and set up cloud billing protection.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Research project topics and select the *Automated Document Summarizer Platform* concept <br> - Outline the primary architectural objectives | 2026-06-01 | 2026-06-01 | Internal FCAJ Guidelines |
| 3 | - Detail the architectural layers (User, Edge, Compute, AI, Data, Scheduling, Security) <br> - Document explicit component inclusions and exclusions | 2026-06-02 | 2026-06-02 | AWS Serverless Architectures |
| 4 | - Provision a dedicated, isolated AWS sandbox account for the capstone project <br> - Configure safety guardrails via **AWS Budgets** with a $25 threshold | 2026-06-03 | 2026-06-03 | AWS Billing Console |
| 5 | - Install core local engineering tools: Terraform (v1.5+), Git, and the AWS CLI tool <br> - Verify programmatic terminal access to the new AWS environment | 2026-06-04 | 2026-06-04 | <https://registry.terraform.io/> |
| 6 | - Complete the official project proposal documentation <br> - Map out the 13-phase project development lifecycle and submit to mentors | 2026-06-05 | 2026-06-05 | Project Proposal Docs |

### Week 10 Achievements:

* Project Concept Selection & Finalization: Authored a production-ready proposal for an *Automated Document Summarizer Platform on AWS*. Mapped out the serverless request path using API Gateway, Lambda, Cognito, and Bedrock.

* Proactive Cloud Budget Governance: Established strict financial guardrails inside the new sandbox account via AWS Budgets, configuring automated alerts at a $25 monthly ceiling to prevent unexpected billing runaways during load testing cycles.

* Local DevOps Toolchain Configuration: Installed and verified the local workspace engineering toolchain—binding Terraform (v1.5+), Git, and the AWS CLI together. Configured localized IAM administrative credentials using safe execution contexts.
