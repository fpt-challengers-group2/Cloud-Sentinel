---
title: "Step Functions Orchestrator"
weight: 1
chapter: false
pre: " <b> 2.3.1. </b> "
---

# 2.3.1. AWS Step Functions – Orchestrator

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
