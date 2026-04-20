---
title: "Workshop"
weight: 1
chapter: false
pre: " <b> 2. </b> "
---

# Cloud-Sentinel: Xây dựng hệ thống SOAR dựa trên AI & Serverless

### Tổng quan (Executive Summary)
**Cloud-Sentinel** là một giải pháp bảo mật thế hệ mới áp dụng mô hình **SOAR** (Security Orchestration, Automation, and Response). Hệ thống tận dụng sức mạnh của **Trí tuệ nhân tạo (Generative AI)** và **Hạ tầng hướng sự kiện (Event-Driven Architecture)** trên AWS để tự động hóa quy trình phát hiện và ứng phó sự cố bảo mật với chi phí tối ưu.

Khác với các hệ thống tự động hóa thông thường, Cloud-Sentinel xây dựng một đội ngũ **AI Agents** hoạt động trên **Amazon Bedrock** để thay thế con người trong các bước phân tích và đánh giá sự cố bảo mật. Hệ thống được thiết kế hoàn toàn theo mô hình **Serverless** để đảm bảo chi phí vận hành thấp nhất, chỉ phát sinh khi có sự cố thực tế.

---

### Các cột mốc học tập (Agentic Learning Path)

#### 1. Thiết lập Hạ tầng IaC (Terraform)
Xây dựng "sân chơi" cho các Agent. Sử dụng **Terraform** để định nghĩa VPC, IAM Roles và các quyền hạn cần thiết để AI có thể "đọc" và "hiểu" hạ tầng AWS.

#### 2. Kích hoạt Giác quan (Detection Layer)
Kết nối **GuardDuty** và **EventBridge** để tạo ra các "tín hiệu thần kinh", đánh thức **Supervisor Agent** ngay khi có dấu hiệu xâm nhập.

#### 3. Xây dựng Trí tuệ Agent (Intelligence Layer)
Đây là trọng tâm của workshop. Bạn sẽ học cách:
* Lập trình **Supervisor Agent** để điều phối máy trạng thái (Step Functions).
* Xây dựng **RAG Agent** kết nối với **Pinecone** để truy xuất tri thức bảo mật.
* Cấu hình **Advisor Agent** trên **Bedrock** để soạn thảo giải pháp.

#### 4. Phản ứng & Tương tác (Action Layer)
Thiết lập luồng phê duyệt giữa **Advisor Agent** và người dùng thật qua **Telegram**. Thực thi các lệnh bảo mật (Isolate/Block) thông qua **Lambda Trigger**.

#### 5. Giám sát & Tổng kết
Đánh giá hiệu quả của các Agent qua **CloudWatch** và tối ưu hóa chi phí vận hành (~14.46 USD/tháng) và hướng dẫn cách dọn dẹp tài nguyên.

---

### Kết quả đạt được sau Workshop
* **Kỹ năng IaC:** Thành thạo triển khai hạ tầng phức tạp bằng Terraform.
* **Kỹ năng AI:** Hiểu và thực thi luồng RAG để ứng dụng vào các bài toán thực tế.
* **Tư duy Security:** Làm chủ quy trình vận hành bảo mật hiện đại theo mô hình SOAR.

---

#### Nội dung chi tiết
1. [Module 1: Giới thiệu & Thiết lập](2.1-Workshop-overview/_index.vi.md)
2. [Module 2: Giám sát & Phát hiện](2.2-Detection-Layer/)
3. [Module 3: Trí tuệ & Điều phối](2.3-Intelligence-Orchestration/)
4. [Module 4: Phản ứng & Tương tác](2.4-Response-Interaction/)
5. [Module 5: Giám sát & Tổng kết](2.5-Summary-Cleanup/)