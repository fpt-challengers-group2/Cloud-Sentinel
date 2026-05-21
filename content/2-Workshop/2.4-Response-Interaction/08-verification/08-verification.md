---
title: "Verification"
weight: 8
chapter: false
pre: " <b> 2.4.8. </b> "
---

# 2.4.8. Verification

## Check execution status

```bash
# List recent executions
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:5].{Name:name,Status:status,Start:startDate}" \
  --output table
```

## Check task_tokens cleared

After execution completes (Approve or Reject), tokens should be removed from `task_tokens` table:

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count"
```

Expected: `0` (table empty after processing).

## Check API Gateway logs

```bash
# Tail API Gateway access logs (if enabled)
aws logs tail /aws/apigateway/cloud-sentinel \
  --region ap-southeast-1 \
  --since 30m \
  --format short
```

## Troubleshooting – Common issues

| Symptom | Possible cause | Remediation |
|---------|----------------|-------------|
| No Telegram messages | Incorrect bot token or webhook not registered | Verify `TELEGRAM_TOKEN` in Lambda env vars; run `setWebhook` again |
| Approve pressed but Step Function still Waiting | Incorrect webhook URL or API Gateway not deployed | Check webhook with `getWebhookInfo`; confirm API Gateway `prod` stage deployed |
| `approval_handler` returns 403 | `telegram_id` not mapped in Cognito | Run `admin-update-user-attributes` with correct telegram ID |
| `lambda_executor` permission error | IAM role missing DynamoDB/S3 permissions | Check CloudWatch logs; ensure Terraform applied IAM policies |
| Task token expired | Execution waited over 24 hours | Start a new execution; token TTL is 24 hours |
```

I will also create the summary file next.