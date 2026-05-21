---
title: "Proposal"
weight: 1
chapter: false
---
title: "Proposal"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
<img src="/images/logo.jpg" class="img-responsive" style="max-width:300px; display:block; margin:auto;">

# AWS FIRST CLOUD AI JOURNEY – PROJECT PLAN
# Cloud-Sentinel – SOAR System for Critical Infrastructure on AWS

---

## 0. APPENDIX

- [AWS FIRST CLOUD AI JOURNEY – PROJECT PLAN](#aws-first-cloud-ai-journey--project-plan)
- [Cloud-Sentinel – SOAR System for Critical Infrastructure on AWS](#cloud-sentinel--soar-system-for-critical-infrastructure-on-aws)
  - [0. APPENDIX](#0-appendix)
  - [1. CONTEXT \& DRIVERS](#1-context--drivers)
    - [1.1 Summary](#11-summary)
      - [1. Customer context \& Problem](#1-customer-context--problem)
      - [2. Business \& Technical Objectives](#2-business--technical-objectives)
      - [3. Use Cases](#3-use-cases)
      - [4. Consulting summary](#4-consulting-summary)
    - [1.2 Success criteria](#12-success-criteria)
    - [1.3 Assumptions and constraints](#13-assumptions-and-constraints)
  - [2. SOLUTION ARCHITECTURE](#2-solution-architecture)
    - [2.1 High-level architecture](#21-high-level-architecture)
    - [2.2 Technical approach](#22-technical-approach)
    - [2.3 Project plan (high level)](#23-project-plan-high-level)
    - [2.4 Security considerations](#24-security-considerations)
  - [3. ACTIVITIES AND DELIVERABLES](#3-activities-and-deliverables)
  - [4. COST ANALYSIS BY SERVICE](#4-cost-analysis-by-service)
  - [5. RESOURCES AND ESTIMATES](#5-resources-and-estimates)
  - [6. ACCEPTANCE](#6-acceptance)

---

## 1. CONTEXT & DRIVERS

### 1.1 Summary

#### 1. Customer context & Problem

Organizations that operate critical infrastructure on AWS (banks, financial institutions, and fintechs) face a growing volume of security alerts from sources such as GuardDuty, CloudTrail, and VPC Flow Logs. Current analyst workflows are mostly manual: each finding must be read, historical context reviewed, a remediation plan drafted, and actions executed. This leads to slow response times, missed incidents, and heavy operational burden. High false positive rates further reduce monitoring effectiveness and trust.

There is an urgent need for a SOAR (Security Orchestration, Automation and Response) solution that automates the end-to-end detection-to-response lifecycle while using AI to reduce false positives and improve analysis quality—keeping humans as final decision-makers.

#### 2. Business & Technical Objectives

| Objective Type | Details |
|---|---|
| Business | - Reduce Mean Time to Respond (MTTR) from hours to minutes.
  - Reduce operational load on security teams through automated analysis and action proposals.
  - Minimize operating costs: serverless design yields near-zero cost when idle. |
| Technical | - Smart detection: ingest GuardDuty findings, VPC Flow Logs, DNS logs and CloudTrail events.
  - Multi-agent cross-evaluation: Supervisor and Advisor agents (Amazon Bedrock) collaborate to lower false positives and produce structured remediation plans.
  - RAG knowledge base: Pinecone vector index provides grounding to reduce LLM hallucinations.
  - Human-in-the-loop: operator approval (Telegram) required before any enforcement action.
  - Full IaC: Terraform-managed infrastructure for repeatability and control. |

#### 3. Use Cases

Primary flows handled by Cloud-Sentinel:

- Automated detection & analysis: when GuardDuty emits a finding (e.g., SSH brute force, port scan, IAM anomaly), the pipeline triggers analysis automatically.
- Precedent lookup: query DynamoDB for prior incidents affecting the same resource within 90 days to provide historical context.
- Structured remediation planning: the Advisor agent produces a 7-part remediation plan (immediate actions, investigation steps, long-term recommendations, rollback plan, etc.).
- Approval via Telegram: operators receive the plan and can Approve/Reject from mobile.
- Action execution: upon approval, the system executes protective measures (block IP, isolate instance, revoke IAM keys) and records results in S3 and DynamoDB.

#### 4. Consulting summary

Professional services delivered to achieve the objectives include:

- Design and implement an event-driven SOAR architecture (6‑state Step Functions pipeline) integrated with multi-agent AI on Amazon Bedrock.
- Define and deploy all infrastructure as Terraform modules (Lambda, Step Functions, Pinecone, S3, DynamoDB, API Gateway, Cognito, etc.).
- Implement a RAG pipeline (embedding with Titan, store/query via Pinecone) to ground agent prompts and reduce hallucination.
- Optimize serverless cost and operational model for pay-as-you-go operation.

---

### 1.2 Success criteria

Key acceptance criteria for the project:

- End-to-end pipeline: a mock finding triggers the Step Function, sends the Telegram approval message, receives approval, executes actions (simulation or real), and writes results to S3/DynamoDB without errors.
- Multi-agent output: Supervisor and Advisor agents consistently produce structured remediation plans covering the required sections.
- Human-in-the-loop: Telegram approval workflow functions and correctly advances or aborts the Step Function.
- Cost optimization: operational cost remains within target (example estimates provided in Section 4).
- Infrastructure as code: all resources can be created and destroyed via `terraform apply` / `terraform destroy`.

---

### 1.3 Assumptions and constraints

- Project depends on availability of Amazon Bedrock, GuardDuty, Step Functions, and other AWS services in `ap-southeast-1`.
- Bedrock model access (e.g., `claude-3-5-sonnet`) must be enabled in the account before running the pipeline.
- Pinecone index can be empty initially; agent performance improves as guideline data is seeded.
- The Lambda executor operates in simulation mode by default for workshop/demo safety.
- Current triggers are mock events; production should wire GuardDuty → EventBridge for real triggers.
- LLM outputs may still hallucinate; RAG grounding and human approval mitigate the risk.

---

## 2. SOLUTION ARCHITECTURE

### 2.1 High-level architecture

Serverless, event-driven architecture on AWS with four logical layers: Detection, Intelligence & Orchestration, Response & Interaction, and Storage.

Key components:

- Detection: Amazon GuardDuty generates findings; a `lambda_parser` normalizes events.
- Orchestration: AWS Step Functions (6 states) orchestrate the pipeline from parse → analyze → wait for approval → execute.
- Intelligence: `lambda_history` (DynamoDB), `lambda_knowledge` (Pinecone RAG), `lambda_advisor` invoking Supervisor and Advisor agents on Bedrock.
- Response & Interaction: `lambda_telegram_sender` posts the plan to Telegram; API Gateway receives callbacks; `lambda_approval_handler` validates approvals via Cognito and resumes the Step Function.
- Execution: `lambda_executor` applies remediation (simulation mode by default) and records results in `incident_history` (DynamoDB) and S3.

### 2.2 Technical approach

- Infrastructure as Code: Terraform modules for Lambdas, Step Functions, Cognito, API Gateway, DynamoDB, S3, and Pinecone integration. Use an S3 backend for Terraform state.
- Multi-agent workflow: Supervisor agent runs data collection actions (parser, history, knowledge). Advisor agent receives structured JSON and synthesizes the remediation plan.
- RAG flow: guideline texts → Titan embeddings → Pinecone upsert. On incident: finding → embedding → Pinecone top-k → include results in agent prompt.
- Human-in-the-loop: Step Functions uses `waitForTaskToken`; task tokens persist in `task_tokens` table with TTL; `lambda_approval_handler` calls `send_task_success` or `send_task_failure`.

### 2.3 Project plan (high level)

- Phase 1 (Weeks 1–2): Core pipeline, Terraform skeleton, Lambdas `parser`, `history`, `knowledge`, Step Functions initial states.
- Phase 2 (Weeks 3–4): Agent integration, Pinecone index, `lambda_advisor` and agentic loop testing.
- Phase 3 (Weeks 5–6): Telegram integration, API Gateway webhook, Cognito, `lambda_executor` (simulation → optional real execution), end-to-end testing.

### 2.4 Security considerations

- IAM least-privilege: each Lambda role scoped to required resources.
- Secrets management: sensitive values injected via Terraform variables (not committed) and stored in environment variables or Secrets Manager.
- Cognito-based admin mapping ensures only registered admins can approve via Telegram.
- API Gateway and webhook validation performed in `lambda_approval_handler`.
- CloudWatch logging and retention policies established; S3 report buckets are non-public.

---

## 3. ACTIVITIES AND DELIVERABLES

[Sections and delivery tables retained as in source; detailed deliverables, timelines and out-of-scope items mirrored from the Vietnamese source.]

---

## 4. COST ANALYSIS BY SERVICE

[Cost table and notes preserved; example total around ~$14.46/month for demo usage — exact values in source document.]

---

## 5. RESOURCES AND ESTIMATES

[Personnel rates, estimated hours, and phase cost breakdown preserved from the source document.]

---

## 6. ACCEPTANCE

[Acceptance criteria, deliverables, UAT and rejection conditions preserved from the source document.]

