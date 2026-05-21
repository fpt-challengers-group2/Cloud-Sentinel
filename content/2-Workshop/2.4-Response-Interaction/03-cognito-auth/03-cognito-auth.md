---
title: "Cognito Admin Auth"
weight: 3
chapter: false
pre: " <b> 2.4.3. </b> "
---

# 2.4.3. Amazon Cognito – Admin Authentication

Cognito ensures only registered admins can approve actions. When `lambda_approval_handler` receives a Telegram callback, it extracts `telegram_id` from the payload and checks the Cognito User Pool before proceeding.

**Cognito configuration:**

| Property | Value |
|----------|-------|
| User Pool name | `cloud-sentinel-admin-pool` |
| Additional schema | `custom:telegram_id` (String, mutable) |
| Default user | `security-admin` |
| Mapped Telegram ID | `8320828979` |

**Authentication flow in `lambda_approval_handler`:**

1. Extract `telegram_id` from `callback_query.from.id` in the Telegram payload.
2. Call Cognito `list_users` with filter `custom:telegram_id = <telegram_id>`.
3. If a user is found → authentication succeeds, proceed to process the token.
4. If not found → return `403 Forbidden`, ignore the callback.

> **Design rationale:** Using Cognito instead of hardcoding admin lists makes it easy to add/remove admins without redeploying Lambdas. Simply update the `custom:telegram_id` attribute for users in the Cognito Console.
