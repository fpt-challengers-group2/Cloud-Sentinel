---
title: "Lớp phát hiện (Detection Layer)"
weight: 2
chapter: false
pre: " <b> 2.2. </b> "
---

# 2.2. Lớp phát hiện – Amazon GuardDuty

## 2.2.1. Tổng quan

Lớp phát hiện có nhiệm vụ phát sinh cảnh báo khi có hành vi bất thường trên hạ tầng AWS. Trong CloudSentinel, nguồn đầu vào là **Amazon GuardDuty** – dịch vụ phát hiện đe dọa dựa trên machine learning. GuardDuty phân tích VPC Flow Logs, DNS logs, CloudTrail events để tạo các **finding** (sự cố bảo mật).

**Mục tiêu của lớp này:**
- Cung cấp cấu trúc finding chuẩn để các thành phần sau (Parser, History, Knowledge, Agent) có thể xử lý.
- Kích hoạt luồng điều phối của Step Function.

## 2.2.2. Thiết kế và triển khai

### 2.2.2.1. GuardDuty detector

- GuardDuty được bật thủ công trên region `ap-southeast-1` (nơi hệ thống hoạt động).
- Chi phí ước tính: khoảng **6.90 USD/tháng** với lưu lượng 6GB VPC Flow Logs.

### 2.2.2.2. Cấu trúc một finding (sau khi qua Lambda Parser)

Hệ thống sử dụng Lambda `lambda_parser` để trích xuất các trường quan trọng từ raw GuardDuty finding. Kết quả là một JSON chuẩn có các trường sau:

| Trường | Ý nghĩa | Ví dụ |
|--------|---------|-------|
| `finding_id` | UUID của sự cố | `"abc123-def456"` |
| `finding_type` | Loại tấn công | `"UnauthorizedAccess:EC2/SSHBruteForce"` |
| `severity` | Mức độ (0.1 – 8.9) | `5.0` |
| `resource_type` | Loại tài nguyên | `"Instance"` |
| `target_id` | ID tài nguyên | `"i-0123456789abcdef0"` |
| `region` | Vùng AWS | `"ap-southeast-1"` |
| `attacker_ip` | IP tấn công (nếu có) | `"203.0.113.45"` |
| `attacker_location` | Vị trí địa lý | `"Hanoi, Vietnam"` |
| `title` | Tiêu đề | `"SSH brute force from known malicious IP"` |
| `description` | Mô tả chi tiết | `"The remote host ..."` |
| `timestamp` | Thời gian (ISO) | `"2026-04-26T12:34:56Z"` |

### 2.2.2.3. Cơ chế kích hoạt Step Function

Trong thiết kế kiến trúc, GuardDuty sẽ gửi finding đến EventBridge, và EventBridge rule sẽ trigger Step Function. Tuy nhiên, **trong bản triển khai hiện tại, EventBridge rule chưa được hoàn thiện**. Để vẫn có thể vận hành và thử nghiệm pipeline, chúng ta sử dụng **mock event** – gửi trực tiếp finding JSON vào Step Function bằng AWS CLI (hướng dẫn ở phần Lab).

## 2.2.3. Hướng dẫn thực hành (Lab)

Trong phần này, bạn sẽ tự tay gửi một finding mẫu (mock) vào Step Function để kích hoạt toàn bộ pipeline.

### Bước 1: Tạo file mock event

Tạo file `mock_finding.json` với nội dung sau (bạn có thể sửa `target_id`, `attacker_ip` tùy ý):

```json
{
  "finding_id": "mock-finding-001",
  "finding_type": "Recon:EC2/Portscan",
  "severity": 5.0,
  "resource_type": "EC2",
  "target_id": "i-0123456789abcdef0",
  "region": "ap-southeast-1",
  "attacker_ip": "203.0.113.45",
  "attacker_location": "Hanoi, Vietnam",
  "title": "Port scan from known malicious IP",
  "description": "EC2 instance i-0123456789abcdef0 was probed by IP 203.0.113.45 on ports 22, 80, 443.",
  "timestamp": "2026-04-26T10:00:00Z"
}
```

### Bước 2: Lấy ARN của State Machine

```bash
aws stepfunctions list-state-machines --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text
```

### Bước 3: Khởi chạy execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn <ARN-thu-duoc> \
  --name test-run-$(date +%s) \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

### Bước 4: Quan sát kết quả

- Vào **AWS Console → Step Functions → State machines → `cloud-sentinel-orchestrator` → Executions**.
- Chọn execution vừa tạo.
- Xem từng state (Parse Finding → Check Precedent → Get Knowledge → Invoke Agents Pipeline → Send Telegram And Wait → Execute Remediation).
- Nếu bạn đã cấu hình Telegram và Cognito (theo hướng dẫn ở mục 2.4), bạn sẽ nhận được tin nhắn yêu cầu phê duyệt. Bấm **Approve** để pipeline chạy tiếp.

## 2.2.4. Kiểm tra và xác minh

- **CloudWatch Logs**: Mỗi Lambda (parser, history, knowledge, advisor, telegram_sender, executor) có log group riêng. Bạn có thể kiểm tra log để debug.
- **DynamoDB**: Sau khi execution hoàn thành, bảng `incident_history` sẽ có một bản ghi mới (nếu executor ghi thành công).
- **S3**: Báo cáo remediation được lưu dưới dạng JSON trong bucket `cloud-sentinel-reports-<account-id>`.

## 2.2.5. Tổng kết lớp phát hiện

- GuardDuty cung cấp nguồn sự kiện bảo mật, được chuẩn hóa qua `lambda_parser`.
- Mặc dù chưa có EventBridge trigger tự động, mock event cho phép chạy pipeline đầy đủ.
- Bạn đã thực hành gửi một finding mẫu và quan sát luồng xử lý.
