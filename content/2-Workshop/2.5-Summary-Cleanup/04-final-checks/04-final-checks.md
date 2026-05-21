---
title: "Final Checks"
weight: 4
chapter: false
pre: " <b> 2.5.4. </b> "
---

# 2.5.2. Final Checks

Before cleanup, run a full verification to confirm the pipeline executed correctly.

## Check incident history – DynamoDB

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S,Target:target_id.S,Action:action_taken.S,Time:timestamp.S}" \
  --output table
```

Expected result: at least one record corresponding to an execution from Lab 2.4.

## Check reports – S3

```bash
# List stored reports
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# Read a report
aws s3 cp \
  s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

## Check execution status

```bash
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:10].{Name:name,Status:status,Start:startDate,Stop:stopDate}" \
  --output table
```

Executions from the lab should show `SUCCEEDED` (if Approved) or `FAILED` (if Rejected — expected behaviour).

## Check `task_tokens` table

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count" \
  --output text
```

Expected result: `0` (table empty after executions complete and tokens are removed).
