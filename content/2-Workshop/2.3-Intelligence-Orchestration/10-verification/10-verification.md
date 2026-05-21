---
title: "Verification"
weight: 10
chapter: false
pre: " <b> 2.3.10. </b> "
---

# 2.3.10. Verification

## Check Lambda History – DynamoDB

After execution completes, inspect the `incident_history` table:

**Console:**
- Open DynamoDB → Tables → `cloud-sentinel-incident_history` → Explore items.

**CLI:**

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S, Target:target_id.S, Action:action_taken.S}" \
  --output table
```

## Check reports – S3

```bash
# List saved reports
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# View a report
aws s3 cp s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

## Check Lambda logs

Each Lambda has its own log group with 30-day retention. Example:

```bash
# Replace lambda_parser with the Lambda name you want to inspect
aws logs tail /aws/lambda/cloud-sentinel-lambda_parser \
  --region ap-southeast-1 \
  --since 30m \
  --format short
```
