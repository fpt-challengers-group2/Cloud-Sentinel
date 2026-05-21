---
title: "State 2: Check Precedent"
weight: 3
chapter: false
pre: " <b> 2.3.3. </b> "
---

# 2.3.3. State 2 – Check Precedent (Lambda `lambda_history`)

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
