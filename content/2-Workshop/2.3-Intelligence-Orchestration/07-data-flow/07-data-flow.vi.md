---
title: "Luồng dữ liệu"
weight: 7
chapter: false
pre: " <b> 2.3.7. </b> "
---

# 2.3.7. Luồng dữ liệu

Sơ đồ dưới đây mô tả hành trình của dữ liệu qua 4 state đầu tiên trong lớp điều phối:

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
│  → Lưu báo cáo vào S3                   │
└────────┬─────────────────────────────────┘
         │
         ▼
  [Send Telegram And Wait] → Lớp 2.4
```
