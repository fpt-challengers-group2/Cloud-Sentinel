---
title: "Lab: Phê duyệt qua Telegram"
weight: 7
chapter: false
pre: " <b> 2.4.7. </b> "
---

# 2.4.7. Hướng dẫn thực hành (Lab)

Phần Lab này tiếp nối trực tiếp từ Lab của mục 2.3 – execution đang ở trạng thái **Waiting** tại state `Send Telegram And Wait`. Bạn sẽ thực hành quan sát tin nhắn Telegram, phê duyệt và theo dõi kết quả thực thi.

## Bước 1: Cấu hình Telegram Bot token (nếu chưa làm)

Nếu bạn chưa thiết lập Bot token trong Terraform, thực hiện như sau:

```bash
# Kiểm tra xem biến môi trường Lambda đã có token chưa
aws lambda get-function-configuration \
  --function-name cloud-sentinel-lambda_telegram_sender \
  --region ap-southeast-1 \
  --query "Environment.Variables.TELEGRAM_TOKEN" \
  --output text
```

Nếu kết quả trống hoặc lỗi, cập nhật `terraform.tfvars` với giá trị `telegram_token` và `telegram_chat_id`, sau đó chạy lại `terraform apply`.

## Bước 2: Xác nhận Cognito user đã có telegram_id

```bash
# Lấy User Pool ID
USER_POOL_ID=$(aws cognito-idp list-user-pools \
  --max-results 10 \
  --region ap-southeast-1 \
  --query "UserPools[?contains(Name,'cloud-sentinel')].Id" \
  --output text)

echo "User Pool ID: $USER_POOL_ID"

# Kiểm tra user security-admin
aws cognito-idp admin-get-user \
  --user-pool-id $USER_POOL_ID \
  --username security-admin \
  --region ap-southeast-1 \
  --query "UserAttributes[?Name=='custom:telegram_id']"
```

Kết quả mong đợi: attribute `custom:telegram_id` có giá trị là Telegram user ID của bạn.

Nếu chưa có, cập nhật bằng lệnh sau (thay `<your-telegram-id>` bằng ID thực của bạn):

```bash
aws cognito-idp admin-update-user-attributes \
  --user-pool-id $USER_POOL_ID \
  --username security-admin \
  --user-attributes Name="custom:telegram_id",Value="<your-telegram-id>" \
  --region ap-southeast-1
```

> **Cách lấy Telegram ID của bạn:** Gửi tin nhắn `/start` đến bot `@userinfobot` trên Telegram – bot sẽ trả về `Id` của tài khoản bạn đang dùng.

## Bước 3: Kiểm tra webhook đã được đăng ký

API Gateway tạo endpoint webhook. Kiểm tra URL:

```bash
aws apigateway get-rest-apis \
  --region ap-southeast-1 \
  --query "items[?contains(name,'cloud-sentinel')].{Name:name,ID:id}" \
  --output table
```

Lấy API ID từ kết quả trên, sau đó kiểm tra URL thực tế:

```bash
# URL có dạng: https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/webhook
echo "Webhook URL: https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/webhook"
```

Đăng ký webhook với Telegram (thay `<BOT_TOKEN>` và `<WEBHOOK_URL>`):

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/setWebhook" \
  -d "url=<WEBHOOK_URL>"
```

Kết quả mong đợi: `{"ok":true,"result":true,"description":"Webhook was set"}`.

## Bước 4: Khởi chạy execution và quan sát Telegram

Nếu chưa khởi chạy từ mục 2.3, chạy lại:

```bash
aws stepfunctions start-execution \
  --state-machine-arn $STATE_MACHINE_ARN \
  --name "test-run-$(date +%s)" \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

Quan sát nhóm Telegram của bạn:
- Sau khoảng 30–90 giây (thời gian Bedrock Agent phân tích), bot sẽ gửi tin nhắn chứa tóm tắt kế hoạch ứng phó và hai nút **APPROVE** / **REJECT**.
- Đọc kỹ kế hoạch ứng phó trước khi quyết định.

## Bước 5: Phê duyệt và theo dõi kết quả

**Bấm APPROVE** trên Telegram. Quan sát:

1. Trên **AWS Console → Step Functions**: state `Send Telegram And Wait` chuyển sang **Succeeded**, tiếp đến `Execute Remediation` bắt đầu chạy.
2. Kiểm tra log `lambda_executor`:

```bash
aws logs tail /aws/lambda/cloud-sentinel-lambda_executor \
  --region ap-southeast-1 \
  --since 5m \
  --format short
```

Log mong đợi:
```
[SIMULATION] Blocking IP 203.0.113.45 in ap-southeast-1
Action result: {"action": "block_ip", "status": "simulated_success"}
Incident record saved to DynamoDB.
Report saved to S3: remediations/mock-finding-001.json
```

3. Kiểm tra bản ghi DynamoDB:

```bash
aws dynamodb get-item \
  --table-name cloud-sentinel-incident_history \
  --key '{"finding_type":{"S":"Recon:EC2/Portscan"},"target_id":{"S":"i-0123456789abcdef0"}}' \
  --region ap-southeast-1 \
  --output json
```

4. Kiểm tra báo cáo S3:

```bash
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1
```

**Thử bấm REJECT** với một execution khác: Step Function sẽ chuyển sang trạng thái **Failed** với nguyên nhân `AdminRejected`. Không có hành động nào được thực thi và không có bản ghi nào được ghi vào DynamoDB.
