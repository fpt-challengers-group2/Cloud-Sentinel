---
title: "Kiểm tra kết quả cuối cùng"
weight: 4
chapter: false
pre: " <b> 2.5.4. </b> "
---

# 2.5.2. Kiểm tra kết quả cuối cùng

Trước khi dọn dẹp, thực hiện một lượt kiểm tra toàn bộ để xác nhận pipeline đã chạy đúng.

## Kiểm tra lịch sử sự cố – DynamoDB

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S,Target:target_id.S,Action:action_taken.S,Time:timestamp.S}" \
  --output table
```

Kết quả mong đợi: ít nhất một bản ghi tương ứng với execution đã chạy ở phần Lab 2.4.

## Kiểm tra báo cáo – S3

```bash
# Liệt kê tất cả báo cáo đã lưu
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# Đọc nội dung báo cáo
aws s3 cp \
  s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

## Kiểm tra trạng thái execution

```bash
aws stepfunctions list-executions \
  --state-machine-arn $STATE_MACHINE_ARN \
  --region ap-southeast-1 \
  --query "executions[0:10].{Name:name,Status:status,Start:startDate,Stop:stopDate}" \
  --output table
```

Các execution từ phần Lab phải có trạng thái `SUCCEEDED` (nếu đã Approve) hoặc `FAILED` (nếu đã Reject – đây là hành vi đúng, không phải lỗi).

## Kiểm tra bảng task_tokens

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-task_tokens \
  --region ap-southeast-1 \
  --query "Count" \
  --output text
```

Kết quả mong đợi: `0` (bảng rỗng sau khi execution hoàn thành, token đã được xóa).