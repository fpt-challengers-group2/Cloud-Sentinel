---
title: "Những gì đã xây dựng"
weight: 1
chapter: false
pre: " <b> 2.5.1. </b> "
---

# 2.5.1.1. Những gì đã xây dựng

Triển khai một hệ thống SOAR hoàn chỉnh trên AWS với kiến trúc Serverless thuần túy:

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
  │       └─ [Admin phê duyệt] → API Gateway → lambda_approval_handler ↔ Cognito
  └─ Execute Remediation   → lambda_executor   ↔  DynamoDB + S3
```

**Tổng số tài nguyên AWS đã triển khai qua Terraform:**

| Loại tài nguyên | Số lượng | Tên / Ghi chú |
|----------------|----------|---------------|
| Lambda Functions | 8 | parser, history, knowledge, advisor, telegram_sender, approval_handler, executor, token_cleaner |
| Step Functions | 1 | `cloud-sentinel-orchestrator` |
| Bedrock Agents | 2 | Supervisor Agent, Advisor Agent |
| DynamoDB Tables | 2 | `incident_history`, `task_tokens` |
| S3 Buckets | 1 | `cloud-sentinel-reports-{account_id}` |
| API Gateway | 1 | REST API với endpoint `/webhook` |
| Cognito User Pool | 1 | `cloud-sentinel-admin-pool` |
| IAM Roles | 8+ | Mỗi Lambda có role riêng, theo nguyên tắc least privilege |
| CloudWatch Log Groups | 8+ | Retention 30 ngày cho mỗi Lambda |
| Telegram Bot | 1 | Kết nối qua BotFather, sử dụng token trong Lambda |