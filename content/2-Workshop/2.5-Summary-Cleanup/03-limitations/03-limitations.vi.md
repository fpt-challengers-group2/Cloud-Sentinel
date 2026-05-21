---
title: "Giới hạn và hướng phát triển"
weight: 3
chapter: false
pre: " <b> 2.5.3. </b> "
---

# 2.5.1.3. Giới hạn hiện tại và hướng phát triển

Hệ thống hiện tại đã hoạt động đầy đủ như một proof-of-concept. Một số điểm cần lưu ý khi muốn đưa vào môi trường sản xuất:

| Hạng mục | Trạng thái hiện tại | Hướng phát triển |
|----------|--------------------|--------------------|
| GuardDuty trigger | Dùng mock event thủ công | Triển khai EventBridge rule tự động kích hoạt Step Function khi GuardDuty phát sinh finding |
| Lambda Executor | Simulation (chỉ log) | Implement hành động thật: NACL modification, Security Group update, IAM key revocation |
| Pinecone knowledge base | Đã tạo index, cần seed guideline thực tế | Xây dựng quy trình nhập và cập nhật guideline định kỳ |
| Token cleaner | Lambda đã có, chưa có CloudWatch Schedule | Thêm EventBridge Scheduled Rule chạy `lambda_token_cleaner` mỗi giờ |
| Monitoring | CloudWatch Logs cơ bản | Xây dựng CloudWatch Dashboard tổng hợp + CloudWatch Alarms cho Lambda errors |
| Bedrock Agent alias | Dùng `TSTALIASID` (DRAFT) | Tạo alias PROD sau khi đã kiểm thử ổn định |
