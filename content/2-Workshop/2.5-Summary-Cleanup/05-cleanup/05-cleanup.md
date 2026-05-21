---
title: "Infrastructure Cleanup Guide"
weight: 5
chapter: false
pre: " <b> 2.5.5. </b> "
---

# 2.5.3. Infrastructure Cleanup Guide

> **Important:** Run this after completing the workshop and when you no longer need the infrastructure. `terraform destroy` will **permanently delete** all created resources, including data in DynamoDB and S3.

## Step 1: Backup necessary data (optional)

If you want to keep reports generated during the workshop:

```bash
# Download all reports locally
aws s3 sync \
  s3://cloud-sentinel-reports-<account-id>/remediations/ \
  ./backup-reports/ \
  --region ap-southeast-1

echo "Backed up $(ls ./backup-reports/ | wc -l) reports."
```

## Step 2: Empty S3 (required before destroy)

Terraform cannot delete a non-empty S3 bucket. You must empty the bucket first:

```bash
# Remove all objects in the bucket
aws s3 rm s3://cloud-sentinel-reports-<account-id>/ \
  --recursive \
  --region ap-southeast-1

echo "Bucket emptied."
```

## Step 3: Unregister Telegram webhook

Before deleting API Gateway, remove the webhook so Telegram stops sending requests to deleted endpoints:

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/deleteWebhook"
```

Expected result: `{"ok":true,"result":true,"description":"Webhook was deleted"}`.

## Step 4: Run `terraform destroy`

Change to the project's Terraform directory:

```bash
cd <path-to-terraform-directory>

# Preview resources to be deleted
terraform plan -destroy

# Confirm and execute destroy
terraform destroy
```

Terraform will list resources and ask for confirmation. Type `yes` to proceed.

## Step 5: Verify cleanup

After `terraform destroy` completes:

```bash
# Verify Lambda functions removed
aws lambda list-functions \
  --region ap-southeast-1 \
  --query "Functions[?starts_with(FunctionName,'cloud-sentinel')].FunctionName" \
  --output table

# Verify Step Function removed
aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel')].name" \
  --output text

# Verify DynamoDB tables removed
aws dynamodb list-tables \
  --region ap-southeast-1 \
  --query "TableNames[?starts_with(@,'cloud-sentinel')]" \
  --output text
```

Expected: all commands return empty results.

## Step 6: Delete S3 backend (optional)

If you created an S3 bucket for `terraform.tfstate` as described in 2.1, delete it if unused by other projects:

```bash
# Remove tfstate contents
aws s3 rm s3://cloud-sentinel-tfstate-<yourname>/ \
  --recursive \
  --region ap-southeast-1

# Remove bucket
aws s3 rb s3://cloud-sentinel-tfstate-<yourname> \
  --region ap-southeast-1
```

## Step 7: Disable GuardDuty (if enabled)

If you enabled GuardDuty during the workshop and no longer need it:

```bash
# Get Detector ID
DETECTOR_ID=$(aws guardduty list-detectors \
  --region ap-southeast-1 \
  --query "DetectorIds[0]" \
  --output text)

# Delete detector
aws guardduty delete-detector \
  --detector-id $DETECTOR_ID \
  --region ap-southeast-1

echo "GuardDuty detector deleted."
```

> GuardDuty charges for log analysis. Skipping this step may result in ongoing costs even after CloudSentinel resources are removed.
