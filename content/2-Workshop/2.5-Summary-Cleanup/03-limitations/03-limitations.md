---
title: "Limitations and Future Work"
weight: 3
chapter: false
pre: " <b> 2.5.3. </b> "
---

# 2.5.1.3. Current Limitations and Future Directions

The current system functions as a proof-of-concept. Notes for production readiness:

| Area | Current Status | Future Work |
|------|----------------|-------------|
| GuardDuty trigger | Using manual mock events | Implement EventBridge rule to auto-trigger Step Function on GuardDuty findings |
| Lambda Executor | Simulation (logs only) | Implement real actions: NACL modifications, Security Group updates, IAM key revocation |
| Pinecone knowledge base | Index created, needs real guideline seeding | Build ingestion and periodic update pipeline for guidelines |
| Token cleaner | Lambda exists, no CloudWatch Schedule | Add EventBridge Scheduled Rule to run `lambda_token_cleaner` hourly |
| Monitoring | Basic CloudWatch Logs | Build aggregate CloudWatch Dashboard + CloudWatch Alarms for Lambda errors |
| Bedrock Agent alias | Using `TSTALIASID` (DRAFT) | Create PROD alias after stable testing |
