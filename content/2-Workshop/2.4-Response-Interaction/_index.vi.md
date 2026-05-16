---
title: "Lớp phản ứng & tương tác (Response & Interaction Layer)"
weight: 4
chapter: false
pre: " <b> 2.4. </b> "
---

# 2.4. Lớp phản ứng & tương tác – Telegram, Cognito, API Gateway & Executor

## 2.4.1. Tổng quan

Lớp phản ứng là điểm giao tiếp giữa hệ thống tự động và con người. Thay vì thực thi hành động bảo mật hoàn toàn tự động (vốn tiềm ẩn rủi ro với hạ tầng sản xuất), CloudSentinel áp dụng mô hình **Human-in-the-Loop**: hệ thống phân tích và đề xuất, chuyên viên bảo mật xem xét và quyết định.

Luồng hoạt động của lớp này:

```
[Kế hoạch ứng phó từ Advisor Agent]
        │
        ▼
lambda_telegram_sender   → Gửi tin nhắn Telegram kèm nút Approve/Reject
        │
        ▼ (Step Function pause – waitForTaskToken)
[Admin nhận tin, đọc kế hoạch, bấm nút]
        │
        ▼
Telegram webhook → API Gateway → lambda_approval_handler
        │
        ├─ Approve → Step Function tiếp tục → lambda_executor
        │                                          │
        │                                    Thực thi hành động
        │                                    (simulation) + ghi DynamoDB
        └─ Reject  → Step Function kết thúc (không thực thi)
```

Các thành phần cốt lõi của lớp này:

| Thành phần | Dịch vụ AWS / Bên thứ ba | Vai trò |
|------------|--------------------------|---------|
| Telegram Bot | Telegram API | Kênh gửi cảnh báo và nhận phê duyệt |
| Sender | Lambda `lambda_telegram_sender` | Định dạng và gửi tin nhắn |
| Webhook | API Gateway `/webhook` (POST) | Nhận callback từ Telegram |
| Approval Handler | Lambda `lambda_approval_handler` | Xác thực admin, giải phóng Step Function |
| Auth | Amazon Cognito User Pool | Quản lý danh sách admin được phép phê duyệt |
| Executor | Lambda `lambda_executor` | Thực thi hành động ứng phó (simulation) |

---

## 2.4.2. Thiết kế và triển khai

### 2.4.2.1. Telegram Bot và Lambda `lambda_telegram_sender`

**Chức năng:** Nhận kế hoạch ứng phó Markdown từ Advisor Agent và gửi tin nhắn phê duyệt tới nhóm admin trên Telegram.

**Cấu hình:**

| Biến môi trường | Giá trị | Nguồn |
|-----------------|---------|-------|
| `TELEGRAM_TOKEN` | Bot token từ BotFather | `terraform.tfvars` (sensitive) |
| `TELEGRAM_CHAT_ID` | Chat ID của nhóm admin | `terraform.tfvars` (sensitive) |
| `STEP_FUNCTIONS_ARN` | ARN của state machine | Tự động từ Terraform output |

**Cấu trúc tin nhắn gửi đến Telegram:**

```
🚨 CLOUD SENTINEL – YÊU CẦU PHÊ DUYỆT

Finding ID : mock-finding-001
Loại sự cố : Recon:EC2/Portscan
Mức độ     : 5.0 (Medium)
Tài nguyên : i-0123456789abcdef0
IP tấn công: 203.0.113.45 (Hanoi, Vietnam)

📋 KẾ HOẠCH ỨNG PHÓ (tóm tắt):
• Hành động đề xuất: block_ip
• Block IP 203.0.113.45 tại Network ACL
• Review Security Group rules
• ...

[✅ APPROVE]  [❌ REJECT]
```

Tin nhắn sử dụng **Telegram Inline Keyboard** với hai nút:
- **APPROVE** – callback_data chứa `approve:{task_token_hash}`
- **REJECT** – callback_data chứa `reject:{task_token_hash}`

`task_token` là token `waitForTaskToken` của Step Functions, được lưu tạm trong bảng DynamoDB `task_tokens` với TTL 24 giờ.

---

### 2.4.2.2. API Gateway – Webhook endpoint

API Gateway tạo một HTTP endpoint công khai để Telegram có thể gửi callback khi admin bấm nút.

| Thuộc tính | Giá trị |
|------------|---------|
| Endpoint | `POST /webhook` |
| Integration | Lambda `lambda_approval_handler` |
| Auth | Không có (Telegram tự xác thực qua secret token header) |
| Stage | `prod` |

**Luồng xử lý webhook:**

1. Admin bấm **Approve** hoặc **Reject** trên Telegram.
2. Telegram gửi `POST /webhook` với payload chứa `callback_query`.
3. API Gateway forward ngay đến `lambda_approval_handler`.

---

### 2.4.2.3. Amazon Cognito – Xác thực admin

Cognito đảm bảo chỉ những admin đã được đăng ký mới có quyền phê duyệt. Khi `lambda_approval_handler` nhận callback từ Telegram, nó trích xuất `telegram_id` từ payload và đối chiếu với Cognito User Pool trước khi xử lý.

**Cấu hình Cognito:**

| Thuộc tính | Giá trị |
|------------|---------|
| User Pool name | `cloud-sentinel-admin-pool` |
| Schema bổ sung | `custom:telegram_id` (String, mutable) |
| User mặc định | `security-admin` |
| Telegram ID đã map | `8320828979` |

**Cơ chế xác thực trong `lambda_approval_handler`:**

1. Nhận `telegram_id` từ `callback_query.from.id` trong payload Telegram.
2. Gọi Cognito `list_users` với filter `custom:telegram_id = <telegram_id>`.
3. Nếu tìm thấy user → xác thực thành công, tiến hành xử lý token.
4. Nếu không tìm thấy → trả về `403 Forbidden`, bỏ qua callback.

> **Lý do thiết kế:** Xác thực qua Cognito thay vì đơn giản hardcode danh sách admin giúp dễ dàng thêm/bớt admin mà không cần deploy lại Lambda. Chỉ cần cập nhật attribute `custom:telegram_id` cho user trong Cognito Console.

---

### 2.4.2.4. Lambda `lambda_approval_handler`

**Chức năng:** Nhận callback từ Telegram, xác thực admin, lấy task token từ DynamoDB, và gửi tín hiệu tiếp tục hoặc dừng tới Step Functions.

**Luồng xử lý:**

```
Nhận callback_query từ API Gateway
        │
        ▼
Trích xuất telegram_id + callback_data
        │
        ▼
Xác thực với Cognito (telegram_id có trong User Pool?)
        │
        ├─ Không hợp lệ → Return 403
        │
        └─ Hợp lệ
               │
               ▼
        Lấy task_token từ DynamoDB (PK = finding_id)
               │
               ▼
        ┌─ Approve ──→ step_functions.send_task_success(taskToken, output)
        └─ Reject  ──→ step_functions.send_task_failure(taskToken, cause)
               │
               ▼
        Xóa token khỏi DynamoDB (tránh dùng lại)
               │
               ▼
        Return 200 OK cho Telegram
```

IAM permissions của Lambda này bao gồm: `states:SendTaskSuccess`, `states:SendTaskFailure`, `dynamodb:GetItem`, `dynamodb:DeleteItem`, `cognito-idp:ListUsers`.

---

### 2.4.2.5. Lambda `lambda_executor`

**Chức năng:** Sau khi admin Approve, Step Functions chuyển sang state `Execute Remediation` và gọi `lambda_executor` với thông tin sự cố và hành động đề xuất.

**Các hành động được hỗ trợ (hiện tại ở chế độ simulation):**

| Hành động | Hàm | Mô tả |
|-----------|-----|-------|
| `block_ip` | `block_ip_nacl()` | Block IP nguồn tại Network ACL |
| `isolate_instance` | `isolate_instance()` | Cô lập EC2 instance khỏi network |
| `revoke_keys` | `revoke_iam_keys()` | Thu hồi IAM access keys |

**Chế độ simulation hiện tại:** Các hàm trên chỉ ghi log `[SIMULATION]` và trả về `status: "simulated_success"` mà không thực sự gọi AWS API để thay đổi tài nguyên. Đây là thiết kế phù hợp cho môi trường workshop – an toàn để chạy lặp lại nhiều lần.

Ví dụ output của một hành động simulation:

```json
{
  "action": "block_ip",
  "ip": "203.0.113.45",
  "status": "simulated_success",
  "timestamp": "2026-04-26T10:05:32Z"
}
```

**Sau khi thực thi**, `lambda_executor` thực hiện hai thao tác ghi:

1. **DynamoDB `incident_history`** – ghi bản ghi sự cố với `action_taken`, `severity`, `timestamp`.
2. **S3** – cập nhật file `remediations/{finding_id}.json` với kết quả thực thi.

---

## 2.4.3. Luồng dữ liệu

Sơ đồ dưới đây mô tả toàn bộ vòng lặp phê duyệt từ khi Step Function tạm dừng đến khi execution hoàn thành:

```
Step Function: Send Telegram And Wait
        │
        ├─ Lưu task_token vào DynamoDB (task_tokens)
        │
        └─ lambda_telegram_sender
               │
               ▼
         [Telegram Bot gửi tin đến nhóm admin]
               │
               ▼ (Admin đọc kế hoạch ứng phó)
         [Admin bấm APPROVE hoặc REJECT]
               │
               ▼
         Telegram API → POST /webhook
               │
               ▼
         API Gateway → lambda_approval_handler
               │
               ├─ Xác thực Cognito (telegram_id)
               ├─ Lấy task_token từ DynamoDB
               │
               ├─ APPROVE: send_task_success → Step Function tiếp tục
               │                    ↓
               │             lambda_executor
               │                    ↓
               │         [SIMULATION] Thực thi hành động
               │                    ↓
               │         Ghi DynamoDB + S3
               │                    ↓
               │         Execution: SUCCEEDED
               │
               └─ REJECT: send_task_failure → Execution: FAILED (rejected)
```

---

## 2.4.4. Hướng dẫn thực hành (Lab)

Phần Lab này tiếp nối trực tiếp từ Lab của mục 2.3 – execution đang ở trạng thái **Waiting** tại state `Send Telegram And Wait`. Bạn sẽ thực hành quan sát tin nhắn Telegram, phê duyệt và theo dõi kết quả thực thi.

### Bước 1: Cấu hình Telegram Bot token (nếu chưa làm)

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

### Bước 2: Xác nhận Cognito user đã có telegram_id

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

### Bước 3: Kiểm tra webhook đã được đăng ký

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

### Bước 4: Khởi chạy execution và quan sát Telegram

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

### Bước 5: Phê duyệt và theo dõi kết quả

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

---

## 2.4.5. Kiểm tra và xác minh

### Kiểm tra trạng thái execution

```bash
# Liệt kê các execution gần đây
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:5].{Name:name,Status:status,Start:startDate}" \
  --output table
```

### Kiểm tra task_tokens đã được xóa

Sau khi execution hoàn thành (Approve hoặc Reject), token phải được xóa khỏi bảng `task_tokens`:

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count"
```

Kết quả mong đợi: `0` (bảng rỗng sau khi xử lý xong).

### Kiểm tra API Gateway logs

```bash
# Xem access log của API Gateway (nếu đã bật)
aws logs tail /aws/apigateway/cloud-sentinel \
  --region ap-southeast-1 \
  --since 30m \
  --format short
```

### Troubleshooting – Các lỗi thường gặp

| Triệu chứng | Nguyên nhân có thể | Cách xử lý |
|-------------|-------------------|------------|
| Không nhận được tin Telegram | Bot token sai hoặc webhook chưa đăng ký | Kiểm tra `TELEGRAM_TOKEN` trong Lambda env vars; chạy lại lệnh `setWebhook` |
| Bấm Approve nhưng Step Function vẫn Waiting | Webhook URL chưa đúng hoặc API Gateway chưa deploy | Kiểm tra URL webhook bằng `getWebhookInfo`; confirm API Gateway stage `prod` đã deploy |
| Lambda `approval_handler` trả về 403 | `telegram_id` của bạn chưa được map trong Cognito | Chạy lại lệnh `admin-update-user-attributes` với đúng telegram ID |
| `lambda_executor` lỗi permission | IAM role thiếu quyền DynamoDB/S3 | Kiểm tra CloudWatch Logs; đảm bảo Terraform đã apply đầy đủ IAM policy |
| Task token expired | Execution chờ quá 24 giờ | Khởi chạy execution mới; TTL token được thiết kế 24 giờ |

---

## 2.4.6. Tổng kết lớp phản ứng & tương tác

Sau khi hoàn thành phần này, bạn đã hiểu và thực hành:

- **Human-in-the-Loop:** Cơ chế `waitForTaskToken` của Step Functions cho phép tạm dừng workflow vô thời hạn, chờ quyết định từ con người mà không tốn chi phí compute.
- **Telegram Bot làm kênh vận hành:** Thay vì portal web phức tạp, một bot đơn giản đủ để gửi cảnh báo và nhận phê duyệt với inline keyboard.
- **Cognito làm lớp xác thực nhẹ:** Cho phép quản lý danh sách admin linh hoạt mà không cần triển khai hệ thống auth riêng.
- **Executor simulation:** Cấu trúc code rõ ràng, sẵn sàng chuyển sang hành động thật bằng cách thay nội dung các hàm và cập nhật IAM policy.
