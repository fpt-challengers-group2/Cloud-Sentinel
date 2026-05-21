---
title: "Luồng dữ liệu phê duyệt"
weight: 6
chapter: false
pre: " <b> 2.4.6. </b> "
---

# 2.4.6. Luồng dữ liệu

Sơ đồ dưới đây mô tả toàn bộ vòng lặp phê duyệt từ khi Step Function tạm dừng đến khi execution hoàn thành:

```
Step Function: Send Telegram And Wait
        │
        ├─ Lưu task_token vào DynamoDB (task_tokens)
        │
        └─ lambda_telegram_sender
               │
               ▼
         [Telegram Bot gửi tin đến nhóm admin]
               │
               ▼ (Admin đọc kế hoạch ứng phó)
         [Admin bấm APPROVE hoặc REJECT]
               │
               ▼
         Telegram API → POST /webhook
               │
               ▼
         API Gateway → lambda_approval_handler
               │
               ├─ Xác thực Cognito (telegram_id)
               ├─ Lấy task_token từ DynamoDB
               │
               ├─ APPROVE: send_task_success → Step Function tiếp tục
               │                    ↓
               │             lambda_executor
               │                    ↓
               │         [SIMULATION] Thực thi hành động
               │                    ↓
               │         Ghi DynamoDB + S3
               │                    ↓
               │         Execution: SUCCEEDED
               │
               └─ REJECT: send_task_failure → Execution: FAILED (rejected)
```
