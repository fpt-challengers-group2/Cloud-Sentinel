---
title: "Đề Án"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
<img src="/images/logo.jpg" class="img-responsive" style="max-width:300px; display:block; margin:auto;">

# AWS FIRST CLOUD AI JOURNEY – PROJECT PLAN
# Cloud-Sentinel – SOAR System for Critical Infrastructure on AWS

---

## 0. PHỤ LỤC

- [AWS FIRST CLOUD AI JOURNEY – PROJECT PLAN](#aws-first-cloud-ai-journey--project-plan)
- [Cloud-Sentinel – SOAR System for Critical Infrastructure on AWS](#cloud-sentinel--soar-system-for-critical-infrastructure-on-aws)
  - [0. PHỤ LỤC](#0-phụ-lục)
  - [1. BỐI CẢNH VÀ ĐỘNG LỰC](#1-bối-cảnh-và-động-lực)
    - [1.1 Tóm tắt](#11-tóm-tắt)
      - [1. Bối cảnh khách hàng \& Vấn đề](#1-bối-cảnh-khách-hàng--vấn-đề)
      - [2. Mục tiêu kinh doanh \& Kỹ thuật](#2-mục-tiêu-kinh-doanh--kỹ-thuật)
      - [3. Trường hợp sử dụng](#3-trường-hợp-sử-dụng)
      - [4. Tóm tắt dịch vụ tư vấn](#4-tóm-tắt-dịch-vụ-tư-vấn)
    - [1.2 Tiêu chí thành công của dự án](#12-tiêu-chí-thành-công-của-dự-án)
    - [1.3 Giả định và tiền đề](#13-giả-định-và-tiền-đề)
  - [2. KIẾN TRÚC GIẢI PHÁP](#2-kiến-trúc-giải-pháp)
    - [2.1 Sơ đồ kiến trúc kỹ thuật](#21-sơ-đồ-kiến-trúc-kỹ-thuật)
    - [2.2 Kế hoạch kỹ thuật](#22-kế-hoạch-kỹ-thuật)
    - [2.3 Kế hoạch dự án](#23-kế-hoạch-dự-án)
    - [2.4 Các yếu tố bảo mật](#24-các-yếu-tố-bảo-mật)
  - [3. HOẠT ĐỘNG VÀ KẾT QUẢ BÀN GIAO](#3-hoạt-động-và-kết-quả-bàn-giao)
    - [3.1 Activities and deliverables](#31-activities-and-deliverables)
    - [3.2 Nằm ngoài dự án](#32-nằm-ngoài-dự-án)
    - [3.3 Quy trình đưa vào vận hành](#33-quy-trình-đưa-vào-vận-hành)
  - [4. PHÂN TÍCH CHI PHÍ THEO DỊCH VỤ](#4-phân-tích-chi-phí-theo-dịch-vụ)
  - [5. ĐỘI NGŨ](#5-đội-ngũ)
    - [Chịu trách nhiệm tổng thể](#chịu-trách-nhiệm-tổng-thể)
    - [Các bên liên quan](#các-bên-liên-quan)
    - [Đại diện hỗ trợ](#đại-diện-hỗ-trợ)
    - [Đội ngũ thực hiện dự án](#đội-ngũ-thực-hiện-dự-án)
  - [6. NGUỒN LỰC VÀ ƯỚC TÍNH CHI PHÍ](#6-nguồn-lực-và-ước-tính-chi-phí)
    - [Định mức đơn giá nhân sự](#định-mức-đơn-giá-nhân-sự)
    - [Ước tính giờ công \& chi phí theo giai đoạn](#ước-tính-giờ-công--chi-phí-theo-giai-đoạn)
    - [Phân bổ đóng góp chi phí](#phân-bổ-đóng-góp-chi-phí)
  - [7. NGHIỆM THU](#7-nghiệm-thu)
    - [7.1. Gói bàn giao](#71-gói-bàn-giao)
    - [7.2. Quy trình nghiệm thu](#72-quy-trình-nghiệm-thu)
    - [7.3. Điều kiện từ chối](#73-điều-kiện-từ-chối)

---

## 1. BỐI CẢNH VÀ ĐỘNG LỰC

### 1.1 Tóm tắt

#### 1. Bối cảnh khách hàng & Vấn đề

* **Vấn đề:** Các tổ chức vận hành hạ tầng trọng yếu trên AWS (Ngân hàng, Tài chính, Fintech) đang đối mặt với khối lượng cảnh báo bảo mật ngày càng lớn từ các dịch vụ như GuardDuty, CloudTrail và VPC Flow Logs. Quy trình phân tích và ứng phó hiện tại chủ yếu là thủ công: chuyên viên bảo mật phải đọc từng finding, tra cứu lịch sử, soạn thảo kế hoạch và thực thi hành động – dẫn đến thời gian phản ứng chậm, dễ bỏ sót và gánh nặng vận hành lớn. Ngoài ra, tỷ lệ cảnh báo giả (false positive) cao làm giảm hiệu quả và niềm tin vào hệ thống giám sát.
* **Động lực:** Nhu cầu cấp thiết là xây dựng một hệ thống SOAR (Security Orchestration, Automation and Response) có khả năng tự động hóa toàn bộ quy trình từ phát hiện đến phản ứng, đồng thời ứng dụng trí tuệ nhân tạo để loại bỏ cảnh báo giả và nâng cao chất lượng phân tích – trong khi vẫn giữ con người là người ra quyết định cuối cùng.

#### 2. Mục tiêu kinh doanh & Kỹ thuật

<table>
  <thead>
    <tr>
      <th style="width: 20%;">Loại Mục tiêu</th>
      <th style="width: 70%;">Chi tiết</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Kinh doanh</strong></td>
      <td>
        - Giảm thời gian phản ứng sự cố (Mean Time to Respond) từ giờ xuống phút.<br>
        - Giảm tải vận hành cho đội bảo mật thông qua tự động hóa phân tích và đề xuất hành động.<br>
        - Tối ưu chi phí vận hành: kiến trúc Serverless đảm bảo chi phí gần bằng 0 khi không có sự cố.
      </td>
    </tr>
    <tr>
      <td><strong>Kỹ thuật</strong></td>
      <td>
        - <strong>Phát hiện thông minh:</strong> Tích hợp Amazon GuardDuty phân tích VPC Flow Logs, DNS logs và CloudTrail events.<br>
        - <strong>Đánh giá chéo Multi-Agent:</strong> Supervisor Agent và Advisor Agent (Amazon Bedrock – Claude 3.5 Sonnet) phối hợp để loại bỏ cảnh báo giả và tạo kế hoạch ứng phó có cấu trúc.<br>
        - <strong>RAG Knowledge Base:</strong> Pinecone Vector Database cung cấp ngữ cảnh guideline cho AI Agent, giảm ảo giác (hallucination).<br>
        - <strong>Human-in-the-Loop:</strong> Chuyên viên phê duyệt qua Telegram trước khi hành động được thực thi – kiểm soát rủi ro tự động hóa.<br>
        - <strong>IaC hoàn toàn:</strong> Toàn bộ hạ tầng được quản lý bằng Terraform, tái sử dụng và kiểm soát được.
      </td>
    </tr>
  </tbody>
</table>

#### 3. Trường hợp sử dụng

Các luồng chính mà hệ thống Cloud-Sentinel xử lý:

* **Phát hiện và phân tích tự động:** Khi GuardDuty phát sinh finding (ví dụ: SSH Brute Force, Port Scan, IAM anomaly), hệ thống tự động kích hoạt pipeline phân tích mà không cần can thiệp thủ công.
* **Tra cứu tiền lệ:** Kiểm tra DynamoDB xem kiểu tấn công này đã xảy ra trên tài nguyên đó chưa trong 90 ngày gần nhất, cung cấp ngữ cảnh lịch sử cho AI.
* **Lập kế hoạch ứng phó có cấu trúc:** Advisor Agent tạo kế hoạch 7 phần bao gồm hành động ngay lập tức, bước điều tra, khuyến nghị dài hạn và kế hoạch rollback.
* **Phê duyệt qua Telegram:** Chuyên viên nhận cảnh báo kèm kế hoạch ứng phó, bấm Approve/Reject trực tiếp trên điện thoại.
* **Thực thi hành động:** Sau khi được phê duyệt, hệ thống thực thi các hành động bảo vệ (block IP, isolate instance, revoke IAM keys) và lưu kết quả vào S3 và DynamoDB.

#### 4. Tóm tắt dịch vụ tư vấn

Dịch vụ chuyên nghiệp được cung cấp để đạt các mục tiêu trên bao gồm:

* **Thiết kế kiến trúc Event-Driven SOAR:** Xây dựng pipeline 6 states trên AWS Step Functions kết hợp với Multi-Agent AI trên Amazon Bedrock.
* **Triển khai Infrastructure as Code:** Toàn bộ hạ tầng (Lambda, Step Functions, Bedrock Agents, DynamoDB, S3, API Gateway, Cognito) được định nghĩa và quản lý bằng Terraform.
* **Tích hợp RAG Pipeline:** Thiết lập luồng Pinecone Vector Search với Amazon Titan Embeddings để cung cấp ngữ cảnh guideline cho AI Agent.
* **Tối ưu chi phí Serverless:** Thiết kế kiến trúc Pay-as-you-go, chi phí duy trì khi không có sự cố gần bằng 0.

---

### 1.2 Tiêu chí thành công của dự án

Sự thành công của dự án được đánh giá dựa trên các tiêu chí sau:

* **Pipeline hoạt động end-to-end:** Từ mock finding đầu vào đến tin nhắn Telegram phê duyệt và kết quả thực thi được ghi vào S3/DynamoDB – toàn bộ 6 states của Step Function hoàn thành không lỗi.
* **Multi-Agent AI cho ra kết quả có cấu trúc:** Supervisor Agent và Advisor Agent phối hợp tạo ra kế hoạch ứng phó đủ 7 phần với nội dung phù hợp với loại sự cố đầu vào.
* **Human-in-the-Loop hoạt động:** Chuyên viên nhận được tin nhắn Telegram, bấm Approve/Reject, và Step Function phản ứng đúng (tiếp tục hoặc dừng pipeline).
* **Tối ưu chi phí:** Tổng chi phí vận hành ở mức ~14.46 USD/tháng; chi phí khi không có sự cố gần bằng 0.
* **IaC hoàn chỉnh:** Toàn bộ hạ tầng được khởi tạo và xóa bằng `terraform apply` / `terraform destroy` mà không cần thao tác thủ công ngoài Console.
* **Khả năng mở rộng:** Kiến trúc Serverless tự động scale khi số lượng finding tăng mà không cần thay đổi cấu hình.

---

### 1.3 Giả định và tiền đề

Dự án được thực hiện dựa trên các giả định và ràng buộc sau:

* **Phụ thuộc dịch vụ AWS:** Dự án phụ thuộc vào tính sẵn sàng của Amazon Bedrock (model access), GuardDuty, Step Functions và các dịch vụ liên quan tại region `ap-southeast-1`.
* **Model access Bedrock:** Giả định rằng người triển khai đã kích hoạt quyền truy cập model `claude-3-5-sonnet` trong AWS Console trước khi chạy pipeline.
* **Pinecone knowledge base:** Hệ thống hoạt động ổn định ngay cả khi Pinecone index chưa có dữ liệu guideline (Agent sẽ tự suy luận từ ngữ cảnh sự cố). Tuy nhiên, chất lượng kế hoạch ứng phó được cải thiện đáng kể khi có guideline.
* **Phạm vi thực thi:** Lambda Executor hiện ở chế độ simulation – không thực sự thay đổi tài nguyên AWS. Đây là thiết kế có chủ ý để an toàn trong môi trường workshop và demo.
* **EventBridge trigger:** Trong phiên bản hiện tại, pipeline được kích hoạt thủ công bằng mock event thay vì tự động từ GuardDuty qua EventBridge. Đây là hạn chế được ghi nhận để hoàn thiện sau.
* **Rủi ro AI:** Mặc dù sử dụng RAG và cấu trúc Multi-Agent để giảm thiểu ảo giác, kế hoạch ứng phó của AI vẫn cần được chuyên viên bảo mật xem xét trước khi thực thi – được đảm bảo bởi cơ chế Human-in-the-Loop.

---

## 2. KIẾN TRÚC GIẢI PHÁP

### 2.1 Sơ đồ kiến trúc kỹ thuật

Kiến trúc đề xuất là **Serverless Event-Driven** hoàn toàn trên AWS, tổ chức thành 4 lớp: Detection, Intelligence & Orchestration, Response & Interaction, và Storage.

![Architecture Diagram](/images/High_Level_System_Architecture.gif)

**Các thành phần chính:**

* **Detection Layer:** Amazon GuardDuty phân tích VPC Flow Logs, DNS logs, CloudTrail events → phát sinh finding → Lambda `lambda_parser` chuẩn hóa.
* **Orchestration Layer:** AWS Step Functions (6 states) điều phối toàn bộ pipeline từ parse đến execute.
* **Intelligence Layer:** Lambda `lambda_history` (DynamoDB), Lambda `lambda_knowledge` (Pinecone RAG), Lambda `lambda_advisor` gọi tuần tự Supervisor Agent và Advisor Agent trên Amazon Bedrock.
* **Response Layer:** Lambda `lambda_telegram_sender` gửi kế hoạch qua Telegram Bot, API Gateway nhận webhook callback, Lambda `lambda_approval_handler` xác thực qua Amazon Cognito và giải phóng Step Function.
* **Execution Layer:** Lambda `lambda_executor` thực thi hành động bảo vệ, ghi kết quả vào DynamoDB `incident_history` và S3.

### 2.2 Kế hoạch kỹ thuật

Đội ngũ dự án phát triển và triển khai hệ thống theo quy trình kỹ thuật sau:

* **Infrastructure as Code:** Toàn bộ hạ tầng AWS được định nghĩa bằng **Terraform** (modules: lambdas, step\_functions, agents, cognito, api\_gateway, dynamodb, s3). S3 backend lưu `terraform.tfstate` đảm bảo state được bảo vệ và có thể cộng tác.
* **Multi-Agent Pipeline:** Supervisor Agent được cấu hình với 3 action groups (Parser, History, Knowledge) cho phép agentic loop – tự quyết định khi nào cần thu thập thêm dữ liệu. Advisor Agent (pure LLM) nhận structured JSON từ Supervisor và tổng hợp kế hoạch ứng phó.
* **RAG với Pinecone:** Luồng xử lý: guideline text → Amazon Titan Embeddings → vector → Pinecone upsert. Khi có sự cố: finding description → embedding → Pinecone query top-3 → đưa vào prompt Agent.
* **Human-in-the-Loop với `waitForTaskToken`:** Step Functions tạm dừng vô thời hạn tại state `Send Telegram And Wait`, task token được lưu trong DynamoDB `task_tokens` (TTL 24 giờ), chỉ tiếp tục khi Lambda `approval_handler` gọi `send_task_success` hoặc `send_task_failure`.

### 2.3 Kế hoạch dự án

Dự án được thực hiện theo mô hình **incremental** trong 3 giai đoạn:

* **Giai đoạn 1 – Core Pipeline (Tuần 1–2):** Thiết kế kiến trúc, viết Terraform modules cơ bản, triển khai Lambda parser/history/knowledge và Step Function 4 states đầu (không có Telegram/Executor).
* **Giai đoạn 2 – Intelligence Layer (Tuần 3–4):** Tích hợp Bedrock Agents (Supervisor + Advisor), thiết lập Pinecone index, viết và kiểm thử `lambda_advisor` với agentic loop.
* **Giai đoạn 3 – Response & Hardening (Tuần 5–6):** Tích hợp Telegram Bot, API Gateway webhook, Cognito User Pool, `lambda_approval_handler`, `lambda_executor` (simulation mode); kiểm thử end-to-end toàn pipeline.

### 2.4 Các yếu tố bảo mật

Bảo mật được thiết kế theo nguyên tắc **Least Privilege** và **Defense in Depth**:

* **IAM Least Privilege:** Mỗi Lambda function có IAM Role riêng, chỉ được cấp đúng quyền cần thiết (ví dụ: `lambda_history` chỉ có `dynamodb:GetItem`, `dynamodb:PutItem` trên đúng bảng `incident_history`).
* **Secrets Management:** Các thông tin nhạy cảm (`telegram_token`, `telegram_chat_id`, `pinecone_api_key`) được truyền vào Terraform qua `terraform.tfvars` (không commit vào repository) và inject vào Lambda environment variables.
* **Xác thực admin:** Cognito User Pool với attribute `custom:telegram_id` đảm bảo chỉ admin đã đăng ký mới có thể phê duyệt hành động bảo mật.
* **API Gateway security:** Endpoint `/webhook` tiếp nhận callback từ Telegram; xác thực tính hợp lệ của request được thực hiện trong `lambda_approval_handler` (kiểm tra cấu trúc payload và telegram_id).
* **CloudWatch Logging:** Toàn bộ 8 Lambda functions có log group riêng với retention 30 ngày. Step Functions logging ở mức `ALL`.
* **S3 bucket policy:** Bucket báo cáo không public, chỉ cho phép Lambda executor có quyền `s3:PutObject`.

---

## 3. HOẠT ĐỘNG VÀ KẾT QUẢ BÀN GIAO

### 3.1 Activities and deliverables

| Giai đoạn triển khai | Timeline | Hoạt động | Milestones | Ngày hoàn thành |
| :--- | :--- | :--- | :--- | :--- |
| **Core Pipeline** | Tuần 1–2 | - Thiết kế kiến trúc 4 lớp.<br>- Viết Terraform modules.<br>- Triển khai Lambda parser, history, knowledge.<br>- Step Function 4 states. | - Sơ đồ kiến trúc hoàn chỉnh.<br>- Pipeline 4 states chạy được với mock event.<br>- DynamoDB và S3 hoạt động. | Tuần 2 |
| **Intelligence Layer** | Tuần 3–4 | - Cấu hình Bedrock Agents (Supervisor + Advisor).<br>- Thiết lập Pinecone index + seed guideline mẫu.<br>- Viết và kiểm thử `lambda_advisor`. | - Supervisor Agent agentic loop hoạt động.<br>- Advisor Agent tạo kế hoạch ứng phó 7 phần.<br>- RAG trả về guideline phù hợp. | Tuần 4 |
| **Response & Hardening** | Tuần 5–6 | - Tích hợp Telegram Bot và API Gateway webhook.<br>- Triển khai Cognito User Pool + user mapping.<br>- Viết `lambda_approval_handler` và `lambda_executor`.<br>- Kiểm thử end-to-end toàn pipeline. | - Tin nhắn Telegram gửi được kèm Approve/Reject.<br>- Pipeline hoàn chỉnh 6 states Succeeded.<br>- Kết quả ghi vào DynamoDB và S3. | Tuần 6 |

### 3.2 Nằm ngoài dự án

Các hạng mục sau nằm ngoài phạm vi của phiên bản hiện tại:

* Tự động trigger từ GuardDuty qua EventBridge (hiện dùng mock event thay thế).
* Thực thi hành động bảo mật thật (NACL modification, Security Group update, IAM key revocation) – hiện ở chế độ simulation.
* Dashboard giám sát tập trung (CloudWatch Dashboard tổng hợp metrics toàn pipeline).
* Tích hợp Bedrock Knowledge Base (Amazon native) thay thế Pinecone.
* CloudWatch Scheduled Rule cho `lambda_token_cleaner`.

### 3.3 Quy trình đưa vào vận hành

Phiên bản hiện tại là **proof-of-concept** đã hoạt động đầy đủ. Để đưa vào môi trường sản xuất, cần thực hiện thêm:

* **Kích hoạt EventBridge trigger:** Tạo EventBridge rule lắng nghe GuardDuty findings và tự động kích hoạt Step Function – thay thế hoàn toàn mock event.
* **Implement Executor thật:** Thay nội dung simulation trong `block_ip_nacl()`, `isolate_instance()`, `revoke_iam_keys()` bằng lệnh gọi AWS SDK thực tế; cập nhật IAM policy cho Lambda executor.
* **Seed Pinecone knowledge base:** Xây dựng quy trình nhập và cập nhật guideline bảo mật định kỳ (playbook nội bộ, NIST, CIS Controls).
* **Operational Excellence:** Thêm CloudWatch Alarms cho Lambda errors/timeouts, thiết lập CloudWatch Dashboard tổng hợp, bật CloudWatch Scheduled Rule cho token cleaner.
* **Production Agent alias:** Tạo alias PROD cho Bedrock Agents (thay `TSTALIASID`) sau khi kiểm thử ổn định.

---

## 4. PHÂN TÍCH CHI PHÍ THEO DỊCH VỤ

Chi phí ước tính vận hành ở mức **~14.46 USD/tháng** khi chạy với lưu lượng test thông thường. Chi phí khi không có sự cố gần bằng 0 nhờ kiến trúc Serverless.

| Layer | AWS Service | Mục đích | Chi phí/tháng |
| :--- | :--- | :--- | :--- |
| **Detection** | Amazon GuardDuty | Phân tích VPC Flow Logs (6GB) | ~$6.90 |
| **Intelligence** | Amazon Bedrock (On-Demand) | Bedrock Agent inference (Claude 3.5 Sonnet) | ~$5.00 |
| **Orchestration** | AWS Step Functions | Standard Workflow executions | ~$0.50 |
| **Compute** | AWS Lambda (8 functions) | Invocations + Duration | ~$0.50 |
| **Storage** | DynamoDB + S3 | Lịch sử sự cố + Báo cáo remediation | ~$0.60 |
| **Integration** | API Gateway + Cognito | Webhook + Auth | ~$0.10 |
| **External** | Pinecone | Vector DB (Free Tier) | $0.00 |
| **Tổng** | | | **~$14.46** |

> **Lưu ý:** Chi phí Bedrock phụ thuộc trực tiếp vào số lần phân tích sự cố. Với kiến trúc On-Demand, không phát sinh chi phí khi không có finding nào được xử lý.

---

## 5. ĐỘI NGŨ

### Chịu trách nhiệm tổng thể

| Tên | Chức danh | Mô tả | Email |
| :--- | :--- | :--- | :--- |
| *Nguyễn Gia Hưng* | *Head of Architecture Solution* | *Thiết kế và phát triển các nền tảng đám mây gốc và phi máy chủ* | *hunggia@amazon.com* |

### Các bên liên quan

| Tên | Chức danh | Mô tả | Email |
| :--- | :--- | :--- | :--- |
| *Đình Quang Sáng* | PQHDN | Đánh giá & Định hướng | *SangDQ6@fe.edu.vn* |

### Đại diện hỗ trợ

| Tên | Chức danh | Mô tả | Email |
| :--- | :--- | :--- | :--- |
| *Văn Hoàng Kha* | Cloud Security Engineer, Co-founded and led Viet-AWS | Thực thi các hướng kỹ thuật về Bảo mật đám mây và DevSecOps | *khavan.work@gmail.com* |

### Đội ngũ thực hiện dự án

| Tên | Chức danh | Mô tả | Email |
| :--- | :--- | :--- | :--- |
| *(Tên thành viên 1)* | Leader + Cloud Architect | Thiết kế kiến trúc tổng thể, viết Terraform modules, triển khai Step Functions | *(email)* |
| *(Tên thành viên 2)* | AI/ML Engineer | Cấu hình Bedrock Agents, xây dựng RAG pipeline, kiểm thử Multi-Agent | *(email)* |
| *(Tên thành viên 3)* | Backend Engineer | Viết Lambda functions, tích hợp Telegram Bot, API Gateway, Cognito | *(email)* |

---

## 6. NGUỒN LỰC VÀ ƯỚC TÍNH CHI PHÍ

### Định mức đơn giá nhân sự
*(Đơn giá nhân sự ước tính dựa trên định mức dự án sinh viên/học thuật)*

| Nguồn lực / Vai trò | Trách nhiệm | Đơn giá (USD) / Giờ |
| :--- | :--- | :--- |
| **Cloud Architect** | - Thiết kế kiến trúc 4 lớp Event-Driven SOAR.<br>- Viết toàn bộ Terraform modules (Lambda, Step Functions, Bedrock Agents, DynamoDB, S3, API Gateway, Cognito).<br>- Đảm bảo IaC nhất quán, IAM least privilege, tối ưu chi phí. | **2.3** |
| **AI/ML Engineer** | - Thiết kế Supervisor Agent (action groups, agentic loop) và Advisor Agent (prompt engineering).<br>- Xây dựng RAG pipeline: Titan Embeddings, Pinecone index, query strategy.<br>- Kiểm thử và đánh giá chất lượng kế hoạch ứng phó của AI. | **0.7** |
| **Backend Engineer** | - Viết code cho 8 Lambda functions (Python).<br>- Tích hợp Telegram Bot, API Gateway webhook, Cognito auth flow.<br>- Thiết lập CloudWatch Logs, kiểm thử end-to-end. | **0.7** |

### Ước tính giờ công & chi phí theo giai đoạn

| Giai đoạn Dự án | Vai trò | Giờ công | Chi phí (USD) | Tổng Chi phí Giai đoạn |
| :--- | :--- | :--- | :--- | :--- |
| **Giai đoạn 1: Core Pipeline**<br>*(Tuần 1–2)* | **Cloud Architect**<br>Backend Engineer<br>AI/ML Engineer | 40<br>30<br>10 | $92.0<br>$21.0<br>$7.0 | **$120.0** |
| **Giai đoạn 2: Intelligence Layer**<br>*(Tuần 3–4)* | Cloud Architect<br>**AI/ML Engineer**<br>Backend Engineer | 20<br>60<br>20 | $46.0<br>$42.0<br>$14.0 | **$102.0** |
| **Giai đoạn 3: Response & Hardening**<br>*(Tuần 5–6)* | Cloud Architect<br>**Backend Engineer**<br>AI/ML Engineer | 20<br>60<br>20 | $46.0<br>$42.0<br>$14.0 | **$102.0** |
| **TỔNG CỘNG** | | **280** | | **$324.0** |

### Phân bổ đóng góp chi phí

| Bên tham gia | Giá trị Đóng góp (USD) | Tỷ lệ Đóng góp trên Tổng số |
| :--- | :--- | :--- |
| **Partner (Team)** | **$324.0** | **95.7%** (Chi phí nhân sự tự đóng góp) |
| **AWS** | **~$14.46/tháng** | **4.3%** (Chi phí hạ tầng Cloud ước tính) |
| **Customer** | **$0** | **0%** (Dự án học thuật/POC) |

---

## 7. NGHIỆM THU

Việc nghiệm thu dự án **Cloud-Sentinel** căn cứ vào hoạt động đúng đắn của toàn bộ pipeline end-to-end và chất lượng đầu ra của AI Agent.

### 7.1. Gói bàn giao

Sản phẩm được coi là sẵn sàng để nghiệm thu khi đội dự án cung cấp đầy đủ các hạng mục sau:

* **Mã nguồn (Source Code):** Repository chứa toàn bộ code Lambda functions (Python), Terraform modules, và script seed Pinecone.
* **Tài liệu kỹ thuật (Documentation):** Workshop report hoàn chỉnh (mục 2.1–2.5), bao gồm thiết kế kiến trúc, hướng dẫn triển khai và Lab từng bước.
* **Hạ tầng vận hành (Live Infrastructure):** Toàn bộ tài nguyên AWS đã được triển khai thành công bằng `terraform apply` trên tài khoản AWS thực tế.

### 7.2. Quy trình nghiệm thu

Quy trình diễn ra theo trình tự 4 bước:

1. **Trình diễn (Live Demo):** Đội dự án chạy toàn bộ pipeline trực tiếp: gửi mock finding → Step Function chạy → Telegram nhận tin nhắn → admin Approve → execution Succeeded → kiểm tra S3/DynamoDB.
2. **Kiểm thử chấp nhận (UAT):** Mentor/Giám khảo tự gửi mock finding với các loại sự cố khác nhau (Portscan, SSHBruteForce, IAMAnomaly), đánh giá chất lượng kế hoạch ứng phó do AI tạo ra.
3. **Phản hồi & Khắc phục:** Đội dự án cam kết xử lý lỗi nghiêm trọng (pipeline không chạy được) trong 24 giờ. Lỗi nhỏ (định dạng output, nội dung prompt) trong 48 giờ.
4. **Xác nhận hoàn thành:** Dự án được nghiệm thu khi pipeline chạy đúng end-to-end và kế hoạch ứng phó AI có nội dung phù hợp với loại sự cố đầu vào.

### 7.3. Điều kiện từ chối

Sản phẩm chưa được nghiệm thu nếu:

* **Lỗi pipeline:** Step Function execution thất bại trước state `Send Telegram And Wait` (các state core không hoàn thành).
* **Telegram không hoạt động:** Bot không gửi được tin nhắn hoặc nút Approve/Reject không kích hoạt được Step Function tiếp tục.
* **AI không tạo được kế hoạch:** Advisor Agent trả về output rỗng hoặc không có cấu trúc (thiếu Executive Summary, Immediate Remediation).
* **Vượt ngân sách:** Chi phí vận hành thực tế vượt quá 20 USD/tháng mà không có giải trình hợp lý.