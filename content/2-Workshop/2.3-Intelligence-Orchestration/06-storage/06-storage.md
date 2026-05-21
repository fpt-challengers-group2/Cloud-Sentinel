---
title: "Storage: DynamoDB and S3"
weight: 6
chapter: false
pre: " <b> 2.3.6. </b> "
---

# 2.3.6. Storage – DynamoDB and S3

## DynamoDB – `cloud-sentinel-security-incident-history` table

Stores history of processed incidents. Table schema:

| Attribute | Type | Role |
|-----------|------|------|
| `finding_type` | String (PK) | Partition key |
| `target_id` | String (SK) | Sort key |
| `timestamp` | String (GSI) | Time-based filtering |
| `action_taken` | String | Action performed |
| `severity` | Number | Severity level |
| TTL | 90 days | Auto-delete old records |

## DynamoDB – `cloud-sentinel-task-tokens` table

Stores Step Functions task tokens while awaiting approvals:

| Attribute | Type | Role |
|-----------|------|------|
| `finding_id` | String (PK) | Partition key |
| `task_token` | String | Step Functions `waitForTaskToken` token |
| TTL | 24 hours | Auto-delete expired tokens |

## S3 – Reports bucket

- **Bucket name:** `cloud-sentinel-reports-{account_id}`
- **Report path:** `remediations/{finding_id}.json`
- **Contents:** JSON containing the Markdown remediation plan + incident metadata.
