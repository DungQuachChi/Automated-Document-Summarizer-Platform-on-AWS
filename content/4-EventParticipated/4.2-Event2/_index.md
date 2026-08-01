---
title: "AgentForge Deepdive - Day 1"
date: 2026-08-01
weight: 1
chapter: false
pre: " <b> 4.2 </b> "
---

# AgentForge - Building Production-Ready Agentic Systems Using Amazon Bedrock AgentCore (Day 1)

### Event Objectives

- Theory: Introduction to Amazon Bedrock AgentCore L300 (Runtime, Gateway, Identity)
- Hands-on lab: Build & Deploy AI Agents on Amazon Bedrock AgentCore using Vibe Coding with Kiro
  - Deploy a basic agent in AgentCore
  - Connect to external tools and knowledge bases
  - Add a web UI with Cognito authentication

### Speakers

- **Nghia Tran** – Agentic SA
- **Anh Pham** – Cloud Consultant, G-AsiaPacific Vietnam


#### What MCP and A2A solve

- **A2A (Agent2Agent)**: protocol for agent-to-agent communication
- **MCP (Model Context Protocol)**: standard protocol for agents to reach tools/data (Slack, GitLab, S3, Jira, internal APIs) without custom integration code per tool

#### Strands Agents SDK

- Open-source SDK for building agents with minimal code
- Core loop: prompt → agent → invoke model (get reasoning/tool selection) → execute tool → return result → final response
- Native tool + MCP support, integrates with AWS services, supports custom model providers

#### Amazon Bedrock AgentCore — platform overview

- Three pillars: ship agents fast, connect to anything, optimize continuously
- Underlying principles: security at scale, enterprise readiness, deterministic control

#### AgentCore Runtime

- Secure, serverless runtime for deploying/scaling agents and tools (e.g. MCP servers), independent of framework/protocol/model
- Package as Docker image (up to 2GB, via ECR) or zip file (up to 250MB compressed / 750MB uncompressed, via S3)
- Endpoints and versions: agent versions can be created/updated independently of which endpoint (e.g. DEFAULT, PROD) points at them
- True session isolation: each session runs in its own microVM (Firecracker) — separate compute, memory, filesystem per session
- Supports async/long-running background tasks and bi-directional audio+text streaming (e.g. Nova Sonic 2, Google Live API, OpenAI Realtime API)

#### AgentCore Identity

- Handles inbound auth (user → app → agent) and outbound auth (agent → tools/vault)
- Four parts: Workload Identities, Credential Providers, Token Vault, Broker Logic
- Client secrets never leave the vault and never reach agent code or the LLM

#### AgentCore Gateway

- Single entry point connecting multiple agents to multiple downstream APIs/tools/resources
- Built-in: MCP support, tool creation/search, authorization, fine-grained access control, private connectivity, tools filtering
- Backed by AgentCore Identity (auth) and CloudWatch (observability)
- Supports secure private inbound connectivity (PrivateLink) for clients on separate VPCs or corporate networks

#### Hands-on lab (vibe coding with Kiro)

- Kiro: AI-powered IDE that generates code from natural-language descriptions (vibe coding) — no manual coding
- Lab built a Returns & Refunds agent in stages: base agent → persistent memory → Gateway/Lambda/Cognito auth → Runtime deployment → observability → evaluations → policies
- Progress today: completed Lab 1 (Kiro setup/features) and part of Lab 2 (through roughly agent build + memory); Gateway, web UI/Cognito, observability, evaluations, and policies not yet reached

### Key Takeaways

- AgentCore separates concerns cleanly: Runtime (execution/scaling), Gateway (tool/API access), Identity (auth) — each independently configurable
- Session isolation via microVMs is a real architectural guarantee, not just a marketing claim — relevant when handling any per-user state
- Vibe coding (Kiro) shifts effort from writing infrastructure code to describing intent precisely; the underlying AWS resources (Lambda, Cognito, IAM) still get created and still need to be understood for debugging and cost control
- Async/background task patterns matter for any agent that calls slow tools or runs multi-step workflows

### Applying to Work

- Review whether [[doc-summarizer]]'s current direct Lambda/Bedrock calls would benefit from AgentCore Gateway if the project ever needs to call multiple external tools/APIs through one managed layer
- Compare current auth handling in the project against AgentCore Identity's model (short-lived workload tokens, secrets never touching application code)
- Revisit session/state handling in the project's Lambda functions against the Runtime session-isolation model
- Continue the lab in a follow-up session to reach Gateway, Cognito web UI, and observability parts

### Event Experience

Attended AgentForge Deepdive Day 1, covering Amazon Bedrock AgentCore theory (Runtime, Gateway, Identity) and a hands-on lab building a Returns & Refunds agent through vibe coding with Kiro. Completed Lab 1 and part of Lab 2; remaining lab parts (Gateway, Cognito web UI, observability, evaluations, policies) are still to do.