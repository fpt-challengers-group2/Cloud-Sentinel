---
title: "Intelligence & Orchestration Layer"
weight: 3
chapter: false
pre: " <b> 2.3. </b> "
---

# 2.3. Intelligence & Orchestration Layer – Step Functions & Bedrock Agents

## Overview

The Intelligence & Orchestration layer is the central processing unit of CloudSentinel. After the Detection Layer (2.2) provides a normalized finding, this layer performs three ordered tasks:

1. **Context gathering** – check incident history (DynamoDB) and query playbooks (Pinecone RAG).
2. **Intelligent analysis** – feed context into a pipeline of two Bedrock Agents to synthesize and generate a remediation plan.
3. **Result handoff** – send the plan to the Response Layer (2.4) for approval and execution.

Core components:

| Component | AWS Service | Role |
|-----------|-------------|------|
| Orchestrator | AWS Step Functions | Orchestrates the entire 6-state flow |
| Parser | Lambda `lambda_parser` | Normalizes GuardDuty findings |
| History | Lambda `lambda_history` | Checks precedents in DynamoDB |
| Knowledge | Lambda `lambda_knowledge` | Queries guidance from Pinecone (RAG) |
| Agents Pipeline | Lambda `lambda_advisor` + Bedrock | Analyze and produce remediation plans |
| Storage | DynamoDB + S3 | Store history and reports |

Detailed contents:
- [Step Functions Orchestrator](./01-stepfunctions/01-stepfunctions.md)
- [State 1: Parse Finding](./02-state-parser/02-state-parser.md)
- [State 2: Check Precedent](./03-state-history/03-state-history.md)
- [State 3: Get Knowledge](./04-state-knowledge/04-state-knowledge.md)
- [State 4: Invoke Agents Pipeline](./05-state-agents/05-state-agents.md)
- [Storage: DynamoDB and S3](./06-storage/06-storage.md)
- [Data flow](./07-data-flow.md)
- [Prerequisites](./08-prerequisites.md)
- [Lab: Run the pipeline](./09-lab-run.md)
- [Verification](./10-verification.md)
- [Summary](./11-summary.md)
