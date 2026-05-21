---
title: "Lambda Executor"
weight: 5
chapter: false
pre: " <b> 2.4.5. </b> "
---

# 2.4.5. Lambda `lambda_executor`

**Function:** After admin Approves, Step Functions moves to `Execute Remediation` and invokes `lambda_executor` with incident details and the proposed action.

**Supported actions (currently simulation):**

| Action | Function | Description |
|--------|----------|-------------|
| `block_ip` | `block_ip_nacl()` | Block source IP in Network ACL |
| `isolate_instance` | `isolate_instance()` | Isolate EC2 instance from network |
| `revoke_keys` | `revoke_iam_keys()` | Revoke IAM access keys |

**Simulation mode:** The functions currently log `[SIMULATION]` and return `status: "simulated_success"` without calling AWS APIs to change resources. This is suitable for workshop environments — safe to run repeatedly.

Example simulation output:

```json
{
  "action": "block_ip",
  "ip": "203.0.113.45",
  "status": "simulated_success",
  "timestamp": "2026-04-26T10:05:32Z"
}
```

**After execution**, `lambda_executor` writes two records:

1. **DynamoDB `incident_history`** – record with `action_taken`, `severity`, `timestamp`.
2. **S3** – update `remediations/{finding_id}.json` with execution results.

IAM permissions for this Lambda include: `dynamodb:PutItem`, `s3:PutObject`, and additional permissions if real actions are enabled (e.g., `ec2:CreateNetworkAclEntry` for blocking IP).
