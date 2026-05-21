---
title: "Lớp phản ứng & tương tác (Response & Interaction Layer)"
weight: 4
chapter: false
pre: " <b> 2.4. </b> "
---

# 2.4. Lớp phản ứng & tương tác – Telegram, Cognito, API Gateway & Executor

## Tổng quan

Lớp phản ứng là điểm giao tiếp giữa hệ thống tự động và con người. Thay vì thực thi hành động bảo mật hoàn toàn tự động (vốn tiềm ẩn rủi ro với hạ tầng sản xuất), CloudSentinel áp dụng mô hình **Human-in-the-Loop**: hệ thống phân tích và đề xuất, chuyên viên bảo mật xem xét và quyết định.

Các thành phần cốt lõi của lớp này:

| Thành phần | Dịch vụ AWS / Bên thứ ba | Vai trò |
|------------|--------------------------|---------|
| Telegram Bot | Telegram API | Kênh gửi cảnh báo và nhận phê duyệt |
| Sender | Lambda `lambda_telegram_sender` | Định dạng và gửi tin nhắn |
| Webhook | API Gateway `/webhook` (POST) | Nhận callback từ Telegram |
| Approval Handler | Lambda `lambda_approval_handler` | Xác thực admin, giải phóng Step Function |
| Auth | Amazon Cognito User Pool | Quản lý danh sách admin được phép phê duyệt |
| Executor | Lambda `lambda_executor` | Thực thi hành động ứng phó (simulation) |

Các nội dung chi tiết:
- [Telegram Bot & Sender](./01-telegram-sender/01-telegram-sender.vi.md)
- [API Gateway Webhook](./02-api-gateway/02-api-gateway.vi.md)
- [Cognito Xác thực admin](./03-cognito-auth/03-cognito-auth.vi.md)
- [Lambda Approval Handler](./04-approval-handler/04-approval-handler.vi.md)
- [Lambda Executor](./05-executor/05-executor.vi.md)
- [Luồng dữ liệu phê duyệt](./06-data-flow/06-data-flow.vi.md)
- [Lab: Phê duyệt qua Telegram](./07-lab-approval.vi.md)
- [Kiểm tra xác minh](./08-verification.vi.md)
- [Tổng kết](./09-summary.vi.md)