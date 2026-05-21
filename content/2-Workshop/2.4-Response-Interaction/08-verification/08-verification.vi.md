---
title: "Kiểm tra xác minh"
weight: 8
chapter: false
pre: " <b> 2.4.8. </b> "
---

# 2.4.8. Kiểm tra và xác minh

## Kiểm tra trạng thái execution

```bash
# Liệt kê các execution gần đây
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:5].{Name:name,Status:status,Start:startDate}" \
  --output table
```

## Kiểm tra task_tokens đã được xóa

Sau khi execution hoàn thành (Approve hoặc Reject), token phải được xóa khỏi bảng `task_tokens`:

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count"
```

Kết quả mong đợi: `0` (bảng rỗng sau khi xử lý xong).

## Kiểm tra API Gateway logs

```bash
# Xem access log của API Gateway (nếu đã bật)
aws logs tail /aws/apigateway/cloud-sentinel \
  --region ap-southeast-1 \
  --since 30m \
  --format short
```

## Troubleshooting – Các lỗi thường gặp

| Triệu chứng | Nguyên nhân có thể | Cách xử lý |
|-------------|-------------------|------------|
| Không nhận được tin Telegram | Bot token sai hoặc webhook chưa đăng ký | Kiểm tra `TELEGRAM_TOKEN` trong Lambda env vars; chạy lại lệnh `setWebhook` |
| Bấm Approve nhưng Step Function vẫn Waiting | Webhook URL chưa đúng hoặc API Gateway chưa deploy | Kiểm tra URL webhook bằng `getWebhookInfo`; confirm API Gateway stage `prod` đã deploy |
| Lambda `approval_handler` trả về 403 | `telegram_id` của bạn chưa được map trong Cognito | Chạy lại lệnh `admin-update-user-attributes` với đúng telegram ID |
| `lambda_executor` lỗi permission | IAM role thiếu quyền DynamoDB/S3 | Kiểm tra CloudWatch Logs; đảm bảo Terraform đã apply đầy đủ IAM policy |
| Task token expired | Execution chờ quá 24 giờ | Khởi chạy execution mới; TTL token được thiết kế 24 giờ |
```

### **2.4-response-interaction/09-summary.vi.md**

```markdown
---
title: "Tổng kết lớp phản ứng"
weight: 9
chapter: false
pre: " <b> 2.4.9. </b> "
---

# 2.4.9. Tổng kết lớp phản ứng & tương tác

Sau khi hoàn thành phần này, bạn đã hiểu và thực hành:

- **Human-in-the-Loop:** Cơ chế `waitForTaskToken` của Step Functions cho phép tạm dừng workflow vô thời hạn, chờ quyết định từ con người mà không tốn chi phí compute.
- **Telegram Bot làm kênh vận hành:** Thay vì portal web phức tạp, một bot đơn giản đủ để gửi cảnh báo và nhận phê duyệt với inline keyboard.
- **Cognito làm lớp xác thực nhẹ:** Cho phép quản lý danh sách admin linh hoạt mà không cần triển khai hệ thống auth riêng.
- **Executor simulation:** Cấu trúc code rõ ràng, sẵn sàng chuyển sang hành động thật bằng cách thay nội dung các hàm và cập nhật IAM policy.
- **Kiểm tra và xác minh:** Các bước kiểm tra logs, DynamoDB, S3 giúp đảm bảo toàn bộ luồng hoạt động như thiết kế.