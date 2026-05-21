---
title : "Introduction"
weight : 1 
chapter : false
pre : " <b> 2.1. </b> "
---

# Cloud-Sentinel

## 1. Project overview — Cloud-Sentinel

**Cloud-Sentinel** is a modern **SOAR (Security Orchestration, Automation and Response)** system designed to protect critical infrastructures (e.g., Banking/Finance) on AWS. The system operates in an **Event-Driven** manner combined with **Multi-Agent AI** to solve: **Fast detection - Intelligent analysis - Automated response**.

### Problem Statement & Technical Value
- **Challenge:** Shift security operations from manual to automated processes to reduce operator burden and eliminate false alarms through cross-evaluation between AI Agents.
- **Technical Solution:** We address this by implementing an intelligent orchestration pipeline using **AWS Step Functions** together with AI from **Amazon Bedrock**. The system provides real-time incident analysis and can automatically execute protective actions after human review.

---

## 2. Design Strategy & Cost Optimization

The project is designed as fully **Serverless** to optimize operational costs and ensure pay-as-you-go billing.

- **Estimated total cost:** ~14.46 USD/month.
- **Idle cost:** Near zero thanks to event-driven architecture.

### Cost optimization highlights:
- **Amazon GuardDuty:** Focus on EC2 VPC Flow Logs analysis (estimated 6GB/month) instead of scanning all raw logs to limit budget to ~6.90 USD.
- **Cost-efficient AI:** Use Amazon Bedrock On-Demand models for Agents so costs occur only during actual analysis.
- **Transparent infra:** Use **Terraform** to enumerate resources and expected costs before deployment, avoiding stray resources.

---

## 3. Prerequisites

Prepare the following to ensure smooth experiments and cost control:

### A. Development & Infra tools
- **AWS CLI v2:** Installed and configured with access_key/secret_key having **AdministratorAccess**.
- **Terraform CLI:** Version 1.5+ for professional infra management.
- **Python 3.11+:** Required for packaging Lambda runtimes and Agent logic.

### B. AWS service configuration
- **Amazon Bedrock:** Access the AWS Console in **Singapore (ap-southeast-1)**, go to **Model Access** and enable access for **Claude** or **Llama** models.
- **S3 Backend:** Create an S3 Bucket (e.g., cloud-sentinel-tfstate-yourname) for `terraform.tfstate` to secure terraform state.

### C. External integrations
- **Telegram:** Create a Bot via @BotFather and save the Bot Token for the Action Layer.
- **Pinecone:** Sign up (Free Tier) and obtain an API Key for the knowledge vector database.

---

## 4. Bootstrapping infrastructure with Terraform

In this module you'll learn Terraform-based infra provisioning for Cloud-Sentinel. Terraform ensures consistency and reusability.

**Common commands used:**
1. `terraform init`: Initialize the project and connect to the S3 backend.
2. `terraform plan`: Preview changes and estimate resources.
3. `terraform apply`: Deploy infrastructure to AWS.

![Agentic Workflow](/images/High_Level_System_Architecture.gif)
