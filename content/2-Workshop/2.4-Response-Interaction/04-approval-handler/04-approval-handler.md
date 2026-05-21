---
title: "Lambda Approval Handler"
weight: 4
chapter: false
pre: " <b> 2.4.4. </b> "
---

# 2.4.4. Lambda `lambda_approval_handler`

**Function:** Receive callbacks from Telegram, authenticate the admin, fetch the task token from DynamoDB, and signal Step Functions to continue or stop.

**Processing flow:**

```
Receive callback_query from API Gateway
        │
        ▼
Extract telegram_id + callback_data
        │
        ▼
Authenticate with Cognito (is telegram_id in User Pool?)
        │
        ├─ Invalid → Return 403
        │
        └─ Valid
               │
               ▼
        Fetch task_token from DynamoDB (PK = finding_id)
               │
               ▼
        ┌─ Approve ──→ step_functions.send_task_success(taskToken, output)
        └─ Reject  ──→ step_functions.send_task_failure(taskToken, cause)
               │
               ▼
        Delete token from DynamoDB (prevent reuse)
               │
               ▼
        Return 200 OK to Telegram
```

IAM permissions for this Lambda include: `states:SendTaskSuccess`, `states:SendTaskFailure`, `dynamodb:GetItem`, `dynamodb:DeleteItem`, `cognito-idp:ListUsers`.
