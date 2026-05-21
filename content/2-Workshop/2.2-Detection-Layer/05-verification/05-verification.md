---
title: "Verification"
weight: 5
chapter: false
pre: " <b> 2.2.5. </b> "
---

# 2.2.5. Verification

- **CloudWatch Logs:** Each Lambda (parser, history, knowledge, advisor, telegram_sender, executor) has its own log group. Check logs for debugging.
- **DynamoDB:** After execution completes, the `incident_history` table should contain a new record (if executor wrote successfully).
- **S3:** Remediation reports are stored as JSON in `cloud-sentinel-reports-<account-id>` bucket.
