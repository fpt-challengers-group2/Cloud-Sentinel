---
title: "Lambda Approval Handler"
weight: 4
chapter: false
pre: " <b> 2.4.4. </b> "
---

# 2.4.4. Lambda `lambda_approval_handler`

**Chức năng:** Nhận callback từ Telegram, xác thực admin, lấy task token từ DynamoDB, và gửi tín hiệu tiếp tục hoặc dừng tới Step Functions.

**Luồng xử lý:**

```
Nhận callback_query từ API Gateway
        │
        ▼
Trích xuất telegram_id + callback_data
        │
        ▼
Xác thực với Cognito (telegram_id có trong User Pool?)
        │
        ├─ Không hợp lệ → Return 403
        │
        └─ Hợp lệ
               │
               ▼
        Lấy task_token từ DynamoDB (PK = finding_id)
               │
               ▼
        ┌─ Approve ──→ step_functions.send_task_success(taskToken, output)
        └─ Reject  ──→ step_functions.send_task_failure(taskToken, cause)
               │
               ▼
        Xóa token khỏi DynamoDB (tránh dùng lại)
               │
               ▼
        Return 200 OK cho Telegram
```

IAM permissions của Lambda này bao gồm: `states:SendTaskSuccess`, `states:SendTaskFailure`, `dynamodb:GetItem`, `dynamodb:DeleteItem`, `cognito-idp:ListUsers`.