---
title: "Tham chiếu chi phí - Tài liệu tham khảo"
weight: 6
chapter: false
pre: " <b> 2.5.6. </b> "
---

# 2.5.4. Tham chiếu chi phí

Chi phí ước tính cho toàn bộ workshop (tham khảo từ phần 2.1):

| Dịch vụ | Chi phí ước tính/tháng | Ghi chú |
|---------|----------------------|---------|
| Amazon GuardDuty | ~6.90 USD | 6GB VPC Flow Logs |
| Amazon Bedrock (On-Demand) | ~5.00 USD | Phụ thuộc số lần phân tích |
| AWS Lambda | ~0.50 USD | Pay-per-invocation |
| AWS Step Functions | ~0.50 USD | Standard Workflow |
| DynamoDB | ~0.50 USD | On-demand capacity |
| S3 | ~0.10 USD | Lưu báo cáo JSON |
| API Gateway | ~0.10 USD | REST API calls |
| Cognito | ~0.00 USD | Free tier (< 50,000 MAU) |
| Pinecone | ~0.00 USD | Free tier |
| **Tổng cộng** | **~14.46 USD/tháng** | Khi chạy với lưu lượng test |

**Chi phí khi không có sự cố:** Gần bằng 0. Kiến trúc Serverless đảm bảo không phát sinh chi phí compute khi không có finding nào được xử lý. Chỉ GuardDuty là dịch vụ luôn chạy nền và tính phí theo lưu lượng log.

# 2.5.5. Tài liệu tham khảo

| Chủ đề | Tài liệu |
|--------|----------|
| AWS Step Functions – waitForTaskToken | [docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html) |
| Amazon Bedrock Agents | [docs.aws.amazon.com/bedrock/latest/userguide/agents.html](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html) |
| Amazon GuardDuty Finding Types | [docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html) |
| Pinecone Python SDK | [docs.pinecone.io/reference/python-sdk](https://docs.pinecone.io/reference/python-sdk) |
| Terraform AWS Provider | [registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) |
| Amazon Titan Embeddings | [docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) |
