---
title: "Kiểm tra và xác minh"
weight: 5
chapter: false
pre: " <b> 2.2.5. </b> "
---

# 2.2.5. Kiểm tra và xác minh

- **CloudWatch Logs**: Mỗi Lambda (parser, history, knowledge, advisor, telegram_sender, executor) có log group riêng. Bạn có thể kiểm tra log để debug.
- **DynamoDB**: Sau khi execution hoàn thành, bảng `incident_history` sẽ có một bản ghi mới (nếu executor ghi thành công).
- **S3**: Báo cáo remediation được lưu dưới dạng JSON trong bucket `cloud-sentinel-reports-<account-id>`.
