---
title: "State 3: Get Knowledge"
weight: 4
chapter: false
pre: " <b> 2.3.4. </b> "
---

# 2.3.4. State 3 – Get Knowledge (Lambda `lambda_knowledge`)

**Chức năng:** Truy vấn Pinecone Vector Database để lấy các hướng dẫn xử lý (guideline) phù hợp nhất với loại sự cố hiện tại. Đây là lớp RAG (Retrieval-Augmented Generation) của hệ thống.

**Cơ chế hoạt động:**
1. Lambda nhận `finding_type` và `description` làm input.
2. Gọi **Amazon Titan Text Embeddings** để chuyển đổi text thành vector.
3. Truy vấn Pinecone index `cloud-sentinel-index` lấy **top 3** guideline có độ tương đồng cao nhất.
4. Trả về danh sách guideline dưới dạng text để đưa vào prompt của Bedrock Agent.

**Pinecone index hiện tại:**
- Index name: `cloud-sentinel-index`
- Trạng thái: Đã tạo, chưa có dữ liệu guideline (record count = 0).
- Xem hướng dẫn seed dữ liệu tại phần Lab (mục 2.3.4 trong file `08-prerequisites.vi.md`).
