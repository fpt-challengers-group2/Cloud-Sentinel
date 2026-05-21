---
title: "Lambda Executor"
weight: 5
chapter: false
pre: " <b> 2.4.5. </b> "
---

# 2.4.5. Lambda `lambda_executor`

**Chức năng:** Sau khi admin Approve, Step Functions chuyển sang state `Execute Remediation` và gọi `lambda_executor` với thông tin sự cố và hành động đề xuất.

**Các hành động được hỗ trợ (hiện tại ở chế độ simulation):**

| Hành động | Hàm | Mô tả |
|-----------|-----|-------|
| `block_ip` | `block_ip_nacl()` | Block IP nguồn tại Network ACL |
| `isolate_instance` | `isolate_instance()` | Cô lập EC2 instance khỏi network |
| `revoke_keys` | `revoke_iam_keys()` | Thu hồi IAM access keys |

**Chế độ simulation hiện tại:** Các hàm trên chỉ ghi log `[SIMULATION]` và trả về `status: "simulated_success"` mà không thực sự gọi AWS API để thay đổi tài nguyên. Đây là thiết kế phù hợp cho môi trường workshop – an toàn để chạy lặp lại nhiều lần.

Ví dụ output của một hành động simulation:

```json
{
  "action": "block_ip",
  "ip": "203.0.113.45",
  "status": "simulated_success",
  "timestamp": "2026-04-26T10:05:32Z"
}
```

**Sau khi thực thi**, `lambda_executor` thực hiện hai thao tác ghi:

1. **DynamoDB `incident_history`** – ghi bản ghi sự cố với `action_taken`, `severity`, `timestamp`.
2. **S3** – cập nhật file `remediations/{finding_id}.json` với kết quả thực thi.
IAM permissions của Lambda này bao gồm: `dynamodb:PutItem`, `s3:PutObject`, và các quyền tương ứng nếu thực hiện hành động thật (ví dụ: `ec2:CreateNetworkAclEntry` cho block IP).