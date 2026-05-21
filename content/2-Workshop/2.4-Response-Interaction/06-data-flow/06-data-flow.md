---
title: "Approval data flow"
weight: 6
chapter: false
pre: " <b> 2.4.6. </b> "
---

# 2.4.6. Data Flow

The diagram below describes the approval loop from Step Functions' pause to execution completion:

```
Step Function: Send Telegram And Wait
        │
        ├─ Store task_token in DynamoDB (task_tokens)
        │
        └─ lambda_telegram_sender
               │
               ▼
         [Telegram Bot sends message to admin group]
               │
               ▼ (Admin reads remediation plan)
         [Admin presses APPROVE or REJECT]
               │
               ▼
         Telegram API → POST /webhook
               │
               ▼
         API Gateway → lambda_approval_handler
               │
               ├─ Authenticate via Cognito (telegram_id)
               ├─ Fetch task_token from DynamoDB
               │
               ├─ APPROVE: send_task_success → Step Function continues
               │                    ↓
               │             lambda_executor
               │                    ↓
               │         [SIMULATION] Execute action
               │                    ↓
               │         Write to DynamoDB + S3
               │                    ↓
               │         Execution: SUCCEEDED
               │
               └─ REJECT: send_task_failure → Execution: FAILED (rejected)
```
