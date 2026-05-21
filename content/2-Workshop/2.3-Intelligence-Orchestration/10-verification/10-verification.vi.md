---
title: "Kiểm tra và xác minh"
weight: 10
chapter: false
pre: " <b> 2.3.10. </b> "
---

# 2.3.10. Kiểm tra và xác minh

## Kiểm tra Lambda History – DynamoDB

Sau khi execution hoàn thành, kiểm tra bảng `incident_history`:

**Qua Console:**
- Vào **DynamoDB → Tables → `cloud-sentinel-incident_history` → Explore items** (lưu ý tên bảng có thể khác tùy triển khai).

**Qua CLI:**

```bash
aws dynamodb scan \
  --table-name cloud-sentinel-incident_history \
  --region ap-southeast-1 \
  --query "Items[*].{Type:finding_type.S, Target:target_id.S, Action:action_taken.S}" \
  --output table
```

## Kiểm tra báo cáo – S3

```bash
# Liệt kê các báo cáo đã lưu
aws s3 ls s3://cloud-sentinel-reports-<account-id>/remediations/ \
  --region ap-southeast-1

# Xem nội dung một báo cáo
aws s3 cp s3://cloud-sentinel-reports-<account-id>/remediations/mock-finding-001.json \
  - | python3 -m json.tool
```

## Kiểm tra Lambda logs

Mỗi Lambda có log group riêng với retention 30 ngày. Cú pháp kiểm tra:

```bash
# Thay lambda_parser bằng tên Lambda cần kiểm tra
aws logs tail /aws/lambda/cloud-sentinel-lambda_parser \
  --region ap-southeast-1 \
  --since 30m \
  --format short
