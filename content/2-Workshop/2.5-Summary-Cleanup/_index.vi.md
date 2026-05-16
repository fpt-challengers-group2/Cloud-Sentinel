---
title: "Tổng kết & Dọn dẹp (Summary & Cleanup)"
weight: 5
chapter: false
pre: " <b> 2.5. </b> "
---

# 2.5. Tổng kết & Dọn dẹp

## 2.5.1. Tổng kết workshop

### 2.5.1.1. Những gì đã xây dựng

Triển khai một hệ thống SOAR hoàn chỉnh trên AWS với kiến trúc Serverless thuần túy:

```
[Amazon GuardDuty]
       │  finding (mock event)
       ▼
[AWS Step Functions – 6 states]
  │
  ├─ Parse Finding         → lambda_parser
  ├─ Check Precedent       → lambda_history    ↔  DynamoDB (incident_history)
  ├─ Get Knowledge         → lambda_knowledge  ↔  Pinecone (RAG)
  ├─ Invoke Agents         → lambda_advisor
  │       ├─ Supervisor Agent (Bedrock – Claude 3.5 Sonnet)
  │       └─ Advisor Agent  (Bedrock – Claude 3.5 Sonnet)
  ├─ Send Telegram & Wait  → lambda_telegram_sender  → Telegram Bot
  │       └─ [Admin phê duyệt] → API Gateway → lambda_approval_handler ↔ Cognito
  └─ Execute Remediation   → lambda_executor   ↔  DynamoDB + S3
```

**Tổng số tài nguyên AWS đã triển khai qua Terraform:**

| Loại tài nguyên | Số lượng | Tên / Ghi chú |
|----------------|----------|---------------|
| Lambda Functions | 8 | parser, history, knowledge, advisor, telegram_sender, approval_handler, executor, token_cleaner |
| Step Functions | 1 | `cloud-sentinel-orchestrator` |
| Bedrock Agents | 2 | Supervisor Agent, Advisor Agent |
| DynamoDB Tables | 2 | `incident_history`, `task_tokens` |
| S3 Buckets | 1 | `cloud-sentinel-reports-{account_id}` |
| API Gateway | 1 | REST API với endpoint `/webhook` |
| Cognito User Pool | 1 | `cloud-sentinel-admin-pool` |
| IAM Roles | 8+ | Mỗi Lambda có role riêng, theo nguyên tắc least privilege |
| CloudWatch Log Groups | 8+ | Retention 30 ngày cho mỗi Lambda |

---

### 2.5.1.2. Những gì đã học được

**Về kiến trúc SOAR trên AWS:**
- Thiết kế hệ thống hướng sự kiện (Event-Driven) với chi phí gần bằng 0 khi không có sự cố.
- Áp dụng mô hình Human-in-the-Loop với `waitForTaskToken` của Step Functions để kiểm soát hành động tự động.
- Tổ chức IAM theo nguyên tắc least privilege: mỗi Lambda chỉ có đúng quyền cần thiết.

**Về Multi-Agent AI:**
- Supervisor Agent sử dụng action groups để chủ động thu thập ngữ cảnh (agentic loop).
- Advisor Agent (pure LLM) tổng hợp và lập kế hoạch ứng phó có cấu trúc.
- Phân tách vai trò rõ ràng giúp dễ kiểm soát và debug từng Agent độc lập.

**Về RAG với Pinecone:**
- Vector search cho phép truy vấn guideline theo ngữ nghĩa thay vì từ khóa chính xác.
- Amazon Titan Embeddings làm cầu nối giữa text và vector space.

**Về vận hành:**
- Dùng Terraform để quản lý toàn bộ hạ tầng – mọi tài nguyên đều có thể tái tạo hoặc xóa bằng một lệnh.
- CloudWatch Logs tập trung, dễ debug từng Lambda trong pipeline.

---

### 2.5.1.3. Giới hạn hiện tại và hướng phát triển

Hệ thống hiện tại đã hoạt động đầy đủ như một proof-of-concept. Một số điểm cần lưu ý khi muốn đưa vào môi trường sản xuất:

| Hạng mục | Trạng thái hiện tại | Hướng phát triển |
|----------|--------------------|--------------------|
| GuardDuty trigger | Dùng mock event thủ công | Triển khai EventBridge rule tự động kích hoạt Step Function khi GuardDuty phát sinh finding |
| Lambda Executor | Simulation (chỉ log) | Implement hành động thật: NACL modification, Security Group update, IAM key revocation |
| Pinecone knowledge base | Đã tạo index, cần seed guideline thực tế | Xây dựng quy trình nhập và cập nhật guideline định kỳ |
| Token cleaner | Lambda đã có, chưa có CloudWatch Schedule | Thêm EventBridge Scheduled Rule chạy `lambda_token_cleaner` mỗi giờ |
| Monitoring | CloudWatch Logs cơ bản | Xây dựng CloudWatch Dashboard tổng hợp + CloudWatch Alarms cho Lambda errors |
| Bedrock Agent alias | Dùng `TSTALIASID` (DRAFT) | Tạo alias PROD sau khi đã kiểm thử ổn định |

---

## 2.5.2. Kiểm tra kết quả cuối cùng

Trước khi dọn dẹp, thực hiện một lượt kiểm tra toàn bộ để xác nhận pipeline đã chạy đúng.

### Kiểm tra lịch sử sự cố – DynamoDB

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S,Target:target_id.S,Action:action_taken.S,Time:timestamp.S}" \
  --output table
```

Kết quả mong đợi: ít nhất một bản ghi tương ứng với execution đã chạy ở phần Lab 2.4.

### Kiểm tra báo cáo – S3

```bash
# Liệt kê tất cả báo cáo đã lưu
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# Đọc nội dung báo cáo
aws s3 cp \
  s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

### Kiểm tra trạng thái execution

```bash
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:10].{Name:name,Status:status,Start:startDate,Stop:stopDate}" \
  --output table
```

Các execution từ phần Lab phải có trạng thái `SUCCEEDED` (nếu đã Approve) hoặc `FAILED` (nếu đã Reject – đây là hành vi đúng, không phải lỗi).

### Kiểm tra bảng task_tokens

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count" \
  --output text
```

Kết quả mong đợi: `0` – bảng đã được dọn sạch sau mỗi lần xử lý.

---

## 2.5.3. Hướng dẫn dọn dẹp hạ tầng

> **Lưu ý quan trọng:** Thực hiện phần này khi bạn đã hoàn thành toàn bộ workshop và không cần dùng hạ tầng nữa. Lệnh `terraform destroy` sẽ **xóa vĩnh viễn** toàn bộ tài nguyên đã tạo, bao gồm dữ liệu trong DynamoDB và S3.

### Bước 1: Sao lưu dữ liệu cần thiết (tùy chọn)

Nếu muốn giữ lại báo cáo đã tạo trong quá trình workshop:

```bash
# Tải toàn bộ báo cáo về máy local
aws s3 sync \
  s3://cloud-sentinel-reports-<account-id>/remediations/ \
  ./backup-reports/ \
  --region ap-southeast-1

echo "Đã sao lưu $(ls ./backup-reports/ | wc -l) báo cáo."
```

### Bước 2: Xóa dữ liệu trong S3 (bắt buộc trước khi destroy)

Terraform không thể xóa S3 bucket còn chứa object. Cần làm rỗng bucket trước:

```bash
# Xóa toàn bộ object trong bucket
aws s3 rm s3://cloud-sentinel-reports-<account-id>/ \
  --recursive \
  --region ap-southeast-1

echo "Bucket đã được làm rỗng."
```

### Bước 3: Hủy đăng ký Telegram webhook

Trước khi xóa API Gateway, hủy webhook để Telegram ngừng gửi request đến endpoint đã xóa:

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/deleteWebhook"
```

Kết quả mong đợi: `{"ok":true,"result":true,"description":"Webhook was deleted"}`.

### Bước 4: Chạy terraform destroy

Di chuyển vào thư mục Terraform của dự án:

```bash
cd <đường-dẫn-đến-thư-mục-terraform>

# Xem trước danh sách tài nguyên sẽ bị xóa
terraform plan -destroy

# Xác nhận và thực thi xóa
terraform destroy
```

Terraform sẽ hiển thị danh sách tài nguyên và yêu cầu xác nhận. Gõ `yes` để tiến hành.

```
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes
```

Quá trình destroy thường mất 3–8 phút. Các tài nguyên được xóa theo thứ tự ngược với thứ tự tạo.

### Bước 5: Xác nhận đã dọn dẹp hoàn toàn

Sau khi `terraform destroy` hoàn thành:

```bash
# Xác nhận Lambda đã bị xóa
aws lambda list-functions \
  --region ap-southeast-1 \
  --query "Functions[?starts_with(FunctionName,'cloud-sentinel')].FunctionName" \
  --output table

# Xác nhận Step Function đã bị xóa
aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel')].name" \
  --output text

# Xác nhận DynamoDB tables đã bị xóa
aws dynamodb list-tables \
  --region ap-southeast-1 \
  --query "TableNames[?starts_with(@,'cloud-sentinel')]" \
  --output text
```

Kết quả mong đợi: tất cả các lệnh trả về kết quả rỗng.

### Bước 6: Xóa S3 Backend (tùy chọn)

Nếu bạn đã tạo S3 bucket để lưu `terraform.tfstate` theo hướng dẫn ở mục 2.1, đây là lúc xóa nó nếu không cần dùng cho dự án khác:

```bash
# Xóa toàn bộ nội dung bucket tfstate
aws s3 rm s3://cloud-sentinel-tfstate-<yourname>/ \
  --recursive \
  --region ap-southeast-1

# Xóa bucket
aws s3 rb s3://cloud-sentinel-tfstate-<yourname> \
  --region ap-southeast-1
```

### Bước 7: Tắt GuardDuty (nếu đã bật)

Nếu bạn đã kích hoạt GuardDuty trong quá trình workshop và không cần dùng nữa:

```bash
# Lấy Detector ID
DETECTOR_ID=$(aws guardduty list-detectors \
  --region ap-southeast-1 \
  --query "DetectorIds[0]" \
  --output text)

# Xóa detector
aws guardduty delete-detector \
  --detector-id $DETECTOR_ID \
  --region ap-southeast-1

echo "GuardDuty detector đã bị xóa."
```

> GuardDuty tính phí theo lưu lượng log phân tích. Nếu bỏ qua bước này, chi phí sẽ tiếp tục phát sinh ngay cả khi không còn dùng hệ thống CloudSentinel.

---

## 2.5.4. Tham chiếu chi phí

Chi phí ước tính cho toàn bộ workshop (tham khảo từ phần 2.1):

| Dịch vụ | Chi phí ước tính/tháng | Ghi chú |
|---------|----------------------|---------|
| Amazon GuardDuty | ~6.90 USD | 6GB VPC Flow Logs |
| Amazon Bedrock (On-Demand) | ~5.00 USD | Phụ thuộc số lần phân tích |
| AWS Lambda | ~0.50 USD | Pay-per-invocation |
| AWS Step Functions | ~0.50 USD | Standard Workflow |
| DynamoDB | ~0.50 USD | On-demand capacity |
| S3 | ~0.10 USD | Lưu báo cáo JSON |
| API Gateway | ~0.10 USD | REST API calls |
| Cognito | ~0.00 USD | Free tier (< 50,000 MAU) |
| Pinecone | ~0.00 USD | Free tier |
| **Tổng cộng** | **~14.46 USD/tháng** | Khi chạy với lưu lượng test |

**Chi phí khi không có sự cố:** Gần bằng 0. Kiến trúc Serverless đảm bảo không phát sinh chi phí compute khi không có finding nào được xử lý. Chỉ GuardDuty là dịch vụ luôn chạy nền và tính phí theo lưu lượng log.

---

## 2.5.5. Tài liệu tham khảo

| Chủ đề | Tài liệu |
|--------|----------|
| AWS Step Functions – waitForTaskToken | [docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html) |
| Amazon Bedrock Agents | [docs.aws.amazon.com/bedrock/latest/userguide/agents.html](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html) |
| Amazon GuardDuty Finding Types | [docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html) |
| Pinecone Python SDK | [docs.pinecone.io/reference/python-sdk](https://docs.pinecone.io/reference/python-sdk) |
| Terraform AWS Provider | [registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) |
| Amazon Titan Embeddings | [docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) |