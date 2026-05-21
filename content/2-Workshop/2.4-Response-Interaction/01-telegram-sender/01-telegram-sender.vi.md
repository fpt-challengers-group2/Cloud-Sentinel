---
title: "Telegram Bot & Lambda Sender"
weight: 1
chapter: false
pre: " <b> 2.4.1. </b> "
---

# 2.4.1. Telegram Bot và Lambda `lambda_telegram_sender`

**Chức năng:** Nhận kế hoạch ứng phó Markdown từ Advisor Agent và gửi tin nhắn phê duyệt tới nhóm admin trên Telegram.

**Cấu hình:**

| Biến môi trường | Giá trị | Nguồn |
|-----------------|---------|-------|
| `TELEGRAM_TOKEN` | Bot token từ BotFather | `terraform.tfvars` (sensitive) |
| `TELEGRAM_CHAT_ID` | Chat ID của nhóm admin | `terraform.tfvars` (sensitive) |
| `STEP_FUNCTIONS_ARN` | ARN của state machine | Tự động từ Terraform output |

**Cấu trúc tin nhắn gửi đến Telegram:**

```
🚨 CLOUD SENTINEL – YÊU CẦU PHÊ DUYỆT

Finding ID : mock-finding-001
Loại sự cố : Recon:EC2/Portscan
Mức độ     : 5.0 (Medium)
Tài nguyên : i-0123456789abcdef0
IP tấn công: 203.0.113.45 (Vietnam)

📋 KẾ HOẠCH ỨNG PHÓ (tóm tắt):
• Hành động đề xuất: block_ip
• Block IP 203.0.113.45 tại Network ACL
• Review Security Group rules
• ...

[✅ APPROVE]  [❌ REJECT]
```

Tin nhắn sử dụng **Telegram Inline Keyboard** với hai nút:
- **APPROVE** – callback_data chứa `approve:{task_token_hash}`
- **REJECT** – callback_data chứa `reject:{task_token_hash}`

`task_token` là token `waitForTaskToken` của Step Functions, được lưu tạm trong bảng DynamoDB `task_tokens` với TTL 24 giờ.
