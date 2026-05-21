---
title: "Lưu trữ DynamoDB và S3"
weight: 6
chapter: false
pre: " <b> 2.3.6. </b> "
---

# 2.3.6. Lưu trữ – DynamoDB và S3

## DynamoDB – Bảng `cloud-sentinel-security-incident-history`

Lưu lịch sử toàn bộ sự cố đã xử lý. Cấu trúc bảng:

| Thuộc tính | Loại | Vai trò |
|------------|------|---------|
| `finding_type` | String (PK) | Partition key |
| `target_id` | String (SK) | Sort key |
| `timestamp` | String (GSI) | Lọc theo thời gian |
| `action_taken` | String | Hành động đã thực hiện |
| `severity` | Number | Mức độ nghiêm trọng |
| TTL | 90 ngày | Tự động xóa bản ghi cũ |

## DynamoDB – Bảng `cloud-sentinel-task-tokens`

Lưu task token của Step Functions trong quá trình chờ phê duyệt:

| Thuộc tính | Loại | Vai trò |
|------------|------|---------|
| `finding_id` | String (PK) | Partition key |
| `task_token` | String | Token `waitForTaskToken` của Step Functions |
| TTL | 24 giờ | Tự động xóa token hết hạn |

## S3 – Bucket báo cáo

- **Tên bucket:** `cloud-sentinel-reports-{account_id}`
- **Đường dẫn lưu báo cáo:** `remediations/{finding_id}.json`
- **Nội dung:** JSON bao gồm kế hoạch ứng phó Markdown + metadata sự cố.
