---
title: "State 1: Parse Finding"
weight: 2
chapter: false
pre: " <b> 2.3.2. </b> "
---

# 2.3.2. State 1 – Parse Finding (Lambda `lambda_parser`)

**Chức năng:** Tiếp nhận finding đầu vào (từ mock event hoặc EventBridge) và chuẩn hóa thành cấu trúc JSON thống nhất để các state sau có thể xử lý.

**Đầu vào:** Raw finding JSON (có cấu trúc GuardDuty với `detail` wrapper). `lambda_parser` sẽ trích xuất các trường cần thiết từ `detail.*` (xem bảng tại mục 2.2.2.2).

**Đầu ra – cấu trúc finding chuẩn (lặp lại từ 2.2.2.2):**

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
