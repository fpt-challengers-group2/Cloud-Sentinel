---
title: "Response & Interaction Layer"
weight: 4
chapter: false
pre: " <b> 2.4. </b> "
---

# 2.4. Response & Interaction Layer – Telegram, Cognito, API Gateway & Executor

## Overview

The Response Layer is the bridge between the automated system and humans. Instead of executing remediation fully automatically (which risks production infrastructure), CloudSentinel uses a **Human-in-the-Loop** model: the system analyzes and proposes, and security operators review and decide.

Core components:

| Component | AWS / Third-party | Role |
|-----------|-------------------|------|
| Telegram Bot | Telegram API | Alert channel and approval interface |
| Sender | Lambda `lambda_telegram_sender` | Format and send messages |
| Webhook | API Gateway `/webhook` (POST) | Receive callbacks from Telegram |
| Approval Handler | Lambda `lambda_approval_handler` | Authenticate admin, release Step Function |
| Auth | Amazon Cognito User Pool | Manage admin approvers |
| Executor | Lambda `lambda_executor` | Execute remediation actions (simulation) |

Detailed contents:
- [Telegram Bot & Sender](./01-telegram-sender/01-telegram-sender.md)
- [API Gateway Webhook](./02-api-gateway/02-api-gateway.md)
- [Cognito Admin Auth](./03-cognito-auth/03-cognito-auth.md)
- [Lambda Approval Handler](./04-approval-handler/04-approval-handler.md)
- [Lambda Executor](./05-executor/05-executor.md)
- [Approval data flow](./06-data-flow/06-data-flow.md)
- [Lab: Approve via Telegram](./07-lab-approval.md)
- [Verification](./08-verification/08-verification.md)
- [Summary](./09-summary.md)
