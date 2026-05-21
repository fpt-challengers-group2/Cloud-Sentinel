---
title: "Lab: Approve via Telegram"
weight: 7
chapter: false
pre: " <b> 2.4.7. </b> "
---

# 2.4.7. Lab Instructions

This lab continues from 2.3 where an execution is **Waiting** at `Send Telegram And Wait`. You'll observe the Telegram message, approve, and track execution results.

## Step 1: Configure Telegram Bot token (if not set)

Check Lambda environment for token:

```bash
aws lambda get-function-configuration \
  --function-name cloud-sentinel-lambda_telegram_sender \
  --region ap-southeast-1 \
  --query "Environment.Variables.TELEGRAM_TOKEN" \
  --output text
```

If empty, update `terraform.tfvars` with `telegram_token` and `telegram_chat_id`, then run `terraform apply`.

## Step 2: Ensure Cognito user has `telegram_id`

```bash
# Get User Pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
  --max-results 10 \
  --region ap-southeast-1 \
  --query "UserPools[?contains(Name,'cloud-sentinel')].Id" \
  --output text)

# Check security-admin attributes
aws cognito-idp admin-get-user \
  --user-pool-id $USER_POOL_ID \
  --username security-admin \
  --region ap-southeast-1 \
  --query "UserAttributes[?Name=='custom:telegram_id']"
```

If missing, update attribute:

```bash
aws cognito-idp admin-update-user-attributes \
  --user-pool-id $USER_POOL_ID \
  --username security-admin \
  --user-attributes Name="custom:telegram_id",Value="<your-telegram-id>" \
  --region ap-southeast-1
```

(Find your Telegram ID via `@userinfobot`.)

## Step 3: Verify webhook is registered

Get API ID and construct webhook URL, then set webhook:

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/setWebhook" \
  -d "url=<WEBHOOK_URL>"
```

Expected: `{"ok":true,"result":true,"description":"Webhook was set"}`.

## Step 4: Start execution and watch Telegram

If not started, run the execution from 2.3. The bot should send an approval message within ~30–90s.

## Step 5: Approve and observe

Press APPROVE. Then:

- In Step Functions: `Send Telegram And Wait` moves to Succeeded; `Execute Remediation` runs.
- Tail `lambda_executor` logs to see simulation output.
- Verify DynamoDB and S3 for incident record and report.

If REJECT is pressed, the execution fails with `AdminRejected` and no actions are executed.
