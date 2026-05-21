---
title: "State 4: Invoke Agents Pipeline"
weight: 5
chapter: false
pre: " <b> 2.3.5. </b> "
---

# 2.3.5. State 4 – Invoke Agents Pipeline (Lambda `lambda_advisor`)

Đây là state quan trọng nhất của lớp thông minh. Lambda `lambda_advisor` nhận `intelligence_package` (bao gồm `finding_details`, `historical_context`, `remediation_guidelines`) và gọi tuần tự hai Bedrock Agent.

## Supervisor Agent

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

## Advisor Agent

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
