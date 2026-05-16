---
title: "Lớp điều phối thông minh (Intelligence & Orchestration Layer)"
weight: 3
chapter: false
pre: " <b> 2.3. </b> "
---

# 2.3. Lớp điều phối thông minh – Step Functions & Bedrock Agents

## 2.3.1. Tổng quan

Lớp điều phối thông minh là trung tâm xử lý của CloudSentinel. Sau khi lớp phát hiện (2.2) cung cấp một finding đã được chuẩn hóa, lớp này tiếp nhận và thực hiện ba nhiệm vụ theo thứ tự:

1. **Thu thập ngữ cảnh** – kiểm tra lịch sử sự cố (DynamoDB) và truy vấn hướng dẫn xử lý (Pinecone RAG).
2. **Phân tích thông minh** – đưa ngữ cảnh vào pipeline hai Bedrock Agent để tổng hợp và đưa ra kế hoạch ứng phó.
3. **Chuyển giao kết quả** – gửi kế hoạch sang lớp phản ứng (2.4) để phê duyệt và thực thi.

Các thành phần cốt lõi của lớp này bao gồm:

| Thành phần | Dịch vụ AWS | Vai trò |
|------------|-------------|---------|
| Orchestrator | AWS Step Functions | Điều phối toàn bộ luồng xử lý (6 states) |
| Parser | Lambda `lambda_parser` | Chuẩn hóa finding từ GuardDuty |
| History | Lambda `lambda_history` | Kiểm tra tiền lệ sự cố trong DynamoDB |
| Knowledge | Lambda `lambda_knowledge` | Truy vấn guideline từ Pinecone (RAG) |
| Agents Pipeline | Lambda `lambda_advisor` + Bedrock | Phân tích và lập kế hoạch ứng phó |
| Storage | DynamoDB + S3 | Lưu lịch sử và báo cáo |

---

## 2.3.2. Thiết kế và triển khai

### 2.3.2.1. AWS Step Functions – Orchestrator

State machine `cloud-sentinel-orchestrator` điều phối toàn bộ pipeline theo **6 states** tuần tự:

```
Parse Finding
    ↓
Check Precedent
    ↓
Get Knowledge
    ↓
Invoke Agents Pipeline
    ↓
Send Telegram And Wait   ← waitForTaskToken (chờ phê duyệt)
    ↓
Execute Remediation
```

> **Lưu ý thiết kế:** Bản thiết kế gốc mô tả 7 states (tách riêng "Send Telegram" và "Wait for approval"). Trong triển khai hiện tại, hai bước này được gộp thành một state duy nhất `Send Telegram And Wait` sử dụng cơ chế `waitForTaskToken` của Step Functions – vừa gửi tin nhắn vừa tự động tạm dừng cho đến khi nhận được callback phê duyệt từ webhook. Về mặt chức năng, hành vi hoàn toàn tương đương.

Cấu hình chính của state machine:

| Thuộc tính | Giá trị |
|------------|---------|
| Tên | `cloud-sentinel-orchestrator` |
| Logging | `ALL` (ghi log tất cả transitions) |
| Region | `ap-southeast-1` |
| Loại | Standard Workflow |

---

### 2.3.2.2. State 1 – Parse Finding (Lambda `lambda_parser`)

**Chức năng:** Tiếp nhận finding đầu vào (từ mock event hoặc EventBridge) và chuẩn hóa thành cấu trúc JSON thống nhất để các state sau có thể xử lý.

**Đầu vào:** Raw finding JSON (có thể từ GuardDuty EventBridge hoặc mock event).

**Đầu ra – cấu trúc finding chuẩn:**

| Trường | Ý nghĩa | Ví dụ |
|--------|---------|-------|
| `finding_id` | UUID của sự cố | `"mock-finding-001"` |
| `finding_type` | Loại tấn công | `"Recon:EC2/Portscan"` |
| `severity` | Mức độ nghiêm trọng (0.1–8.9) | `5.0` |
| `resource_type` | Loại tài nguyên bị ảnh hưởng | `"EC2"` |
| `target_id` | ID tài nguyên | `"i-0123456789abcdef0"` |
| `region` | Vùng AWS | `"ap-southeast-1"` |
| `attacker_ip` | IP tấn công (nếu có) | `"203.0.113.45"` |
| `attacker_location` | Vị trí địa lý IP | `"Hanoi, Vietnam"` |
| `title` | Tiêu đề sự cố | `"Port scan from malicious IP"` |
| `description` | Mô tả chi tiết | `"EC2 instance was probed..."` |
| `timestamp` | Thời gian phát sinh (ISO 8601) | `"2026-04-26T10:00:00Z"` |

---

### 2.3.2.3. State 2 – Check Precedent (Lambda `lambda_history`)

**Chức năng:** Truy vấn bảng DynamoDB `incident_history` để xác định liệu kiểu tấn công này đã từng xảy ra trên tài nguyên đó chưa, trong vòng 90 ngày gần nhất.

**Cơ chế truy vấn:**
- **Partition Key:** `finding_type` (loại tấn công)
- **Sort Key:** `target_id` (ID tài nguyên bị tấn công)
- **GSI:** `timestamp` (lọc theo thời gian, TTL 90 ngày)

**Đầu ra:**

| Trường | Ý nghĩa |
|--------|---------|
| `has_precedent` | `true`/`false` – có tiền lệ không |
| `previous_action` | Hành động đã thực hiện lần trước |
| `recurrence_count` | Số lần lặp lại trong 90 ngày |

Kết quả này được đưa vào `historical_context` và truyền xuống cho Bedrock Agent phân tích, giúp Agent nhận diện được các mẫu tấn công lặp lại.

---

### 2.3.2.4. State 3 – Get Knowledge (Lambda `lambda_knowledge`)

**Chức năng:** Truy vấn Pinecone Vector Database để lấy các hướng dẫn xử lý (guideline) phù hợp nhất với loại sự cố hiện tại. Đây là lớp RAG (Retrieval-Augmented Generation) của hệ thống.

**Cơ chế hoạt động:**
1. Lambda nhận `finding_type` và `description` làm input.
2. Gọi **Amazon Titan Text Embeddings** để chuyển đổi text thành vector.
3. Truy vấn Pinecone index `cloud-sentinel-index` lấy **top 3** guideline có độ tương đồng cao nhất.
4. Trả về danh sách guideline dưới dạng text để đưa vào prompt của Bedrock Agent.

**Pinecone index hiện tại:**
- Index name: `cloud-sentinel-index`
- Trạng thái: Đã tạo, chưa có dữ liệu guideline (record count = 0).
- Xem hướng dẫn seed dữ liệu tại phần Lab (2.3.3, Bước 2).

---

### 2.3.2.5. State 4 – Invoke Agents Pipeline (Lambda `lambda_advisor`)

Đây là state quan trọng nhất của lớp thông minh. Lambda `lambda_advisor` nhận `intelligence_package` (bao gồm `finding_details`, `historical_context`, `remediation_guidelines`) và gọi tuần tự hai Bedrock Agent.

#### Supervisor Agent

| Thuộc tính | Giá trị |
|------------|---------|
| Model | `anthropic.claude-3-5-sonnet-20240620-v1:0` |
| Alias | `TSTALIASID` (DRAFT – tránh vòng lặp Terraform) |
| Action Groups | 3 nhóm: Parser, History, Knowledge |

Supervisor Agent có thể chủ động gọi lại các Lambda (parser, history, knowledge) thông qua action groups nếu cần bổ sung thông tin. Đây là cơ chế **agentic loop** của Bedrock: Agent tự quyết định khi nào cần thu thập thêm dữ liệu trước khi đưa ra phán đoán.

**Đầu ra của Supervisor Agent** là một JSON có cấu trúc chuẩn:

```json
{
  "executive_summary": "Mô tả tổng quan sự cố",
  "target": { "id": "...", "type": "...", "region": "..." },
  "attacker": { "ip": "...", "location": "..." },
  "historical_analysis": {
    "has_precedent": true,
    "recurrence_count": 2,
    "previous_action": "block_ip"
  },
  "remediation_guidelines": ["Guideline 1", "Guideline 2", "..."]
}
```

#### Advisor Agent

| Thuộc tính | Giá trị |
|------------|---------|
| Model | `anthropic.claude-3-5-sonnet-20240620-v1:0` |
| Alias | `TSTALIASID` |
| Action Groups | Không có (pure LLM) |

Advisor Agent nhận JSON từ Supervisor và tổng hợp thành **kế hoạch ứng phó dạng Markdown** với 7 phần:

1. Executive Summary
2. Threat Details
3. Immediate Remediation (hành động cụ thể: `block_ip`, `isolate_instance`, `revoke_keys`)
4. Investigation Steps
5. Long-term Recommendations
6. Verification Steps
7. Rollback Plan

Báo cáo Markdown này được lưu vào S3 và gửi đến admin qua Telegram để phê duyệt.

---

### 2.3.2.6. Lưu trữ – DynamoDB và S3

#### DynamoDB – Bảng `cloud-sentinel-security-incident-history`

Lưu lịch sử toàn bộ sự cố đã xử lý. Cấu trúc bảng:

| Thuộc tính | Loại | Vai trò |
|------------|------|---------|
| `finding_type` | String (PK) | Partition key |
| `target_id` | String (SK) | Sort key |
| `timestamp` | String (GSI) | Lọc theo thời gian |
| `action_taken` | String | Hành động đã thực hiện |
| `severity` | Number | Mức độ nghiêm trọng |
| TTL | 90 ngày | Tự động xóa bản ghi cũ |

#### DynamoDB – Bảng `cloud-sentinel-task-tokens`

Lưu task token của Step Functions trong quá trình chờ phê duyệt:

| Thuộc tính | Loại | Vai trò |
|------------|------|---------|
| `finding_id` | String (PK) | Partition key |
| `task_token` | String | Token `waitForTaskToken` của Step Functions |
| TTL | 24 giờ | Tự động xóa token hết hạn |

#### S3 – Bucket báo cáo

- **Tên bucket:** `cloud-sentinel-reports-{account_id}`
- **Đường dẫn lưu báo cáo:** `remediations/{finding_id}.json`
- **Nội dung:** JSON bao gồm kế hoạch ứng phó Markdown + metadata sự cố.

---

## 2.3.3. Luồng dữ liệu

Sơ đồ dưới đây mô tả hành trình của dữ liệu qua 4 state đầu tiên trong lớp điều phối:

```
[Mock Event / GuardDuty]
        │
        ▼
┌─────────────────────┐
│  Parse Finding      │  lambda_parser
│  → finding_details  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Check Precedent    │  lambda_history → DynamoDB
│  → historical_ctx   │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Get Knowledge      │  lambda_knowledge → Pinecone
│  → guidelines       │
└────────┬────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│  Invoke Agents Pipeline                  │
│  lambda_advisor:                         │
│    1. Supervisor Agent (agentic loop)    │
│       → structured JSON analysis         │
│    2. Advisor Agent (pure LLM)           │
│       → Markdown remediation plan        │
│  → Lưu báo cáo vào S3                   │
└────────┬─────────────────────────────────┘
         │
         ▼
  [Send Telegram And Wait] → Lớp 2.4
```

---

## 2.3.4. Điều kiện tiên quyết trước khi thực hành

Trước khi thực hiện Lab, đảm bảo các mục sau đã sẵn sàng:

### A. Kích hoạt Bedrock Model Access

Bedrock Agent sử dụng model `anthropic.claude-3-5-sonnet-20240620-v1:0`. Model này cần được kích hoạt thủ công trong AWS Console trước khi có thể gọi.

**Các bước thực hiện:**

1. Vào **AWS Console** → chọn region **Singapore (`ap-southeast-1`)**.
2. Tìm kiếm và mở dịch vụ **Amazon Bedrock**.
3. Trong menu trái, chọn **Model access** (hoặc **Bedrock configurations → Model access**).
4. Chọn tab **Available to request**.
5. Tìm **Anthropic → Claude 3.5 Sonnet**, đánh dấu chọn.
6. Nhấn **Request model access** → Xác nhận.
7. Chờ trạng thái chuyển sang **Access granted** (thường dưới 5 phút).

> **Lưu ý:** Nếu bỏ qua bước này, Lambda `lambda_advisor` sẽ nhận lỗi `AccessDeniedException` khi gọi Bedrock API.

### B. Seed dữ liệu Pinecone (Guideline RAG)

Index `cloud-sentinel-index` đã được tạo nhưng chưa có dữ liệu. Không có guideline, Lambda `lambda_knowledge` sẽ trả về danh sách rỗng và Bedrock Agent sẽ phải tự suy luận mà không có ngữ cảnh từ knowledge base.

**Tùy chọn 1 – Seed dữ liệu mẫu (khuyến nghị cho workshop):**

Tạo file `seed_pinecone.py` với nội dung sau:

```python
from pinecone import Pinecone
import boto3
import json

# Cấu hình
PINECONE_API_KEY = "<your-pinecone-api-key>"
INDEX_NAME = "cloud-sentinel-index"
REGION = "ap-southeast-1"

# Khởi tạo Pinecone
pc = Pinecone(api_key=PINECONE_API_KEY)
index = pc.Index(INDEX_NAME)

# Khởi tạo Bedrock để tạo embedding
bedrock = boto3.client("bedrock-runtime", region_name=REGION)

def get_embedding(text):
    response = bedrock.invoke_model(
        modelId="amazon.titan-embed-text-v1",
        body=json.dumps({"inputText": text})
    )
    return json.loads(response["body"].read())["embedding"]

# Dữ liệu guideline mẫu
guidelines = [
    {
        "id": "guide-portscan-001",
        "text": "Recon:EC2/Portscan – Khi phát hiện port scan từ IP bên ngoài: (1) Block IP tại Network ACL, (2) Review Security Group rules, (3) Enable VPC Flow Logs nếu chưa có, (4) Kiểm tra các port đang mở không cần thiết."
    },
    {
        "id": "guide-ssh-bruteforce-001",
        "text": "UnauthorizedAccess:EC2/SSHBruteForce – Khi phát hiện SSH brute force: (1) Block IP nguồn tại NACL, (2) Rotate SSH key pairs, (3) Xem xét cấu hình SSH chỉ cho phép key-based auth, (4) Enable fail2ban hoặc tương đương."
    },
    {
        "id": "guide-iam-anomaly-001",
        "text": "UnauthorizedAccess:IAMUser/ConsoleLoginSuccess – Khi phát hiện đăng nhập IAM bất thường: (1) Revoke access keys ngay lập tức, (2) Kiểm tra CloudTrail 24 giờ trước, (3) Review IAM policies của user bị ảnh hưởng, (4) Enable MFA bắt buộc."
    }
]

# Upload lên Pinecone
vectors = []
for g in guidelines:
    embedding = get_embedding(g["text"])
    vectors.append({
        "id": g["id"],
        "values": embedding,
        "metadata": {"text": g["text"]}
    })

index.upsert(vectors=vectors)
print(f"Đã seed {len(vectors)} guideline vào Pinecone index '{INDEX_NAME}'.")
```

Chạy script:

```bash
pip install pinecone boto3
python seed_pinecone.py
```

**Tùy chọn 2 – Bỏ qua (tiếp tục Lab không có guideline):** Lambda `lambda_knowledge` sẽ trả về mảng rỗng. Bedrock Agent vẫn hoạt động nhưng kế hoạch ứng phó sẽ không tham chiếu được guideline cụ thể.

---

## 2.3.5. Hướng dẫn thực hành (Lab)

### Bước 1: Xác nhận hạ tầng đã sẵn sàng

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

### Bước 2: Tạo mock event

Tạo file `mock_finding.json`:

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

Bạn có thể thay đổi các trường `finding_type`, `target_id`, `attacker_ip` để kiểm tra các loại sự cố khác nhau.

### Bước 3: Lấy ARN của State Machine

```bash
STATE_MACHINE_ARN=$(aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel-orchestrator')].stateMachineArn" \
  --output text)

echo "ARN: $STATE_MACHINE_ARN"
```

### Bước 4: Khởi chạy execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn $STATE_MACHINE_ARN \
  --name "test-run-$(date +%s)" \
  --input file://mock_finding.json \
  --region ap-southeast-1
```

Lệnh sẽ trả về `executionArn`. Lưu lại giá trị này để theo dõi trạng thái.

### Bước 5: Quan sát luồng xử lý

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

---

## 2.3.6. Kiểm tra và xác minh

### Kiểm tra Lambda History – DynamoDB

Sau khi execution hoàn thành, kiểm tra bảng `incident_history`:

**Qua Console:**
- Vào **DynamoDB → Tables → `cloud-sentinel-incident_history` → Explore items**.

**Qua CLI:**

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S, Target:target_id.S, Action:action_taken.S}" \
  --output table
```

### Kiểm tra báo cáo – S3

```bash
# Liệt kê các báo cáo đã lưu
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# Xem nội dung một báo cáo
aws s3 cp s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

### Kiểm tra Lambda logs

Mỗi Lambda có log group riêng với retention 30 ngày. Cú pháp kiểm tra:

```bash
# Thay lambda_parser bằng tên Lambda cần kiểm tra
aws logs tail /aws/lambda/cloud-sentinel-lambda_parser \
  --region ap-southeast-1 \
  --since 30m \
  --format short
```

---

## 2.3.7. Tổng kết lớp điều phối thông minh

Sau khi hoàn thành phần này, bạn đã hiểu và thực hành:

- **Kiến trúc pipeline:** Luồng dữ liệu 6 states trong Step Functions từ finding đến kế hoạch ứng phó.
- **Thiết kế Multi-Agent:** Supervisor Agent thu thập ngữ cảnh qua action groups, Advisor Agent tổng hợp và lập kế hoạch ứng phó.
- **RAG với Pinecone:** Lambda `lambda_knowledge` kết hợp vector search để cung cấp guideline phù hợp cho AI Agent.
- **Lưu trữ kết quả:** DynamoDB ghi lịch sử sự cố, S3 lưu báo cáo remediation dạng JSON.
