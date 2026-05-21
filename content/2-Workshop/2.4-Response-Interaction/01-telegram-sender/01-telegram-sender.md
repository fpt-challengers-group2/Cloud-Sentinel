---
title: "Telegram Bot & Lambda Sender"
weight: 1
chapter: false
pre: " <b> 2.4.1. </b> "
---

# 2.4.1. Telegram Bot and Lambda `lambda_telegram_sender`

**Function:** Receive Markdown remediation plans from the Advisor Agent and send approval requests to the admin group on Telegram.

**Environment variables:**

| Env var | Value | Source |
|---------|-------|--------|
| `TELEGRAM_TOKEN` | Bot token from BotFather | `terraform.tfvars` (sensitive) |
| `TELEGRAM_CHAT_ID` | Admin group Chat ID | `terraform.tfvars` (sensitive) |
| `STEP_FUNCTIONS_ARN` | State machine ARN | Terraform output |

**Telegram message structure:**

```
🚨 CLOUD SENTINEL – APPROVAL REQUEST

Finding ID : mock-finding-001
Type       : Recon:EC2/Portscan
Severity   : 5.0 (Medium)
Resource   : i-0123456789abcdef0
Attacker IP: 203.0.113.45 (Vietnam)

📋 REMEDIATION PLAN (summary):
• Proposed action: block_ip
• Block IP 203.0.113.45 at Network ACL
• Review Security Group rules
• ...

[✅ APPROVE]  [❌ REJECT]
```

The message uses a **Telegram Inline Keyboard** with two buttons:
- **APPROVE** – callback_data contains `approve:{task_token_hash}`
- **REJECT** – callback_data contains `reject:{task_token_hash}`

`task_token` is the Step Functions `waitForTaskToken`, temporarily stored in DynamoDB `task_tokens` table with 24-hour TTL.
