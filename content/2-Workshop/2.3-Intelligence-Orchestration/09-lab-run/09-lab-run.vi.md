---
title: "Lab: Chạy pipeline"
weight: 9
chapter: false
pre: " <b> 2.3.9. </b> "
---

# 2.3.9. Hướng dẫn thực hành (Lab)

## Bước 1: Xác nhận hạ tầng đã sẵn sàng

Kiểm tra các Lambda đã được deploy:

```bash
aws lambda list-functions \
  --region ap-southeast-1 \
  --query "Functions[?starts_with(FunctionName, 'cloud-sentinel')].FunctionName" \
  --output table
```

Kết quả mong đợi: 8 Lambda functions hiện diện, bao gồm `lambda_parser`, `lambda_history`, `lambda_knowledge`, `lambda_advisor`, `lambda_telegram_sender`, `lambda_approval_handler`, `lambda_executor`, `lambda_token_cleaner`.

Kiểm tra Step Function:

```bash
aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel')].{Name:name,ARN:stateMachineArn}" \
  --output table
```

## Bước 2: Tạo mock event

Tạo file `mock_finding.json` với nội dung (cấu trúc đúng theo chuẩn GuardDuty):

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

Bạn có thể thay đổi `finding_type`, `instanceId`, `ipAddressV4` để kiểm tra các loại sự cố khác nhau.

## Bước 3: Lấy ARN của State Machine

```bash
STATE_MACHINE_ARN=$(aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text)

echo "ARN: $STATE_MACHINE_ARN"
```

## Bước 4: Khởi chạy execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn $STATE_MACHINE_ARN \
  --name "test-run-$(date +%s)" \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

Lệnh sẽ trả về `executionArn`. Lưu lại giá trị này để theo dõi trạng thái.

## Bước 5: Quan sát luồng xử lý

Trên AWS Console:

1. Vào **Step Functions → State machines → `cloud-sentinel-orchestrator` → Executions**.
2. Chọn execution vừa tạo (trạng thái ban đầu là **Running**).
3. Quan sát từng state chuyển tiếp trên sơ đồ trực quan:
   - **Parse Finding** → **Check Precedent** → **Get Knowledge** → **Invoke Agents Pipeline**: mỗi state mất khoảng 5–30 giây tùy độ phức tạp của Bedrock inference.
   - **Send Telegram And Wait**: state machine chuyển sang trạng thái **Waiting** – đây là điểm chờ phê duyệt từ admin qua Telegram.

4. Kiểm tra CloudWatch Logs của `lambda_advisor` để xem output của Bedrock Agent:

```bash
aws logs tail /aws/lambda/cloud-sentinel-lambda_advisor \
  --region ap-southeast-1 \
  --since 10m
```
