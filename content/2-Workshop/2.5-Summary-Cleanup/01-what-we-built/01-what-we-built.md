---
title: "What We Built"
weight: 1
chapter: false
pre: " <b> 2.5.1. </b> "
---

# 2.5.1.1. What We Built

We deployed a complete SOAR system on AWS using a fully Serverless architecture:

```
[Amazon GuardDuty]
       │  finding (mock event)
       ▼
[AWS Step Functions – 6 states]
  │
  ├─ Parse Finding         → lambda_parser
  ├─ Check Precedent       → lambda_history    ↔  DynamoDB (incident_history)
  ├─ Get Knowledge         → lambda_knowledge  ↔  Pinecone (RAG)
  ├─ Invoke Agents         → lambda_advisor
  │       ├─ Supervisor Agent (Bedrock – Claude 3.5 Sonnet)
  │       └─ Advisor Agent  (Bedrock – Claude 3.5 Sonnet)
  ├─ Send Telegram & Wait  → lambda_telegram_sender  → Telegram Bot
  │       └─ [Admin approval] → API Gateway → lambda_approval_handler ↔ Cognito
  └─ Execute Remediation   → lambda_executor   ↔  DynamoDB + S3
```

**Total AWS resources deployed via Terraform:**

- **Lambda Functions:** 8 — parser, history, knowledge, advisor, telegram_sender, approval_handler, executor, token_cleaner
- **Step Functions:** 1 — `cloud-sentinel-orchestrator`
- **Bedrock Agents:** 2 — Supervisor Agent, Advisor Agent
- **DynamoDB Tables:** 2 — `incident_history`, `task_tokens`
- **S3 Buckets:** 1 — `cloud-sentinel-reports-{account_id}`
- **API Gateway:** 1 — REST API with `/webhook` endpoint
- **Cognito User Pool:** 1 — `cloud-sentinel-admin-pool`
- **IAM Roles:** 8+ — each Lambda has its own role, following least privilege
- **CloudWatch Log Groups:** 8+ — 30-day retention for each Lambda
- **Telegram Bot:** 1 — connected via BotFather, token used in Lambda
