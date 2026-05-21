---
title: "Điều kiện tiên quyết trước Lab"
weight: 8
chapter: false
pre: " <b> 2.3.8. </b> "
---

# 2.3.8. Điều kiện tiên quyết trước khi thực hành

Trước khi thực hiện Lab, đảm bảo các mục sau đã sẵn sàng:

## A. Kích hoạt Bedrock Model Access

Bedrock Agent sử dụng model `anthropic.claude-3-5-sonnet-20240620-v1:0`. Model này cần được kích hoạt thủ công trong AWS Console trước khi có thể gọi.

**Các bước thực hiện:**

1. Vào **AWS Console** → chọn region **Singapore (`ap-southeast-1`)**.
2. Tìm kiếm và mở dịch vụ **Amazon Bedrock**.
3. Trong menu trái, chọn **Model access** (hoặc **Bedrock configurations → Model access**).
4. Chọn tab **Available to request**.
5. Tìm **Anthropic → Claude 3.5 Sonnet**, đánh dấu chọn.
6. Nhấn **Request model access** → Xác nhận.
7. Chờ trạng thái chuyển sang **Access granted** (thường dưới 5 phút).

> **Lưu ý:** Nếu bỏ qua bước này, Lambda `lambda_advisor` sẽ nhận lỗi `AccessDeniedException` khi gọi Bedrock API.

## B. Seed dữ liệu Pinecone (Guideline RAG)

Index `cloud-sentinel-index` đã được tạo nhưng chưa có dữ liệu. Không có guideline, Lambda `lambda_knowledge` sẽ trả về danh sách rỗng và Bedrock Agent sẽ phải tự suy luận mà không có ngữ cảnh từ knowledge base.

**Tùy chọn 1 – Seed dữ liệu mẫu (khuyến nghị cho workshop):**

Tạo file `seed_pinecone.py` với nội dung sau:

```python
from pinecone import Pinecone
import boto3
import json

# Cấu hình
PINECONE_API_KEY = "<your-pinecone-api-key>"
INDEX_NAME = "cloud-sentinel-index"
REGION = "ap-southeast-1"

# Khởi tạo Pinecone
pc = Pinecone(api_key=PINECONE_API_KEY)
index = pc.Index(INDEX_NAME)

# Khởi tạo Bedrock để tạo embedding
bedrock = boto3.client("bedrock-runtime", region_name=REGION)

def get_embedding(text):
    response = bedrock.invoke_model(
        modelId="amazon.titan-embed-text-v1",
        body=json.dumps({"inputText": text})
    )
    return json.loads(response["body"].read())["embedding"]

# Dữ liệu guideline mẫu
guidelines = [
    {
        "id": "guide-portscan-001",
        "text": "Recon:EC2/Portscan – Khi phát hiện port scan từ IP bên ngoài: (1) Block IP tại Network ACL, (2) Review Security Group rules, (3) Enable VPC Flow Logs nếu chưa có, (4) Kiểm tra các port đang mở không cần thiết."
    },
    {
        "id": "guide-ssh-bruteforce-001",
        "text": "UnauthorizedAccess:EC2/SSHBruteForce – Khi phát hiện SSH brute force: (1) Block IP nguồn tại NACL, (2) Rotate SSH key pairs, (3) Xem xét cấu hình SSH chỉ cho phép key-based auth, (4) Enable fail2ban hoặc tương đương."
    },
    {
        "id": "guide-iam-anomaly-001",
        "text": "UnauthorizedAccess:IAMUser/ConsoleLoginSuccess – Khi phát hiện đăng nhập IAM bất thường: (1) Revoke access keys ngay lập tức, (2) Kiểm tra CloudTrail 24 giờ trước, (3) Review IAM policies của user bị ảnh hưởng, (4) Enable MFA bắt buộc."
    }
]

# Upload lên Pinecone
vectors = []
for g in guidelines:
    embedding = get_embedding(g["text"])
    vectors.append({
        "id": g["id"],
        "values": embedding,
        "metadata": {"text": g["text"]}
    })

index.upsert(vectors=vectors)
print(f"Đã seed {len(vectors)} guideline vào Pinecone index '{INDEX_NAME}'.")
```

Chạy script:

```bash
pip install pinecone boto3
python seed_pinecone.py
```

**Tùy chọn 2 – Bỏ qua (tiếp tục Lab không có guideline):** Lambda `lambda_knowledge` sẽ trả về mảng rỗng. Bedrock Agent vẫn hoạt động nhưng kế hoạch ứng phó sẽ không tham chiếu được guideline cụ thể.
