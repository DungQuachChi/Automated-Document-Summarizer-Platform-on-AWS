---
title: "Cloud Architect Quiz Game Day"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 4.1 </b> "
---

# Cloud Architect Quiz Game Day

### Event Objectives

- Team-based competitive quiz on cloud architecture knowledge, ranging from basic concepts to advanced, real-life scenario questions
- 8 teams competing head-to-head across rounds

### Speakers

- *(fill in)*

### Format

- Each round paired 2 teams against each other
- Each team had 2 one-time "powers" to use during the game:
  - **50/50**: if the answer is correct, the team gets half points for that question; if wrong, no points are deducted
  - **X2**: doubles the points for that question regardless of whether the answer is correct or wrong
- Winning team: **Ngũ Đại Hiệp**

### Key Highlights

- Questions spanned a wide difficulty range from basic service definitions to real-life architecture troubleshooting scenarios
- Several rounds had notable point swings near the end due to teams using the X2 power on high-value questions
- Sample question (real-life scenario, AWS Solutions Architect style):

  > A company hosted an e-commerce website on an Auto Scaling group of EC2 instances behind an Application Load Balancer. Illegitimate external requests were coming from multiple systems with frequently changing IP addresses, causing performance issues. Which option blocks these requests with minimal impact on legitimate traffic?
  >
  > A. Regular rule in AWS WAF associated with the ALB
  > B. AWS PrivateLink connection
  > C. Rate-based rule in AWS WAF associated with the ALB
  > D. Custom network ACL on the ALB's subnet

  **Answer: C** a rate-based WAF rule blocks based on request rate per IP rather than a fixed IP list, so it keeps working even as the offending IPs change, while legitimate traffic under the threshold is unaffected.

### Key Takeaways

- Reinforced the distinction between WAF regular rules (static match conditions) and rate-based rules (behavior/rate-based, adaptive to changing sources) a common point of confusion in scenario questions
- Reviewed why network-layer controls (NACLs, PrivateLink) aren't the right tool for filtering based on traffic *behavior* rather than fixed addresses/ports
- Competitive quiz format with risk/reward mechanics (50/50, X2) made revisiting AWS fundamentals more engaging than passive review

### Applying to Work

- Keep the WAF regular-rule vs. rate-based-rule distinction in mind for any future work securing the doc-summarizer project's public-facing endpoints
- Use this kind of scenario-question format as a personal review method for AWS certification prep or before technical interviews

### Event Experience

Attended a cloud architect quiz/game day as one of 8 competing teams. Questions ranged from basic to advanced real-life cloud architecture scenarios, with 50/50 and X2 point powers adding a competitive/strategic element each round. Ngũ Đại Hiệp won the event. The format was engaging, with several plot twists in the point standings near the end of each team matchup.