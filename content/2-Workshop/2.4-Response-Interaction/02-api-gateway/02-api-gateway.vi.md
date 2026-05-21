---
title: "API Gateway Webhook"
weight: 2
chapter: false
pre: " <b> 2.4.2. </b> "
---

# 2.4.2. API Gateway – Webhook endpoint

API Gateway tạo một HTTP endpoint công khai để Telegram có thể gửi callback khi admin bấm nút.

| Thuộc tính | Giá trị |
|------------|---------|
| Endpoint | `POST /webhook` |
| Integration | Lambda `lambda_approval_handler` |
| Auth | Không có (Telegram tự xác thực qua secret token header) |
| Stage | `prod` |

**Luồng xử lý webhook:**

1. Admin bấm **Approve** hoặc **Reject** trên Telegram.
2. Telegram gửi `POST /webhook` với payload chứa `callback_query`.
3. API Gateway forward ngay đến `lambda_approval_handler`.
4. Lambda xử lý callback:
   - Nếu Approve: Cập nhật DynamoDB, gọi Step Function để tiếp tục pipeline.
   - Nếu Reject: Cập nhật DynamoDB, gửi phản hồi từ chối đến Telegram.