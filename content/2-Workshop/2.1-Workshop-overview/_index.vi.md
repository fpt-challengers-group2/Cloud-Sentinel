---
title : "Giới thiệu"
weight : 1 
chapter : false
pre : " <b> 2.1. </b> "
---

# Cloud-Sentinel

## 1. Tổng quan dự án Cloud-Sentinel

**Cloud-Sentinel** là một hệ thống **SOAR (Security Orchestration, Automation and Response)** hiện đại, được thiết kế để bảo vệ các hạ tầng trọng yếu (như Ngân hàng/Tài chính) trên nền tảng AWS. Hệ thống vận hành theo cơ chế **Event-Driven** (Dựa trên sự kiện) kết hợp với **Multi-Agent AI** để giải quyết bài toán: **Phát hiện nhanh - Phân tích thông minh - Phản ứng tự động**.

### Vấn đề được giải quyết & Giá trị Kỹ thuật
* **Thách thức:** Chuyển đổi quy trình vận hành bảo mật từ thủ công sang tự động nhằm giảm thiểu gánh nặng cho quản trị viên. Loại bỏ các báo động giả thông qua quy trình đánh giá chéo giữa các AI Agent.
* **Giải pháp Kỹ thuật:** Chúng tôi giải quyết vấn đề này bằng cách triển khai một luồng điều phối thông minh sử dụng **AWS Step Functions** kết hợp với trí tuệ nhân tạo từ **Amazon Bedrock**. Hệ thống cung cấp khả năng phân tích sự cố theo thời gian thực và tự động thực thi các hành động bảo vệ sau khi có sự kiểm duyệt của chuyên viên.


---

## 2. Chiến lược thiết kế & Tối ưu chi phí

Dự án được thiết kế theo mô hình **Serverless** hoàn toàn để tối ưu hóa chi phí vận hành, đảm bảo bạn chỉ trả tiền cho những gì thực sự sử dụng (Pay-as-you-go).

* **Tổng chi phí ước tính:** ~14.46 USD/tháng.
* **Chi phí duy trì khi không có sự cố:** Gần như bằng 0 nhờ kiến trúc hướng sự kiện.

### Điểm nổi bật về tối ưu hóa:
* **Amazon GuardDuty:** Chỉ tập trung phân tích EC2 VPC Flow Logs (ước tính 6GB/tháng) thay vì quét toàn bộ log thô để kiểm soát ngân sách ở mức ~6.90 USD.
* **Trí tuệ nhân tạo tiết kiệm:** Sử dụng mô hình **On-Demand** của Amazon Bedrock cho các Agent, đảm bảo chi phí chỉ phát sinh khi có yêu cầu phân tích sự cố thực tế.
* **Hạ tầng minh bạch:** Sử dụng **Terraform** giúp liệt kê chính xác mọi tài nguyên và chi phí dự kiến trước khi triển khai, loại bỏ hoàn toàn các tài nguyên "rác".

---

## 3. Điều kiện tiên quyết (Prerequisites)

Để đảm bảo quá trình thực nghiệm diễn ra suôn sẻ và tối ưu chi phí, bạn cần chuẩn bị các mục sau:

### A. Công cụ lập trình & Quản lý hạ tầng
* **AWS CLI v2:** Đã cài đặt và cấu hình `access_key`/`secret_key` có quyền **AdministratorAccess**.
* **Terraform CLI:** Phiên bản từ 1.5 trở lên để quản lý hạ tầng một cách chuyên nghiệp và đồng bộ.
* **Python 3.11+:** Cần thiết để đóng gói các gói Runtime cho Lambda và xây dựng logic cho các AI Agent.

### B. Cấu hình dịch vụ AWS
* **Amazon Bedrock:** Truy cập vào Console AWS vùng **Singapore (ap-southeast-1)**, vào mục **Model Access** và kích hoạt quyền sử dụng cho các model **Claude** hoặc **Llama**.
* **S3 Backend:** Tạo thủ công một S3 Bucket (ví dụ: `cloud-sentinel-tfstate-yourname`) để lưu trữ file `terraform.tfstate`, giúp bảo vệ trạng thái hạ tầng an toàn.

### C. Tích hợp bên thứ ba (External Integration)
* **Telegram:** Tạo một Bot qua `@BotFather` và lưu lại **Bot Token** để phục vụ lớp phản ứng (Action Layer).
* **Pinecone:** Đăng ký tài khoản (Free Tier) và lấy **API Key** để phục vụ cho lớp lưu trữ tri thức (Vector Database).

---

## 4. Khởi tạo hạ tầng với Terraform

Trong module này, người tham dự sẽ học cách triển khai hạ tầng bảo mật hoàn chỉnh bằng Terraform. Việc sử dụng Terraform mang lại tính nhất quán và khả năng tái sử dụng cao cho dự án Cloud-Sentinel.

**Các lệnh cơ bản sẽ sử dụng:**
1. `terraform init`: Khởi tạo project và kết nối với S3 Backend.
2. `terraform plan`: Xem trước các thay đổi và dự toán tài nguyên sẽ được tạo.
3. `terraform apply`: Thực thi triển khai hạ tầng lên AWS.

![Agentic Workflow](/25FALL/OJT/AWS_S2/cloud-sentinel/public/images/High_Level_System_Architecture.gif)