---
title: "Lớp điều phối thông minh (Intelligence & Orchestration Layer)"
weight: 3
chapter: false
pre: " <b> 2.3. </b> "
---

# 2.3. Lớp điều phối thông minh – Step Functions & Bedrock Agents

## Tổng quan

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

Các nội dung chi tiết:
- [Step Functions Orchestrator](./01-stepfunctions/01-stepfunctions.vi.md)
- [State 1: Parse Finding](./02-state-parser/02-state-parser.vi.md)
- [State 2: Check Precedent](./03-state-history/03-state-history.vi.md)
- [State 3: Get Knowledge](./04-state-knowledge/04-state-knowledge.vi.md)
- [State 4: Invoke Agents Pipeline](./05-state-agents/05-state-agents.vi.md)
- [Lưu trữ DynamoDB và S3](./06-storage/06-storage.vi.md)
- [Luồng dữ liệu](./07-data-flow.vi.md)
- [Điều kiện tiên quyết](./08-prerequisites.vi.md)
- [Lab: Chạy pipeline](./09-lab-run.vi.md)
- [Kiểm tra xác minh](./10-verification.vi.md)
- [Tổng kết](./11-summary.vi.md)
