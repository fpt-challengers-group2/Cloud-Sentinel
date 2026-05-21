---
title: "Lab: Run the pipeline"
weight: 9
chapter: false
pre: " <b> 2.3.9. </b> "
---

# 2.3.9. Lab Instructions

## Step 1: Verify infrastructure is ready

Check deployed Lambdas:

```bash
aws lambda list-functions \
  --region ap-southeast-1 \
  --query "Functions[?starts_with(FunctionName, 'cloud-sentinel')].FunctionName" \
  --output table
```

Expected: 8 Lambda functions including `lambda_parser`, `lambda_history`, `lambda_knowledge`, `lambda_advisor`, `lambda_telegram_sender`, `lambda_approval_handler`, `lambda_executor`, `lambda_token_cleaner`.

Check the Step Function:

```bash
aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel')].{Name:name,ARN:stateMachineArn}" \
  --output table
```

## Step 2: Create mock event

Create `mock_finding.json` with the provided GuardDuty-structured content (you can modify `finding_type`, `instanceId`, `ipAddressV4`).

## Step 3: Get State Machine ARN

```bash
STATE_MACHINE_ARN=$(aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text)

echo "ARN: $STATE_MACHINE_ARN"
```

## Step 4: Start execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn $STATE_MACHINE_ARN \
  --name "test-run-$(date +%s)" \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

The command returns an `executionArn`. Save it to track status.

## Step 5: Observe the flow

In the AWS Console:

1. Open Step Functions → State machines → `cloud-sentinel-orchestrator` → Executions.
2. Select the created execution (initial state: **Running**).
3. Watch transitions:
   - **Parse Finding** → **Check Precedent** → **Get Knowledge** → **Invoke Agents Pipeline**: each state may take ~5–30s depending on Bedrock inference.
   - **Send Telegram And Wait**: the state machine enters **Waiting** for admin approval via Telegram.

4. Tail `lambda_advisor` logs to inspect Bedrock output:

```bash
aws logs tail /aws/lambda/cloud-sentinel-lambda_advisor \
  --region ap-southeast-1 \
  --since 10m
```
