---
title: "State 3: Get Knowledge"
weight: 4
chapter: false
pre: " <b> 2.3.4. </b> "
---

# 2.3.4. State 3 – Get Knowledge (Lambda `lambda_knowledge`)

**Function:** Query the Pinecone Vector Database to retrieve playbooks (guidelines) most relevant to the current incident. This implements the RAG (Retrieval-Augmented Generation) layer.

**Flow:**
1. Lambda receives `finding_type` and `description` as input.
2. Calls **Amazon Titan Text Embeddings** to convert text into vectors.
3. Queries Pinecone index `cloud-sentinel-index` to get the **top 3** most similar guidelines.
4. Returns the list of guidelines as text to be included in the Bedrock Agent prompt.

**Pinecone index status:**
- Index name: `cloud-sentinel-index`
- Status: Created, no guideline data yet (record count = 0).
- See seeding instructions in the Lab (section 2.3.4 of `08-prerequisites.md`).
