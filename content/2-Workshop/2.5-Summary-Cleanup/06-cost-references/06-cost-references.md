---
title: "Cost References"
weight: 6
chapter: false
pre: " <b> 2.5.6. </b> "
---

# 2.5.4. Cost Reference

Estimated monthly cost for the whole workshop (see 2.1):

| Service | Estimated Cost/month | Notes |
|---------|----------------------|-------|
| Amazon GuardDuty | ~6.90 USD | 6GB VPC Flow Logs |
| Amazon Bedrock (On-Demand) | ~5.00 USD | Depends on analysis volume |
| AWS Lambda | ~0.50 USD | Pay-per-invocation |
| AWS Step Functions | ~0.50 USD | Standard Workflow |
| DynamoDB | ~0.50 USD | On-demand capacity |
| S3 | ~0.10 USD | Storing JSON reports |
| API Gateway | ~0.10 USD | REST API calls |
| Cognito | ~0.00 USD | Free tier (< 50,000 MAU) |
| Pinecone | ~0.00 USD | Free tier |
| **Total** | **~14.46 USD/month** | For test traffic |

**Cost when idle:** Near zero. Serverless design ensures no compute costs when no findings are processed. GuardDuty remains running and incurs charges based on log volume.

# 2.5.5. References

- AWS Step Functions – waitForTaskToken: https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html
- Amazon Bedrock Agents: https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html
- Amazon GuardDuty Finding Types: https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html
- Pinecone Python SDK: https://docs.pinecone.io/reference/python-sdk
- Terraform AWS Provider: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
- Amazon Titan Embeddings: https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html
