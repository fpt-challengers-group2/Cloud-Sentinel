---
title: "API Gateway Webhook"
weight: 2
chapter: false
pre: " <b> 2.4.2. </b> "
---

# 2.4.2. API Gateway – Webhook endpoint

API Gateway exposes a public HTTP endpoint so Telegram can send callbacks when admins press buttons.

| Property | Value |
|----------|-------|
| Endpoint | `POST /webhook` |
| Integration | Lambda `lambda_approval_handler` |
| Auth | None (Telegram authenticates using a secret token header) |
| Stage | `prod` |

**Webhook processing flow:**

1. Admin presses **Approve** or **Reject** in Telegram.
2. Telegram sends `POST /webhook` with a payload containing `callback_query`.
3. API Gateway forwards to `lambda_approval_handler`.
4. Lambda processes the callback:
   - If Approve: update DynamoDB, call Step Function to continue pipeline.
   - If Reject: update DynamoDB, send rejection response to Telegram.
