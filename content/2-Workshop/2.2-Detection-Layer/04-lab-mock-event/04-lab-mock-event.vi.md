---
title: "Lab: Gửi mock event"
weight: 4
chapter: false
pre: " <b> 2.2.4. </b> "
---

# 2.2.4. Hướng dẫn thực hành (Lab)

Trong phần này, bạn sẽ tự tay gửi một finding mẫu (mock) vào Step Function để kích hoạt toàn bộ pipeline.

## Bước 1: Tạo file mock event

Tạo file `mock_finding.json` với nội dung sau (cấu trúc đúng theo chuẩn GuardDuty gửi đến EventBridge):

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

Có thể thay đổi `instanceId`, `ipAddressV4` tùy ý.

## Bước 2: Lấy ARN của State Machine

```bash
aws stepfunctions list-state-machines --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text
```

## Bước 3: Khởi chạy execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn <ARN-thu-duoc> \
  --name test-run-$(date +%s) \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

## Bước 4: Quan sát kết quả

- Vào **AWS Console → Step Functions → State machines → `cloud-sentinel-orchestrator` → Executions**.
- Chọn execution vừa tạo.
- Xem từng state (Parse Finding → Check Precedent → Get Knowledge → Invoke Agents Pipeline → Send Telegram And Wait → Execute Remediation).
- Nếu bạn đã cấu hình Telegram và Cognito (theo hướng dẫn ở mục 2.4), bạn sẽ nhận được tin nhắn yêu cầu phê duyệt. Bấm **Approve** để pipeline chạy tiếp.
