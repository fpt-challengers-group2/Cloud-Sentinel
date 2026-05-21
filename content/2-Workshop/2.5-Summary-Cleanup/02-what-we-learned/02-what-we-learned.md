---
title: "What We Learned"
weight: 2
chapter: false
pre: " <b> 2.5.2. </b> "
---

# 2.5.1.2. What We Learned

**On SOAR architecture in AWS:**
- Design event-driven systems with near-zero cost when no incidents occur.
- Use Human-in-the-Loop with Step Functions' `waitForTaskToken` to control automated actions.
- Organize IAM with least privilege: each Lambda has only necessary permissions.

**On Multi-Agent AI:**
- Supervisor Agent uses action groups to proactively gather context (agentic loop).
- Advisor Agent (pure LLM) synthesizes and formulates structured remediation plans.
- Clear role separation simplifies control and debugging of each Agent.

**On RAG with Pinecone:**
- Vector search enables semantic guideline queries rather than exact keyword matches.
- Amazon Titan Embeddings bridge text to vector space.

**On operations:**
- Use Terraform to manage all infrastructure — resources are reproducible or removable with a command.
- Centralized CloudWatch Logs simplify debugging of each Lambda in the pipeline.
- Telegram Bot provides a simple, effective operational channel for approvals and notifications.
- Use DynamoDB and S3 to store history and reports, ensuring durability and easy retrieval.
