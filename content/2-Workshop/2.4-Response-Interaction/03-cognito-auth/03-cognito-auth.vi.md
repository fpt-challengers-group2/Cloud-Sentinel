---
title: "Cognito Xác thực admin"
weight: 3
chapter: false
pre: " <b> 2.4.3. </b> "
---

# 2.4.3. Amazon Cognito – Xác thực admin

Cognito đảm bảo chỉ những admin đã được đăng ký mới có quyền phê duyệt. Khi `lambda_approval_handler` nhận callback từ Telegram, nó trích xuất `telegram_id` từ payload và đối chiếu với Cognito User Pool trước khi xử lý.

**Cấu hình Cognito:**

| Thuộc tính | Giá trị |
|------------|---------|
| User Pool name | `cloud-sentinel-admin-pool` |
| Schema bổ sung | `custom:telegram_id` (String, mutable) |
| User mặc định | `security-admin` |
| Telegram ID đã map | `8320828979` |

**Cơ chế xác thực trong `lambda_approval_handler`:**

1. Nhận `telegram_id` từ `callback_query.from.id` trong payload Telegram.
2. Gọi Cognito `list_users` với filter `custom:telegram_id = <telegram_id>`.
3. Nếu tìm thấy user → xác thực thành công, tiến hành xử lý token.
4. Nếu không tìm thấy → trả về `403 Forbidden`, bỏ qua callback.

> **Lý do thiết kế:** Xác thực qua Cognito thay vì đơn giản hardcode danh sách admin giúp dễ dàng thêm/bớt admin mà không cần deploy lại Lambda. Chỉ cần cập nhật attribute `custom:telegram_id` cho user trong Cognito Console.
