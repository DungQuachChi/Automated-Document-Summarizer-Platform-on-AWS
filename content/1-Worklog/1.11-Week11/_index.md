---
title: "Week 11 Worklog"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:

* Establish the codebase repository structure and version control pipelines.
* Design the detailed Terraform modular architecture and backend remote state configurations.
* Map out data access schemas and design the boilerplate application code structures.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2 | - Initialize a private GitHub repository for codebase tracking <br> - Configure branching strategies for infrastructure updates | 2026-06-08 | 2026-06-08 | GitHub Documentation |
| 3 | - Design the structural folder hierarchy for the upcoming modular **Terraform** configuration <br> - Draft the `backend.tf` remote state tracking strategy | 2026-06-09 | 2026-06-09 | Terraform Best Practices |
| 4 | - Map out the DynamoDB Single-Table schema design attributes <br> - Document partition key and sort key structures for history endpoints | 2026-06-10 | 2026-06-10 | Amazon DynamoDB Guide |
| 5 | - Create boilerplate directories for AWS Lambda source code files <br> - Research Bedrock Boto3 SDK parameters and request validation limits | 2026-06-11 | 2026-06-11 | AWS Boto3 SDK Docs |
| 6 | - Set up localized Python virtual environments (`venv`) <br> - Draft initial infrastructure configuration maps and verify dependencies | 2026-06-12 | 2026-06-12 | Python Virtual Environments |

### Week 11 Achievements:

* Version Control Blueprinting: Created the codebase baseline by establishing a secure private GitHub repository, configuring `.gitignore` matrices to protect local credentials, and organizing localized source paths.

* IaC Structural Architecture Mapping: Formulated a clean modular directory setup for Terraform (`auth`, `compute`, `api`, `data`). Engineered a secure deployment strategy utilizing S3 remote state tracking and DynamoDB state locks.

* Data Access Strategy Planning: Completed architectural sketches of the database access layers, planning data flows to support fast user queries and weekly reporting tasks before starting actual resource creation.