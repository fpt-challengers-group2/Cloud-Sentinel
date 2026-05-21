---
title: "Những gì đã học"
weight: 2
chapter: false
pre: " <b> 2.5.2. </b> "
---

# 2.5.1.2. Những gì đã học được

**Về kiến trúc SOAR trên AWS:**
- Thiết kế hệ thống hướng sự kiện (Event-Driven) với chi phí gần bằng 0 khi không có sự cố.
- Áp dụng mô hình Human-in-the-Loop với `waitForTaskToken` của Step Functions để kiểm soát hành động tự động.
- Tổ chức IAM theo nguyên tắc least privilege: mỗi Lambda chỉ có đúng quyền cần thiết.

**Về Multi-Agent AI:**
- Supervisor Agent sử dụng action groups để chủ động thu thập ngữ cảnh (agentic loop).
- Advisor Agent (pure LLM) tổng hợp và lập kế hoạch ứng phó có cấu trúc.
- Phân tách vai trò rõ ràng giúp dễ kiểm soát và debug từng Agent độc lập.

**Về RAG với Pinecone:**
- Vector search cho phép truy vấn guideline theo ngữ nghĩa thay vì từ khóa chính xác.
- Amazon Titan Embeddings làm cầu nối giữa text và vector space.

**Về vận hành:**
- Dùng Terraform để quản lý toàn bộ hạ tầng – mọi tài nguyên đều có thể tái tạo hoặc xóa bằng một lệnh.
- CloudWatch Logs tập trung, dễ debug từng Lambda trong pipeline.
- Telegram Bot làm kênh vận hành đơn giản, hiệu quả cho phê duyệt và thông báo.
- Sử dụng DynamoDB và S3 để lưu trữ lịch sử và báo cáo, đảm bảo tính bền vững và dễ truy xuất khi cần thiết.