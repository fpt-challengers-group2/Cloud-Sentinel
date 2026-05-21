---
title: "Lab: Send mock event"
weight: 4
chapter: false
pre: " <b> 2.2.4. </b> "
---

# 2.2.4. Lab Instructions

In this lab, you'll send a mock finding to the Step Function to trigger the pipeline.

## Step 1: Create mock event file

Create `mock_finding.json` with the following content (structure matches GuardDuty event expected by EventBridge):

```json
{
  "detail": {
    "id": "mock-finding-001",
    "type": "Recon:EC2/Portscan",
    "severity": 5.0,
    "region": "ap-southeast-1",
    "title": "Port scan from known malicious IP",
    "description": "EC2 instance i-0123456789abcdef0 was probed by IP 203.0.113.45 on ports 22, 80, 443.",
    "createdAt": "2026-04-26T10:00:00Z",
    "resource": {
      "resourceType": "Instance",
      "instanceDetails": {
        "instanceId": "i-0123456789abcdef0"
      }
    },
    "service": {
      "action": {
        "networkConnectionAction": {
          "remoteIpDetails": {
            "ipAddressV4": "203.0.113.45",
            "country": {
              "countryName": "Vietnam"
            }
          }
        }
      }
    }
  }
}
```

You can modify `instanceId` and `ipAddressV4` as needed.

## Step 2: Get State Machine ARN

```bash
aws stepfunctions list-state-machines --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text
```

## Step 3: Start execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn <ARN-obtained> \
  --name test-run-$(date +%s) \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

## Step 4: Observe results

- Open the AWS Console → Step Functions → State machines → `cloud-sentinel-orchestrator` → Executions.
- Select the created execution.
- Inspect each state (Parse Finding → Check Precedent → Get Knowledge → Invoke Agents Pipeline → Send Telegram And Wait → Execute Remediation).
- If Telegram and Cognito are configured (per 2.4), you'll receive an approval message. Press **Approve** to continue the pipeline.
