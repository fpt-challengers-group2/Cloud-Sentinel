---
title: "Data flow"
weight: 7
chapter: false
pre: " <b> 2.3.7. </b> "
---

# 2.3.7. Data Flow

The diagram below shows the data journey through the first 4 states of the orchestration layer:

```
[Mock Event / GuardDuty]
        │
        ▼
┌─────────────────────┐
│  Parse Finding      │  lambda_parser
│  → finding_details  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Check Precedent    │  lambda_history → DynamoDB
│  → historical_ctx   │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Get Knowledge      │  lambda_knowledge → Pinecone
│  → guidelines       │
└────────┬────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│  Invoke Agents Pipeline                  │
│  lambda_advisor:                         │
│    1. Supervisor Agent (agentic loop)    │
│       → structured JSON analysis         │
│    2. Advisor Agent (pure LLM)           │
│       → Markdown remediation plan        │
│  → Save report to S3                     │
└────────┬─────────────────────────────────┘
         │
         ▼
  [Send Telegram And Wait] → Layer 2.4
```
