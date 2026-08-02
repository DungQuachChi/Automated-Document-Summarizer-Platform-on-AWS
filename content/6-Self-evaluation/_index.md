---
title: "Self-Assessment"
date: 2026-08-02
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Self-Assessment

During my internship at **First Cloud Journey (FCJ)** from **April 2026** to **July 2026**, I had the opportunity to move from classroom knowledge to building and operating a real Cloud system. After five weeks of structured AWS study, I spent seven weeks building my graduation project, an AI-powered serverless document summarization platform, where I owned the backend, the Amazon Bedrock integration, and the entire infrastructure: Terraform, the CI/CD pipeline, monitoring, and security hardening.

To objectively reflect on my internship period, I would like to evaluate myself based on the following criteria:

| No. | Criteria | Rating | Comment |
|---|---|---|---|
| 1 | Knowledge | Average | I came to this project with lack of knowledge about AWS. I have done some basic project before join in FCAJ program where my teammate has guide me a lot and help me go throught many new knowledge.By the end, I could follow along and operate parts of a full serverless stack (Lambda, API Gateway, Cognito, DynamoDB, Bedrock, Terraform) |
| 2 | Ability to learn | Fair | Started from lack of knowladge I understand that I need to gain more as much as I can started our project, so the first few weeks were slow while I got comfortable with basic concepts like IAM, Lambda, and API Gateway. I could follow documentation and tutorials to get things working, but I often needed to revisit the same topic more than once before it stuck, and I still relied on teammates or guides for anything beyond the basics.|
| 3 | Proactiveness | Good | During the labs and project there are many challenge services I never touched before, configuration errors with no obvious cause, and gaps in my own understanding I noticed myself ,rather than waiting for help each time, I tried to research the issue first, work through the official docs, and attempt a fix before asking my teammate or mentor.|
| 4 | Discipline | Fair | I keep my discipline at first because I know I need to keep up with my team mate. I'd sometimes get distracted or pulled away by other things during the week, and ended up writing most entries in one sitting at the end of the week instead. This is something I want to fix in future projects by logging progress as I go, even briefly, rather than reconstructing it later.|
| 5 | Communication | Fair | In the first few weeks of working together, there was sometime we misunderstanding, partly because I leaned on asking for help too quickly instead of trying to work through the problem myself first. That something I want to improve solving my own problems before turning to my teammate, rather than making communication a substitute for problem-solving.|
| 6 | Teamwork | Good | Our two-person split (backend/infrastructure vs. frontend/reporting/load-testing) had clear ownership boundaries, and we integrated through PRs on a protected main branch. Handoffs, like connecting the finished frontend to the real API, went smoothly. |
| 7 | Problem-solving | Fair | 	Ran into a redirect_mismatch error when testing sign-up, and had to work through it in stages rather than solving it in one step. First I was on the wrong Cognito settings page entirely and had to find the correct "Login pages" tab under App clients to locate the callback URL setting. After updating the allowed callback and sign-out URLs to match my local port, the error persisted — it turned out the browser was sending http://0.0.0.0:8080 instead of http://localhost:8080 because of how I'd started the local server, which Cognito treats as a different origin. I fixed it by hardcoding the redirect URI in script.js and making sure to access the app via localhost instead of 0.0.0.0. It took several rounds of narrowing down the actual cause instead of getting it right the first time, which is something I want to get faster at. |
| 8 | Contribution to the project | Fair | Delivered the frontend layer, testing infrastructure, and documentation: built the complete static UI (HTML/CSS/JS) with OAuth 2.0 Cognito authentication flow, dark theme. Testing with Locust load testing script with Cognito token acquisition, authored the project README, workshop report and handled frontend deployment to S3/CloudFront. |

### Needs Improvement

* Strengthen discipline by logging progress in real time rather than backfilling the worklog at the end of the week, and hold myself to the same rules and standards I'd expect in a company or organizational setting
* Improve problem-solving thinking by narrowing down root causes more systematically instead of trying multiple fixes before identifying the actual issue
* Enhance communication skills by working through problems on my own first before asking for help, so that collaboration doesn't become a substitute for problem-solving — and by getting more comfortable presenting technical work clearly and concisely to others
