---
title: "Hướng dẫn dọn dẹp hạ tầng"
weight: 5
chapter: false
pre: " <b> 2.5.5. </b> "
---

# 2.5.3. Hướng dẫn dọn dẹp hạ tầng

> **Lưu ý quan trọng:** Thực hiện phần này khi bạn đã hoàn thành toàn bộ workshop và không cần dùng hạ tầng nữa. Lệnh `terraform destroy` sẽ **xóa vĩnh viễn** toàn bộ tài nguyên đã tạo, bao gồm dữ liệu trong DynamoDB và S3.

## Bước 1: Sao lưu dữ liệu cần thiết (tùy chọn)

Nếu muốn giữ lại báo cáo đã tạo trong quá trình workshop:

```bash
# Tải toàn bộ báo cáo về máy local
aws s3 sync \
  s3://cloud-sentinel-reports-<account-id>/remediations/ \
  ./backup-reports/ \
  --region ap-southeast-1

echo "Đã sao lưu $(ls ./backup-reports/ | wc -l) báo cáo."
```

## Bước 2: Xóa dữ liệu trong S3 (bắt buộc trước khi destroy)

Terraform không thể xóa S3 bucket còn chứa object. Cần làm rỗng bucket trước:

```bash
# Xóa toàn bộ object trong bucket
aws s3 rm s3://cloud-sentinel-reports-<account-id>/ \
  --recursive \
  --region ap-southeast-1

echo "Bucket đã được làm rỗng."
```

## Bước 3: Hủy đăng ký Telegram webhook

Trước khi xóa API Gateway, hủy webhook để Telegram ngừng gửi request đến endpoint đã xóa:

```bash
curl -s "https://api.telegram.org/bot<BOT_TOKEN>/deleteWebhook"
```

Kết quả mong đợi: `{"ok":true,"result":true,"description":"Webhook was deleted"}`.

## Bước 4: Chạy terraform destroy

Di chuyển vào thư mục Terraform của dự án:

```bash
cd <đường-dẫn-đến-thư-mục-terraform>

# Xem trước danh sách tài nguyên sẽ bị xóa
terraform plan -destroy

# Xác nhận và thực thi xóa
terraform destroy
```

Terraform sẽ hiển thị danh sách tài nguyên và yêu cầu xác nhận. Gõ `yes` để tiến hành.

## Bước 5: Xác nhận đã dọn dẹp hoàn toàn

Sau khi `terraform destroy` hoàn thành:

```bash
# Xác nhận Lambda đã bị xóa
aws lambda list-functions \
  --region ap-southeast-1 \
  --query "Functions[?starts_with(FunctionName,'cloud-sentinel')].FunctionName" \
  --output table

# Xác nhận Step Function đã bị xóa
aws stepfunctions list-state-machines \
  --region ap-southeast-1 \
  --query "stateMachines[?contains(name,'cloud-sentinel')].name" \
  --output text

# Xác nhận DynamoDB tables đã bị xóa
aws dynamodb list-tables \
  --region ap-southeast-1 \
  --query "TableNames[?starts_with(@,'cloud-sentinel')]" \
  --output text
```

Kết quả mong đợi: tất cả các lệnh trả về kết quả rỗng.

## Bước 6: Xóa S3 Backend (tùy chọn)

Nếu bạn đã tạo S3 bucket để lưu `terraform.tfstate` theo hướng dẫn ở mục 2.1, đây là lúc xóa nó nếu không cần dùng cho dự án khác:

```bash
# Xóa toàn bộ nội dung bucket tfstate
aws s3 rm s3://cloud-sentinel-tfstate-<yourname>/ \
  --recursive \
  --region ap-southeast-1

# Xóa bucket
aws s3 rb s3://cloud-sentinel-tfstate-<yourname> \
  --region ap-southeast-1
```

## Bước 7: Tắt GuardDuty (nếu đã bật)

Nếu bạn đã kích hoạt GuardDuty trong quá trình workshop và không cần dùng nữa:

```bash
# Lấy Detector ID
DETECTOR_ID=$(aws guardduty list-detectors \
  --region ap-southeast-1 \
  --query "DetectorIds[0]" \
  --output text)

# Xóa detector
aws guardduty delete-detector \
  --detector-id $DETECTOR_ID \
  --region ap-southeast-1

echo "GuardDuty detector đã bị xóa."
```

> GuardDuty tính phí theo lưu lượng log phân tích. Nếu bỏ qua bước này, chi phí sẽ tiếp tục phát sinh ngay cả khi không còn dùng hệ thống CloudSentinel.
