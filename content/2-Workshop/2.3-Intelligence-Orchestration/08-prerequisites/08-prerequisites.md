---
title: "Prerequisites before Lab"
weight: 8
chapter: false
pre: " <b> 2.3.8. </b> "
---

# 2.3.8. Prerequisites before the lab

Before running the lab, ensure the following items are ready:

## A. Enable Bedrock Model Access

The Bedrock Agent uses model `anthropic.claude-3-5-sonnet-20240620-v1:0`. This model must be manually enabled in the AWS Console before it can be called.

**Steps:**

1. Open the **AWS Console** and set region to **Singapore (`ap-southeast-1`)**.
2. Go to **Amazon Bedrock**.
3. In the left menu, select **Model access** (or **Bedrock configurations → Model access**).
4. Open the **Available to request** tab.
5. Find **Anthropic → Claude 3.5 Sonnet** and request access.
6. Click **Request model access** → Confirm.
7. Wait for status to become **Access granted** (usually under 5 minutes).

> **Note:** Skipping this step will cause `lambda_advisor` to receive `AccessDeniedException` when calling the Bedrock API.

## B. Seed Pinecone data (Guideline RAG)

Index `cloud-sentinel-index` is created but empty. Without guidelines, `lambda_knowledge` returns an empty list and Bedrock Agents will infer without external guidance.

**Option 1 – Seed sample guidelines (recommended for the workshop):**

Create `seed_pinecone.py` with the provided script to upload sample guidelines to Pinecone, then run:

```bash
pip install pinecone boto3
python seed_pinecone.py
```

**Option 2 – Skip:** Proceed with the lab without guidelines; `lambda_knowledge` will return an empty array and Agents will work without referencing specific guidelines.
