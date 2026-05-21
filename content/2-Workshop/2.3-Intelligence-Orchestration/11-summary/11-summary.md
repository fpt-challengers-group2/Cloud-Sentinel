---
title: "Intelligence Layer Summary"
weight: 11
chapter: false
pre: " <b> 2.3.11. </b> "
---

# 2.3.11. Intelligence Layer Summary

After completing this module you understand and practiced:

- **Pipeline architecture:** the 6-state Step Functions flow from finding to remediation plan.
- **Multi-Agent design:** Supervisor Agent gathers context via action groups; Advisor Agent synthesizes and drafts remediation plans.
- **RAG with Pinecone:** `lambda_knowledge` uses vector search to provide relevant guidelines to the AI Agent.
- **Result storage:** DynamoDB records incident history; S3 stores remediation reports as JSON.
